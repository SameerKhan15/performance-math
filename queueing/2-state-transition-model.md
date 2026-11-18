# 02 — State-Transition Model

## 1. Objective

Develop a mathematical understanding of how requests move between different states of a queueing system.

The primary objective is to understand **how arrival and service-completion events change the system state, and why the aggregate completion rate depends on the number of busy servers**.

This provides the foundation for deriving steady-state probabilities and, ultimately, the Erlang-C waiting probability.

By the end of this module, we should be able to:

- Define the state of an M/M/c queueing system.
- Distinguish busy servers from requests waiting in the queue.
- Understand how arrivals and completions cause state transitions.
- Derive the aggregate service-completion rate for any state.
- Understand why completion rates stop increasing once all servers are busy.
- Calculate the probability of the next transition being an arrival or completion.

---

## 2. Running Example: Database Connection Pool

Consider a hypothetical database connection pool:

| Parameter | Symbol | Value |
|---|---|---|
| Number of connections | $c$ | 4 |
| Arrival rate | $\lambda$ | 30 requests/sec |
| Mean service time | $S$ | 100 ms |
| Service rate per connection | $\mu=1/S$ | 10 requests/sec |
| Offered load | $a=\lambda S$ | 3 Erlangs |
| Average utilization | $\rho=a/c$ | 75% |

Each connection serves one request at a time.

When all connections are busy, additional requests wait in a queue.

We use the standard M/M/c model, assuming:

- Poisson arrivals at rate $\lambda$.
- Independent, exponentially distributed service times with mean $S$.
- $c$ identical parallel servers.
- An unlimited waiting queue with first-come-first-served scheduling.
- No request abandonment or cancellation.

The state-transition relationships derived below describe this idealized model.

---

## 3. Defining the System State

We define the system state as:

$$
\boxed{n=\text{Total number of requests in the system}}
$$

This includes both:

1. Requests actively being serviced.
2. Requests waiting for an available server.

For a pool containing $c$ connections:

**Number of busy connections:**

$$
\boxed{B(n)=\min(n,c)}
$$

**Number of waiting requests:**

$$
\boxed{Q(n)=\max(0,n-c)}
$$

### Example: Four-Connection Pool

| State $n$ | Busy connections | Waiting requests |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 3 | 0 |
| 4 | 4 | 0 |
| 5 | 4 | 1 |
| 6 | 4 | 2 |
| 7 | 4 | 3 |

### Physical Intuition

When $n<c$, there are idle connections available.

When $n=c$, all connections are busy, but no requests are waiting.

When $n>c$, all connections are busy and the excess requests must wait.

**The system state can exceed the number of connections because the state includes queued requests, not just requests currently being serviced.**

An important distinction:

- $n$ is the instantaneous number of requests in the system.
- $L$ is the long-run average number of requests in the system, as introduced in Little's Law.

These quantities should not be confused.

---

## 4. State Transitions

In our M/M/c model, the system state changes through two types of events.

### 4.1 Request Arrival

When a new request arrives:

$$
\boxed{n\rightarrow n+1}
$$

For example:

$$
3\rightarrow4
$$

The incoming request occupies the fourth available connection.

Another arrival produces:

$$
4\rightarrow5
$$

All four connections are already busy, so the new request enters the waiting queue.

Under the Poisson arrival assumption, the arrival rate is constant:

$$
\boxed{\lambda_n=\lambda}
$$

In our running example:

$$
\lambda_n=30\text{ arrivals/sec}
$$

This rate is independent of the current state.

### 4.2 Service Completion

When a request finishes service:

$$
\boxed{n\rightarrow n-1}
$$

For example, suppose the system is in state 7:

- Four requests are being serviced.
- Three requests are waiting.

When one active request completes, the total number of requests becomes six.

The next waiting request immediately occupies the released connection.

The resulting system has:

- Four busy connections.
- Two waiting requests.

Thus:

$$
7\rightarrow6
$$

**A service completion reduces the total number of requests by one, but it does not necessarily reduce the number of busy connections.**

---

## 5. Deriving the Aggregate Completion Rate

From Stage 1, we know:

$$
\mu=\frac{1}{S}
$$

With a mean service time of 100 ms:

$$
S=0.1\text{ seconds}
$$

Therefore:

$$
\mu=\frac{1}{0.1}
$$

$$
\boxed{\mu=10\text{ completions/sec}}
$$

This is the mean completion rate of one continuously busy connection.

### 5.1 One Busy Connection

If the system is in state $n=1$, only one connection is busy.

Its completion rate is:

$$
\mu_1=\mu=10/\text{sec}
$$

### 5.2 Two Busy Connections

If the system is in state $n=2$, two connections are busy.

Under the M/M/c assumptions, either connection can complete its request.

