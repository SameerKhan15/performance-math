# 04 — The Geometric Tail

## 1. Objective

Develop a first-principles understanding of why the state probabilities of an M/M/c queue form a **geometric progression once all servers are busy**.

In Stage 3, we derived the state probabilities up to the point where all servers become occupied. We observed that the probabilities contain factorial terms because the aggregate completion rate increases as more servers become busy.

In this stage, we will examine what happens after the system reaches its maximum service capacity.

Our learning objectives are to:

1. Understand why the factorial probability pattern changes once all servers are busy.
2. Derive the constant ratio between successive state probabilities.
3. Understand why this produces a geometric progression.
4. Derive the general expression for the geometric tail.
5. Understand why utilization must remain below 100% for the queue to have a steady-state distribution.
6. Derive the total probability of all servers being busy.
7. Connect the geometric tail to the Erlang-C waiting probability.

The goal is to understand both the **mathematical derivation and the physical behavior it represents**.

---

## 2. Running Example: Database Connection Pool

We continue using our hypothetical database connection pool.

| Parameter | Symbol | Value |
|---|---|---|
| Number of connections | $c$ | 4 |
| Average arrival rate | $\lambda$ | 30 requests/sec |
| Mean connection hold time | $S$ | 100 ms |
| Service rate per connection | $\mu=1/S$ | 10 completions/sec |
| Offered load | $a=\lambda S$ | 3 Erlangs |
| Average utilization | $\rho=a/c$ | 75% |

The total service capacity is:

$$
c\mu=4\times10=40\text{ requests/sec}
$$

Our incoming workload is:

$$
\lambda=30\text{ requests/sec}
$$

Therefore:

$$
\rho=\frac{\lambda}{c\mu}
=\frac{30}{40}
=\boxed{0.75}
$$

We assume the standard M/M/c model:

- Poisson arrivals at a constant average rate.
- Independent, exponentially distributed service times.
- Four identical parallel connections.
- An unlimited first-come-first-served waiting queue.
- No request abandonment, cancellation, or timeout.
- Steady-state operation.

The condition $\rho<1$ ensures that a steady-state distribution exists for this idealized model.

---

## 3. Recap: State Probabilities Before Saturation

In Stage 3, we established the steady-state flow-balance relationship:

$$
\boxed{P_{n-1}\lambda=P_n\mu_n}
$$

Where:

- $P_n$ is the probability of finding exactly $n$ requests in the system.
- $\lambda$ is the arrival rate.
- $\mu_n$ is the aggregate completion rate in state $n$.

Rearranging:

$$
P_n=\frac{\lambda}{\mu_n}P_{n-1}
$$

Before all connections become busy:

$$
\mu_n=n\mu
$$

Therefore:

$$
P_n=\frac{\lambda}{n\mu}P_{n-1}
$$

Since:

$$
a=\frac{\lambda}{\mu}
$$

We obtain:

$$
P_n=\frac{a}{n}P_{n-1}
$$

Repeated substitution gives:

$$
\boxed{
P_n=\frac{a^n}{n!}P_0,\qquad 0\le n\le c
}
$$

For our example, $a=3$:

| State | Probability relative to $P_0$ |
|---|---|
| 0 | $P_0$ |
| 1 | $3P_0$ |
| 2 | $4.5P_0$ |
| 3 | $4.5P_0$ |
| 4 | $3.375P_0$ |

### Why Does the Factorial Appear?

As the system moves from state 1 to state 4, additional connections become busy.

The aggregate completion rate increases:

$$
\mu,\ 2\mu,\ 3\mu,\ 4\mu
$$

Each state introduces another factor into the denominator of the probability expression.

That produces the factorial:

$$
n!=1\times2\times3\times\cdots\times n
$$

**Key insight:** The factorial arises because the aggregate service-completion rate increases with the number of busy connections.

But this growth cannot continue indefinitely.

---

## 4. What Changes When All Connections Are Busy?

Consider states 4, 5, 6, and 7.