The aggregate completion rate is:

$$
\mu_2=2\mu
$$

$$
\boxed{\mu_2=20/\text{sec}}
$$

### 5.3 Three Busy Connections

Similarly:

$$
\mu_3=3\mu
$$

$$
\boxed{\mu_3=30/\text{sec}}
$$

### 5.4 Four Busy Connections

When all four connections are busy:

$$
\mu_4=4\mu
$$

$$
\boxed{\mu_4=40/\text{sec}}
$$

### 5.5 More Requests Than Connections

Suppose the system enters state $n=7$.

There are seven requests, but only four can execute concurrently.

The remaining three requests are waiting.

Therefore:

$$
\mu_7=4\mu
$$

$$
\boxed{\mu_7=40/\text{sec}}
$$

The completion rate remains the same as in state 4.

### General Relationship

The aggregate completion rate is determined by the number of busy servers.

Since:

$$
B(n)=\min(n,c)
$$

We obtain:

$$
\boxed{\mu_n=\min(n,c)\mu}
$$

### Completion Rate by State

| State $n$ | Busy connections | Completion rate |
|---|---|---|
| 0 | 0 | 0/sec |
| 1 | 1 | 10/sec |
| 2 | 2 | 20/sec |
| 3 | 3 | 30/sec |
| 4 | 4 | 40/sec |
| 5 | 4 | 40/sec |
| 6 | 4 | 40/sec |
| 7 | 4 | 40/sec |

### Key Intuition

**Before all connections are occupied, adding another request increases the number of busy connections and therefore increases aggregate completion capacity.**

Once all connections are busy, additional requests increase queue length without increasing processing capacity.

This change at $n=c$ will later explain why the state probabilities have two mathematical forms: a factorial relationship before saturation and a geometric tail afterward.

---

## 6. Understanding Transition Rates

A transition rate represents how frequently a particular type of state transition occurs, on average, while the system occupies a given state.

It does not mean that the next transition is guaranteed to occur in that direction.

For a system in state $n$:

**Arrival transition:**

$$
n\xrightarrow{\lambda}n+1
$$

**Completion transition:**

$$
n\xrightarrow{\mu_n}n-1
$$

Where:

$$
\mu_n=\min(n,c)\mu
$$

### Example: State 2

Suppose the system is in state $n=2$.

The competing transition rates are:

- Arrival rate: $\lambda=30$/sec.
- Aggregate completion rate: $\mu_2=20$/sec.

The next event could be either an arrival or a completion.

The higher arrival rate makes an arrival more likely, but it does not guarantee one.

---

## 7. Probability of the Next Transition

Under the M/M/c assumptions, the time until the next arrival is exponentially distributed.

Similarly, with $n$ busy connections, the time until the next service completion is exponentially distributed with rate $\mu_n$.

These two event processes compete.

The probability that the next event is an arrival is:

$${ P(\text{arrival next}\mid n) = \frac{\lambda}{\lambda+\mu_n} }$$

The probability that the next event is a completion is:

$${ P(\text{completion next}\mid n) = \frac{\mu_n}{\lambda+\mu_n} }$$

These relationships apply for $n>0$. In state 0, no service completion is possible, so the next transition must be an arrival.

### Example 1: State 2

Given:

$$
\lambda=30/\text{sec}
$$

$$
\mu_2=20/\text{sec}
$$

**Probability of an arrival occurring next:**

$$P(\text{arrival next}\mid n=2) = \frac{30}{30+20}$$

$$P(\text{arrival next}\mid n=2)=60\text{perc}$$

**Probability of a completion occurring next:**

$$P(\text{completion next}\mid n=2)=\frac{20}{30+20}$$

$${P(\text{completion next}\mid n=2)=40\text{perc}}$$

Therefore:

- There is a 60% probability of transitioning from state 2 to state 3.
- There is a 40% probability of transitioning from state 2 to state 1.

### Example 2: State 4

Now suppose all four connections are busy.

The aggregate completion rate is:

$$
\mu_4=40/\text{sec}
$$

The probability that the next event is a completion becomes:

$$P(\text{completion next}\mid n=4) = \frac{40}{30+40}$$

$${P(\text{completion next}\mid n=4) = 57.14\text{perc}}$$

The probability that the next event is an arrival is:

$$
P(\text{arrival next}\mid n=4)
=
\frac{30}{70}
$$

$$
\boxed{P(\text{arrival next}\mid n=4)\approx42.86\%}
$$

Thus, a completion is more likely to occur next, although an arrival remains possible.

### Important Distinction

These are **conditional probabilities of the next event given the current state**.

They are not the probabilities of finding the system in a particular state.

For example:

$$
P(\text{completion next}\mid n=4)
$$

is different from:

$$
P_4=P(N=4)
$$