| State $n$ | Busy connections | Waiting requests | Completion rate |
|---|---|---|---|
| 4 | 4 | 0 | $4\mu=40$/sec |
| 5 | 4 | 1 | $4\mu=40$/sec |
| 6 | 4 | 2 | $4\mu=40$/sec |
| 7 | 4 | 3 | $4\mu=40$/sec |

Notice that the number of busy connections remains four.

Once all connections are occupied, additional requests must wait.

Therefore, the aggregate completion rate remains constant:

$$
\boxed{\mu_n=c\mu,\qquad n\ge c}
$$

The arrival rate also remains constant:

$$
\lambda=30/\text{sec}
$$

### Physical Intuition

Before saturation, additional requests can occupy idle connections and increase the number of concurrently executing requests.

After saturation, every connection is already busy.

Additional requests increase the queue length, but they do not increase the system's processing capacity.

**This is the fundamental reason the mathematical pattern changes beyond state $n=c$.**

---

## 5. Deriving $P_5$ from $P_4$

Consider the boundary between states 4 and 5.

```text
                       λ = 30/sec
             State 4  ------------>  State 5
                      <------------
                       4μ = 40/sec
```

State 4 contains:

- Four busy connections.
- Zero waiting requests.

State 5 contains:

- Four busy connections.
- One waiting request.

From steady-state flow balance:

$$
P_4\lambda=P_5(4\mu)
$$

Rearranging:

$$
P_5=\frac{\lambda}{4\mu}P_4
$$

Substitute the numerical values:

$$
P_5=\frac{30}{40}P_4
$$

Therefore:

$$
\boxed{P_5=0.75P_4}
$$

### Physical Interpretation

State 5 is 75% as likely as state 4.

This means:

$$
\frac{P_5}{P_4}=0.75
$$

It does **not** mean the probability of state 5 is 75%.

We are expressing one state probability relative to another.

The actual values of $P_4$ and $P_5$ will be calculated after normalization.

---

## 6. Deriving $P_6$ from $P_5$

Now consider the boundary between states 5 and 6.

The arrival rate remains:

$$
\lambda=30/\text{sec}
$$

The aggregate completion rate remains:

$$
4\mu=40/\text{sec}
$$

The steady-state flow-balance equation is:

$$
P_5\lambda=P_6(4\mu)
$$

Rearranging:

$$
P_6=\frac{\lambda}{4\mu}P_5
$$

Therefore:

$$
\boxed{P_6=0.75P_5}
$$

But we already know:

$$
P_5=0.75P_4
$$

Substituting:

$$
P_6=0.75(0.75P_4)
$$

Thus:

$$
\boxed{P_6=0.75^2P_4}
$$

### Why Is the Ratio Unchanged?

Two conditions remain constant:

1. The arrival rate $\lambda$ does not depend on the state.
2. The aggregate completion rate remains $c\mu$ because all connections are busy.

Therefore, the ratio between successive probabilities is the same.

**A constant arrival rate and a constant aggregate completion rate produce a constant ratio between neighboring state probabilities.**

---

## 7. The Geometric Progression

Continuing the same derivation:

$$
P_7=0.75P_6
$$

Substitute:

$$
P_7=0.75^3P_4
$$

Likewise:

$$
P_8=0.75^4P_4
$$

The resulting probability pattern is:

| State | Probability relative to $P_4$ |
|---|---|
| 4 | $P_4$ |
| 5 | $0.75P_4$ |
| 6 | $0.75^2P_4$ |
| 7 | $0.75^3P_4$ |
| 8 | $0.75^4P_4$ |
| 9 | $0.75^5P_4$ |

Each probability is obtained by multiplying the preceding probability by 0.75.

This is a **geometric progression**.

### What Is a Geometric Progression?

A geometric progression is a sequence in which successive terms have a constant ratio.

For example:

$$
1,\ 0.75,\ 0.75^2,\ 0.75^3,\ldots
$$

The common ratio is:

$$
r=0.75
$$

When the ratio is between zero and one, successive terms become smaller.

When the ratio exceeds one, successive terms become larger.

In our queueing model, the common ratio is the system utilization.

---

## 8. Deriving the General Geometric-Tail Formula

Recall the flow-balance recurrence:

$$
P_{n+1}=\frac{\lambda}{\mu_{n+1}}P_n
$$

For $n\ge c$, all connections are already busy.

Therefore:

$$
\mu_{n+1}=c\mu
$$

Substituting:

$$
P_{n+1}=\frac{\lambda}{c\mu}P_n
$$

Recall from Stage 1:

$$
a=\frac{\lambda}{\mu}
$$

And:

$$
\rho=\frac{a}{c}=\frac{\lambda}{c\mu}
$$

Therefore:

$$
\boxed{P_{n+1}=\rho P_n,\qquad n\ge c}
$$

Starting at state $c$:

$$
P_{c+1}=\rho P_c
$$

$$
P_{c+2}=\rho^2P_c
$$

$$
P_{c+3}=\rho^3P_c
$$

Continuing this pattern, for any integer $k\ge0$:

$$
\boxed{P_{c+k}=P_c\rho^k}
$$

This is the **general geometric-tail relationship for the M/M/c queue**.

### Physical Meaning of the Formula

- $P_c$ is the probability that exactly $c$ requests are present.
- $\rho$ is the average system utilization.
- $k$ is the number of additional requests beyond the number of available servers.

Since all $c$ connections are busy, these additional $k$ requests are waiting.

Therefore:

$$
P_{c+k}=P_c\rho^k
$$

describes how the probabilities decrease as the waiting queue becomes longer.

---

## 9. Why Must Utilization Be Below 100%?

We have established:

$$
P_{c+k}=P_c\rho^k
$$

Consider three scenarios.

### Scenario A: Utilization Below 100%

Suppose:

$$
\rho=0.75
$$

Then:

$$
P_{c+k}=P_c(0.75)^k
$$

As $k$ increases:

$$
0.75^k\rightarrow0
$$

Successively longer queues become less likely.

The geometric series converges, allowing the state probabilities to be normalized into a valid probability distribution.

### Scenario B: Utilization Equal to 100%

Suppose:

$$
\rho=1
$$

Then:

$$
P_{c+k}=P_c
$$

Every state beyond saturation would have the same probability weight.

The infinite sequence becomes:

$$
P_c+P_c+P_c+\cdots
$$

For any positive $P_c$, this sum diverges.

There is no way to normalize these positive state weights into a steady-state probability distribution.

Physically, incoming work equals the system's maximum average processing capacity.

There is no spare capacity to recover reliably from random fluctuations.

### Scenario C: Utilization Above 100%

Suppose:

$$
\rho=1.1
$$

Then:

$$
P_{c+k}=P_c(1.1)^k
$$

Successive terms increase rather than decrease.

For example:

$$
P_{c+1}=1.1P_c
$$

$$
P_{c+2}=1.21P_c
$$

$$
P_{c+3}=1.331P_c
$$

These cannot form a normalizable steady-state distribution.

Physically, incoming requests exceed the system's maximum average processing capacity, so the backlog grows without bound over time in the unlimited-queue model.

### The Stability Constraint

For the geometric series to converge, we require:

$$
0\le\rho<1
$$

Since:

$$
\rho=\frac{a}{c}
$$

The stability requirement becomes:

$$a < c$$

### Key Insight

**The queueing-theory stability constraint is not an arbitrary mathematical restriction. It reflects the physical requirement that processing capacity must exceed incoming workload for a stable unlimited-queue system.**

Even when $\rho<1$, operating very close to 100% utilization can produce substantial queueing delays.

Stability does not guarantee acceptable performance.

---

## 10. Understanding the Infinite Geometric Series

Now that we know the state probabilities form a geometric progression, we want to calculate their total probability.

First, consider the mathematical series:

$$
1+\rho+\rho^2+\rho^3+\cdots
$$

For $0\le\rho<1$, this is a convergent infinite geometric series.

Its sum is:

$$
\boxed{\sum_{k=0}^{\infty}\rho^k=\frac{1}{1-\rho}}
$$

### Deriving the Geometric-Series Formula

Let:

$$
G=1+\rho+\rho^2+\rho^3+\cdots
$$