The first measures the probability of the next transition direction.

The second measures the steady-state probability that the system contains exactly four requests.

We will derive state probabilities in Stage 3.

---

## 8. Why Completions Are Not Synchronized

Suppose four connections each have a mean service time of 100 ms.

Their combined completion rate is:

$$
4\times10=40\text{ completions/sec}
$$

This does not mean that exactly four requests finish every 100 ms.

Under the exponential service-time assumption, individual completion times are random.

Each busy connection has an independent service-completion process.

When four connections are busy, the time until the next completion is exponentially distributed with aggregate rate:

$$
4\mu=40/\text{sec}
$$

Consequently, the mean time until the next completion is:

$$
E[T_{\text{next completion}}]=\frac{1}{4\mu}
$$

$$
=\frac{1}{40}
$$

$$
\boxed{25\text{ ms}}
$$

This is the expected time until the next completion while four connections are busy, not the mean service time of an individual request.

An individual request still has a mean service time of 100 ms.

This illustrates why multiple parallel servers increase the aggregate completion rate.

---

## 9. Understanding the State-Transition Model

We can represent the queueing system as a sequence of states connected by arrival and completion transitions.

For the four-connection example:

```text
         λ          λ          λ          λ          λ          λ
   0 --------> 1 --------> 2 --------> 3 --------> 4 --------> 5 --------> 6 ...
     <--------   <--------   <--------   <--------   <--------   <--------
         μ          2μ         3μ         4μ         4μ         4μ
```

Each rightward transition represents a request arrival.

Each leftward transition represents a service completion.

The arrival rate remains constant at $\lambda$.

The completion rate increases as more connections become busy, then remains at $c\mu$ once all connections are occupied.

This structure is known as a **birth-death process**.

In our model:

- A *birth* corresponds to a request arrival.
- A *death* corresponds to a service completion.

Because arrivals and service times follow the memoryless exponential assumptions, the current state $n$ is sufficient to characterize the rates of future transitions.

We do not need to know how long individual requests have already been waiting or executing.

---

## 10. Worked Example

Consider a system with:

| Parameter | Value |
|---|---|
| Number of connections | 3 |
| Mean service time | 200 ms |
| Arrival rate | 12 requests/sec |
| Current state | $n=7$ |

### Step 1: Calculate the Per-Connection Service Rate

$$
\mu=\frac{1}{0.2}
$$

$$
\boxed{\mu=5/\text{sec}}
$$

### Step 2: Determine Busy and Waiting Requests

Busy connections:

$$
B(7)=\min(7,3)=\boxed{3}
$$

Waiting requests:

$$
Q(7)=\max(0,7-3)=\boxed{4}
$$

### Step 3: Calculate Aggregate Completion Rate

$$
\mu_7=\min(7,3)\times5
$$

$$
\boxed{\mu_7=15/\text{sec}}
$$

### Step 4: Determine Next-Transition Probabilities

Probability of an arrival:

$$
P(\text{arrival next}\mid n=7)
=
\frac{12}{12+15}
$$

$$
\boxed{\approx44.44\%}
$$

Probability of a completion:

$$
P(\text{completion next}\mid n=7)
=
\frac{15}{12+15}
$$

$$
\boxed{\approx55.56\%}
$$

The next event is more likely to be a completion, but either transition remains possible.

---

## 11. Key Takeaways

1. **The state $n$ represents all requests in the system**, including those waiting and those being serviced.
2. The number of busy servers is $\min(n,c)$.
3. The number of waiting requests is $\max(0,n-c)$.
4. An arrival increases the state by one; a completion decreases it by one.
5. The arrival rate is constant across states in the standard M/M/c model.
6. The aggregate completion rate increases with the number of busy servers, up to $c\mu$.
7. Once all servers are busy, additional requests increase queue length rather than processing capacity.
8. Transition rates describe probabilistic event behavior, not deterministic event sequences.
9. The probability of the next transition depends on the competing arrival and completion rates.
10. State-transition probabilities and steady-state state probabilities are distinct concepts.

The central relationship established in this module is:

$$
\boxed{\mu_n=\min(n,c)\mu}
$$

This relationship provides the mathematical foundation for the next stage.

---

## 12. Next Stage

**[3 — Steady-State Flow Balance](3-steady-state-flow-balance.md)**

We will use the state-transition rates developed here to derive the probabilities of occupying different system states.

The central idea will be that, in steady state, the probability flow between adjacent states must balance.

For example:

$$
P_0\lambda=P_1\mu
$$

From this relationship, we will progressively derive $P_1$, $P_2$, $P_3$, and higher state probabilities in terms of $P_0$.

This will explain why factorial terms emerge before saturation and prepare us to derive the geometric tail of the queueing distribution.