Multiply both sides by $\rho$:

$$
\rho G=\rho+\rho^2+\rho^3+\rho^4+\cdots
$$

Now subtract:

$$G-\rho G = (1+\rho+\rho^2+\cdots) - (\rho+\rho^2+\rho^3+\cdots)$$

All terms except the initial 1 cancel:

$$
G-\rho G=1
$$

Factor:

$$
G(1-\rho)=1
$$

Therefore:

$$
\boxed{G=\frac{1}{1-\rho}}
$$

For our example:

$$
\rho=0.75
$$

Therefore:

$$
G=\frac{1}{1-0.75}
$$

$$
G=\frac{1}{0.25}
$$

$$
\boxed{G=4}
$$

### Numerical Interpretation

The infinite sum is:

$$
1+0.75+0.5625+0.421875+\cdots=4
$$

Although infinitely many terms are present, their total is finite because each successive term becomes smaller.

This result will allow us to sum the entire geometric tail of the M/M/c state distribution.

---

## 11. Probability That All Connections Are Busy

We now arrive at an important connection between the geometric tail and connection-pool performance.

Suppose a request arrives at our four-connection pool.

The request must wait whenever all four connections are busy.

That happens in any state satisfying:

$$
n\ge4
$$

For example:

| State | Busy connections | Waiting requests | New arrival waits? |
|---|---|---|---|
| 0 | 0 | 0 | No |
| 1 | 1 | 0 | No |
| 2 | 2 | 0 | No |
| 3 | 3 | 0 | No |
| 4 | 4 | 0 | Yes |
| 5 | 4 | 1 | Yes |
| 6 | 4 | 2 | Yes |
| 7 | 4 | 3 | Yes |

Therefore, we want to calculate:

$$
P(N\ge c)
$$

This is the probability that all connections are busy, including states where requests are already waiting.

Because the states are mutually exclusive:

$$
\boxed{
P(N\ge c)=P_c+P_{c+1}+P_{c+2}+\cdots
}
$$

### Why Do We Include Every State Beyond $c$?

The condition is not that the system contains exactly $c$ requests.

The condition is that **all $c$ connections are occupied**.

That condition holds in states $c$, $c+1$, $c+2$, and every higher state.

Therefore, we add the probabilities of all states satisfying that condition.

This is an application of the probability addition rule for mutually exclusive events.

---

## 12. Summing the Geometric Tail

We already derived:

$$
P_{c+k}=P_c\rho^k
$$

Therefore:

$$P(N\ge c) = P_c+P_c\rho+P_c\rho^2+\cdots$$

Factor out $P_c$:

$$P(N\ge c)=P_c(1+\rho+\rho^2+\cdots)$$

We have just derived:

$$
1+\rho+\rho^2+\cdots=\frac{1}{1-\rho}
$$

Substituting:

$$
\boxed{
P(N\ge c)=\frac{P_c}{1-\rho}
}
$$

This is the total probability of all states where every server is busy.

### Applying It to Our Example

Recall:

$$
c=4,\qquad\rho=0.75
$$

Therefore:

$$
P(N\ge4)=\frac{P_4}{1-0.75}
$$

$$
P(N\ge4)=\frac{P_4}{0.25}
$$

Thus:

$$
\boxed{P(N\ge4)=4P_4}
$$

### Physical Interpretation

The probability of finding all four connections busy is four times the probability of finding exactly four requests in the system.

Why?

Because the busy condition also includes states 5, 6, 7, and every higher state.

Those additional state probabilities collectively contribute another $3P_4$.

**This does not mean the probability is 400%.**

It means the total busy-state probability is four times $P_4$.

We must still determine the actual numerical value of $P_4$.

That will require probability normalization in Stage 5.

---

## 13. Connecting the Geometric Tail to Erlang-C

We have established:

$$
P(N\ge c)=\frac{P_c}{1-\rho}
$$

This represents the probability that all servers are busy at a randomly selected instant in steady state.

But Erlang-C asks a related question:

**What is the probability that an arriving request must wait for service?**

For a Poisson arrival process, arrivals observe the same steady-state state probabilities as random-time observations.

This property is known as **PASTA — Poisson Arrivals See Time Averages**.

Therefore:

$$
P(\text{arrival must wait})=P(N\ge c)
$$

We obtain:

$$
\boxed{
P(\text{wait})=\frac{P_c}{1-\rho}
}
$$

This is an important intermediate result in the derivation of Erlang-C.

However, we do not yet know $P_c$ numerically.

From Stage 3:

$$
P_c=\frac{a^c}{c!}P_0
$$

Substituting:

$$P(\text{wait})=\frac{a^c}{c!}\frac{1}{1-\rho}P_0$$

Therefore:

$${ P(\text{wait})=\frac{a^c}{c!}\frac{P_0}{1-\rho} }$$

This expresses the waiting probability in terms of the still-unknown empty-system probability $P_0$.

In Stage 5, we will calculate $P_0$ using probability normalization.

In Stage 6, we will assemble these relationships into the complete Erlang-C formula.

---

## 14. Probability of Waiting vs. Probability of Not Waiting

It is important to distinguish the two complementary events.

### Probability of Not Waiting

An arriving request does not wait if at least one connection is available.

For our four-connection pool, this occurs in states:

$$
0,\ 1,\ 2,\ 3
$$

Therefore:

$$
\boxed{
P(\text{no wait})=P_0+P_1+P_2+P_3
}
$$

### Probability of Waiting

An arriving request must wait if all connections are busy.

This occurs in states:

$$
4,\ 5,\ 6,\ldots
$$

Therefore:

$$
\boxed{
P(\text{wait})=P_4+P_5+P_6+\cdots
}
$$

These two events are mutually exclusive and collectively exhaustive.

Thus:

$$
\boxed{
P(\text{no wait})+P(\text{wait})=1
}
$$

### Important Distinction: Waiting Probability vs. Normalization

**Probability normalization** requires the sum of the probabilities of every possible state to equal one:

$$
P_0+P_1+P_2+P_3+P_4+P_5+\cdots=1
$$

**Probability of waiting** includes only the states where all connections are busy:

$$
P(\text{wait})=P_4+P_5+P_6+\cdots
$$

These are related but different calculations.

Normalization considers all possible states.

Waiting probability considers only the subset of states satisfying the condition that every connection is occupied.

---

## 15. Worked Example: Consolidating the Derivation

Consider our running example:

$$
c=4,\quad\lambda=30/\text{sec},\quad\mu=10/\text{sec}
$$

### Step 1: Calculate Offered Load

$$
a=\frac{\lambda}{\mu}=\frac{30}{10}
$$

$$
\boxed{a=3}
$$

### Step 2: Calculate Utilization

$$
\rho=\frac{a}{c}=\frac34
$$

$$
\boxed{\rho=0.75}
$$

### Step 3: Recall the State-4 Probability

From Stage 3:

$$
P_4=\frac{a^4}{4!}P_0
$$

Therefore:

$$
P_4=\frac{3^4}{24}P_0
$$

$$
\boxed{P_4=3.375P_0}
$$

### Step 4: Derive the Geometric Tail

Since:

$$
P_{n+1}=0.75P_n,\qquad n\ge4
$$

We obtain:

$$
P_5=0.75P_4
$$

$$
P_6=0.75^2P_4
$$

$$
P_7=0.75^3P_4
$$

And generally:

$$
\boxed{P_{4+k}=P_4(0.75)^k}
$$

### Step 5: Sum the Busy-State Probabilities

$$
P(N\ge4)
=
P_4(1+0.75+0.75^2+\cdots)
$$

The geometric series evaluates to:

$$
\frac{1}{1-0.75}=4
$$

Therefore:

$$
P(N\ge4)=4P_4
$$

### Step 6: Express the Result in Terms of $P_0$

Since:

$$
P_4=3.375P_0
$$

Then:

$$
P(N\ge4)=4(3.375P_0)
$$

Thus:

$$
\boxed{P(N\ge4)=13.5P_0}
$$

Because Poisson arrivals see time averages:

$$
\boxed{P(\text{wait})=13.5P_0}
$$

We have now expressed the waiting probability entirely in terms of $P_0$.

The actual value of $P_0$ will be derived in Stage 5.

---

## 16. Practical Performance Engineering Implications

The geometric tail reveals several important characteristics of capacity-constrained systems.

### 16.1 Queue Lengths Are Probabilistic

Even when average utilization is below 100%, the queue can temporarily become long because arrivals and completions occur randomly.

A stable system does not necessarily have an empty queue.

### 16.2 Higher Utilization Produces a More Slowly Decaying Tail

Consider two utilization levels:

$$
\rho=0.5
$$

And:

$$
\rho=0.9
$$

For the first system:

$$
P_{c+k}=P_c(0.5)^k
$$

For the second:

$$
P_{c+k}=P_c(0.9)^k
$$

The second system's relative state probabilities decrease much more slowly.

Therefore, long queues become more prominent as utilization approaches 100%.

This is one reason heavily utilized systems can experience significant queueing delays despite remaining mathematically stable.

### 16.3 Capacity Headroom Matters

Increasing the number of available servers while keeping workload unchanged reduces utilization:

$$
\rho=\frac{\lambda S}{c}
$$

A smaller utilization produces a faster-decaying geometric tail.

This can improve queueing behavior without changing the incoming workload.

### 16.4 Analytical Predictions Depend on Model Assumptions

The geometric relationship is derived from the M/M/c assumptions.

Real systems may exhibit:

- Bursty or correlated arrivals.
- Heavy-tailed service times.
- Variable service rates.
- Resource contention.
- Finite queues.
- Request timeouts or abandonment.
- Priority-aware scheduling.

These behaviors may cause observed queueing distributions to differ from the idealized geometric tail.

The analytical model remains useful for building intuition and establishing a baseline for experiments.

---

## 17. Key Takeaways

1. **Before saturation**, the aggregate completion rate increases with the number of busy servers, producing factorial terms in the state probabilities.

2. **After saturation**, the aggregate completion rate remains constant because all servers are occupied.

3. The arrival rate also remains constant under the M/M/c model.

4. Consequently, the ratio between successive state probabilities becomes constant:

   $$
   \frac{P_{n+1}}{P_n}=\rho,\qquad n\ge c
   $$

5. This produces the geometric tail:

   $$
   \boxed{P_{c+k}=P_c\rho^k}
   $$

6. A stable unlimited-queue system requires $\rho<1$ so the geometric state probabilities can be normalized.

7. The probability of all servers being busy includes every state satisfying $n\ge c$, not just state $c$.

8. The total probability of these busy states is:

   $$
   \boxed{P(N\ge c)=\frac{P_c}{1-\rho}}
   $$

9. Under Poisson arrivals, this also represents the probability that an arriving request must wait.

10. We have now expressed the waiting probability in terms of $P_0$, but we still need probability normalization to calculate its actual value.

### Central Results of Stage 4

**Geometric tail:**

$$
\boxed{P_{c+k}=P_c\rho^k,\qquad k\ge0}
$$

**Total probability of all servers being busy:**

$$
\boxed{P(N\ge c)=\frac{P_c}{1-\rho}}
$$

**Waiting probability under Poisson arrivals:**

$$
\boxed{P(\text{wait})=\frac{P_c}{1-\rho}}
$$

---

## 18. Next Stage

**[05 — Probability Normalization](05-probability-normalization.md)**

We have derived all state probabilities relative to $P_0$.

For states below saturation:

$$
P_n=\frac{a^n}{n!}P_0
$$

For states at or above saturation:

$$
P_{c+k}=P_c\rho^k
$$

The next task is to determine the actual value of $P_0$.

We will do this using the fundamental probability requirement:

$$
\boxed{\sum_{n=0}^{\infty}P_n=1}
$$

We will divide the possible states into two groups:

1. States where at least one server is available.
2. States where every server is busy.

By combining the factorial terms from Stage 3 with the geometric tail from Stage 4, we will derive the normalization constant $P_0$.

This will bring us one step closer to assembling the complete Erlang-C waiting probability.