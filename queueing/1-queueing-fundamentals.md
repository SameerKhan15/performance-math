# 01 — Queueing Fundamentals

## 1. Objective

Develop a foundational understanding of the quantities that describe a queueing system and the mathematical relationships between them.

The emphasis is on understanding the **physical meaning of each quantity**, rather than simply memorizing formulas.

By the end of this module, we should be able to:

- Distinguish arrival rate, service time, and service rate.
- Calculate offered load and average utilization.
- Understand the relationship between utilization and capacity headroom.
- Distinguish queue waiting time from service time.
- Apply Little's Law to calculate average concurrency and queue length.
- Understand the conditions required for a stable queueing system.

---

## 2. Running Example: Database Connection Pool

Consider a hypothetical database connection pool with the following characteristics:

| Parameter | Symbol | Value |
|---|---|---|
| Number of connections | $c$ | 4 |
| Request arrival rate | $\lambda$ | 30 requests/sec |
| Mean connection hold time | $S$ | 100 ms |
| Service rate per connection | $\mu$ | 10 requests/sec |
| Offered load | $a$ | 3 Erlangs |
| Average utilization | $\rho$ | 75% |

Each database connection can service one request at a time.

When all connections are occupied, additional requests wait until a connection becomes available.

For our initial mathematical model, we assume a stable, lossless queueing system with a constant number of parallel connections.

---

## 3. Arrival Rate ($\lambda$)

Arrival rate measures how frequently new requests enter the queueing system.

$$
\lambda = \frac{\text{Number of arriving requests}}{\text{Elapsed time}}
$$

For example:

$$
\lambda = 30 \text{ requests/sec}
$$

This means an average of 30 new requests arrive every second.

**Important:** Arrival rate describes incoming demand. It does not necessarily equal the rate at which requests complete.

In a stable, lossless system, long-run throughput equals the arrival rate.

---

## 4. Mean Service Time ($S$)

Service time measures how long a request occupies a server while being processed.

In our database connection-pool example, this is the **connection hold time**.

Suppose a request:

- Waits 40 ms to obtain a connection.
- Holds the connection for 100 ms.
- Returns the connection.

The service time of that request is 100 ms, not 140 ms.

Mean service time is the average service duration across requests:

$$
S = E[\text{Service Time}]
$$

For our example:

$$
S = 100 \text{ ms} = 0.1 \text{ seconds}
$$

Service time does not include time spent waiting for an available connection.

Furthermore, connection hold time is not necessarily equivalent to database CPU execution time. A request may hold a connection while waiting for database I/O, locks, or other operations.

---

## 5. Service Rate ($\mu$)

Service rate represents the average number of requests that a single continuously busy server can complete per unit time.

It is the reciprocal of mean service time:

$$
\boxed{\mu = \frac{1}{S}}
$$

For a mean service time of 100 ms:

$$
\mu = \frac{1}{0.1}
$$

$$
\boxed{\mu = 10 \text{ requests/sec}}
$$

Therefore, each connection can complete an average of 10 requests per second when continuously occupied, assuming its mean service time remains unchanged.

### Aggregate Service Capacity

With $c$ parallel connections:

$$
\text{Aggregate service capacity} = c\mu
$$

For four connections:

$$
4 \times 10 = \boxed{40 \text{ requests/sec}}
$$

This is the theoretical aggregate completion rate when all four connections remain continuously busy.

It assumes that sufficient work is available and that concurrent execution does not increase service times through resource contention.

### Key Intuition

**Service time measures how long one request occupies a server. Service rate measures how frequently that server can complete requests.**

A shorter mean service time implies a higher service rate.

---

## 6. Offered Load ($a$)

Offered load represents the average amount of server capacity required by incoming requests.

It is calculated as:

$$
\boxed{a = \lambda S = \frac{\lambda}{\mu}}
$$

For our example:

$$
a = 30 \times 0.1
$$

$$
\boxed{a = 3 \text{ Erlangs}}
$$

### Physical Interpretation

The arrival rate is:

$$
30 \frac{\text{requests}}{\text{second}}
$$

Each request requires, on average:

$$
0.1 \frac{\text{seconds of service}}{\text{request}}
$$

Multiplying:

$$
30 \frac{\text{requests}}{\text{second}}
\times
0.1 \frac{\text{seconds}}{\text{request}}
= 3
$$

The units cancel, producing a dimensionless quantity measured in Erlangs.

**Three Erlangs means the incoming workload requires an average of three server-seconds of processing capacity per second.**

In a stable, lossless system, this corresponds to an average of three busy servers.

This does not mean exactly three servers are busy at every instant.

At different moments, we might observe:

- Two busy connections.
- Four busy connections.
- Three busy connections.

The long-run average is three busy connections.

### Important Distinction

Offered load measures demand for service capacity.

It does not include requests waiting in the queue.

In a stable, lossless system:

$$
\boxed{\text{Average busy servers} = a = \lambda S}
$$

---

## 7. Utilization ($\rho$)

Utilization measures the fraction of total server capacity consumed by the workload on average.

$$
\boxed{\rho = \frac{a}{c} = \frac{\lambda S}{c}}
$$

For our four-connection pool:

$$
\rho = \frac{3}{4}
$$

$$
\boxed{\rho = 0.75 = 75\%}
$$

### Physical Interpretation

The system requires, on average, three busy connections out of four available connections.

Therefore, average utilization is 75%.

Equivalently, each connection is busy approximately 75% of the time, assuming statistically balanced utilization across connections.

### Increasing Capacity

Suppose we increase the pool size from four to six connections while keeping arrival rate and service time unchanged.

Offered load remains:

$$
a = 3
$$

But utilization becomes:

$$
\rho = \frac{3}{6} = 50\%
$$

| Metric | 4 connections | 6 connections |
|---|---|---|
| Offered load | 3 Erlangs | 3 Erlangs |
| Average busy connections | 3 | 3 |
| Total connections | 4 | 6 |
| Average utilization | 75% | 50% |
| Average idle connections | 1 | 3 |

**Key insight:** Increasing capacity does not reduce the amount of work arriving. It distributes the same workload across more available resources, reducing utilization and increasing headroom.

---

## 8. Queue Waiting Time, Service Time, and Response Time

A request can spend time in two distinct phases.

**Queue waiting time ($W_q$):** Time spent waiting for an available server.

**Service time ($S$):** Mean time occupying a server.

**Total response time ($W$):** Mean time spent in the modeled queueing system, including both waiting and service.

The relationship is:

$$
\boxed{W = W_q + S}
$$

For example:

| Component | Duration |
|---|---|
| Queue waiting time | 40 ms |
| Service time | 100 ms |
| Total response time | 140 ms |

Therefore:

$$
W = 40 + 100 = \boxed{140 \text{ ms}}
$$

For a database connection-pool model, this measures time spent waiting for and holding a connection. It does not necessarily represent the complete application request latency.

---

## 9. Little's Law

Little's Law establishes a fundamental relationship between throughput, concurrency, and time.

$$
\boxed{L = \lambda W}
$$

Where:

- $L$ = average number of requests in the system.
- $\lambda$ = average effective arrival rate or throughput.
- $W$ = average time a request spends in the system.

### Physical Intuition

If requests enter a system frequently, more requests will be present concurrently.

Similarly, if each request remains in the system longer, concurrency increases.

Therefore:

**Average concurrency is determined by throughput multiplied by average time in the system.**

### Example

Suppose:

$$
\lambda = 20 \text{ requests/sec}
$$

$$
W = 0.2 \text{ seconds}
$$

Then:

$$
L = 20 \times 0.2
$$

$$
\boxed{L = 4 \text{ requests}}
$$

The system contains an average of four requests over time.

Little's Law is a general conservation relationship. It does not require Poisson arrivals or exponentially distributed service times. It does require consistent system boundaries and appropriate long-run stability assumptions.

---

## 10. Applying Little's Law to Different Parts of a System

We can apply Little's Law independently to the waiting queue and the service portion of the system.

### 10.1 Requests Holding Connections

$$
\boxed{L_s = \lambda S}
$$

Where $L_s$ represents the average number of requests actively holding connections.

For our example:

$$
L_s = 30 \times 0.1 = 3
$$

This is exactly the offered load:

$$
\boxed{L_s = a}
$$

### 10.2 Requests Waiting in the Queue

$$
\boxed{L_q = \lambda W_q}
$$

Suppose the measured average queue waiting time is 50 ms.

Then:

$$
L_q = 30 \times 0.05
$$

$$
\boxed{L_q = 1.5}
$$

The queue contains an average of 1.5 waiting requests.

This is a hypothetical waiting-time measurement, not necessarily the waiting time predicted by an M/M/4 model at 75% utilization.

### 10.3 Total Requests in the System

The total number of requests includes those waiting and those being serviced.

$$
\boxed{L = L_q + L_s}
$$

Therefore:

$$
L = 1.5 + 3
$$

$$
\boxed{L = 4.5}
$$

We can verify using Little's Law directly:

$$
W = W_q + S = 0.05 + 0.10 = 0.15
$$

$$
L = 30 \times 0.15 = \boxed{4.5}
$$

### Key Distinction

**Offered load represents average service concurrency, whereas $L$ represents total system concurrency, including queued requests.**

These are different quantities and should not be used interchangeably.

---

## 11. Understanding the Stability Constraint ($a < c$)

For the standard M/M/c queue with unlimited waiting capacity, a steady-state probability distribution exists only when:

$$
\boxed{a < c}
$$

Equivalently:

$$
\boxed{\rho < 1}
$$

### Physical Intuition

The fundamental requirement for a stable queueing system is that **incoming workload must remain below the system's maximum aggregate processing capacity**.

Consider our four-connection pool:

- Each connection can process an average of 10 requests/sec.
- Four connections provide an aggregate capacity of 40 requests/sec.

| Arrival rate | Offered load | Utilization | Interpretation |
|---|---|---|---|
| 30 req/s | 3 Erlangs | 75% | Stable |
| 40 req/s | 4 Erlangs | 100% | Critical load; no steady-state distribution |
| 50 req/s | 5 Erlangs | 125% | Overloaded |

At 30 requests/sec, the system has enough spare capacity to eventually drain temporary backlogs.

At 40 requests/sec, incoming demand equals maximum processing capacity. There is no spare capacity to recover from random fluctuations.

At 50 requests/sec, requests accumulate faster than the system can process them. In an unlimited-queue model without dropping requests, the backlog grows without bound.

### Why Queueing Becomes Severe Near Saturation

Consider two workloads on the same 40 requests/sec capacity.

**Scenario A: 30 requests/sec**

Spare processing capacity:

$$
40 - 30 = 10 \text{ requests/sec}
$$

**Scenario B: 39 requests/sec**

Spare processing capacity:

$$
40 - 39 = 1 \text{ request/sec}
$$

Suppose a temporary burst produces a backlog of 100 requests.

Under a simplified fluid approximation, assuming full service capacity remains available and arrivals continue at the stated rate:

- Scenario A can drain the backlog in approximately 10 seconds.
- Scenario B can take approximately 100 seconds.

**As utilization approaches 100%, the system loses its ability to recover quickly from temporary fluctuations in demand.**

Mathematically:

$$
c-a=c(1-\rho)
$$

As:

$$
\rho \rightarrow 1
$$

We have:

$$
c-a \rightarrow 0
$$

The available concurrency headroom approaches zero.

### Stability Does Not Guarantee Good Performance

A system at 95% utilization may be mathematically stable but still experience severe queueing delays.

Therefore:

**Capacity planning must account for both stability and sufficient headroom to meet latency objectives.**

The mathematical reason the M/M/c model requires $\rho<1$ will become clearer when we derive its geometric state-probability tail in Stage 4.

---

## 12. Worked Example: Consolidating the Fundamentals

Consider a connection pool with:

| Parameter | Value |
|---|---|
| Arrival rate | 20 requests/sec |
| Mean connection hold time | 200 ms |
| Number of connections | 5 |
| Average queue waiting time | 50 ms |

### Offered Load

$$
a=\lambda S
$$

$$
a=20\times0.2=\boxed{4\text{ Erlangs}}
$$

### Utilization

$$
\rho=\frac{a}{c}
$$

$$
\rho=\frac45=\boxed{80\%}
$$

### Average Queue Length

$$
L_q=\lambda W_q
$$

$$
L_q=20\times0.05=\boxed{1}
$$

### Average Number of Busy Connections

$$
L_s=\lambda S
$$

$$
L_s=20\times0.2=\boxed{4}
$$

### Total Requests in the System

$$
L=L_q+L_s
$$

$$
L=1+4=\boxed{5}
$$

Alternatively:

$$
W=0.05+0.2=0.25\text{ seconds}
$$

$$
L=20\times0.25=\boxed{5}
$$

Both approaches produce the same result.

---

## 13. Key Takeaways

1. **Arrival rate** describes how quickly requests enter the system.
2. **Service time** describes how long each request occupies a server.
3. **Service rate** is the reciprocal of mean service time.
4. **Offered load** measures average demand for server capacity in Erlangs.
5. **Utilization** compares offered load with total available server capacity.
6. **Little's Law** connects throughput, average concurrency, and time spent in the system.
7. **Average busy servers and average total requests are not necessarily equal**, because some requests may be waiting.
8. **A stable system can still have unacceptable latency**, particularly as utilization approaches 100%.
9. **Queueing models are most useful when their assumptions and system boundaries are clearly understood.**

---

## 14. Next Stage

**[02 — State-Transition Model](02-state-transition-model.md)**

We will model the connection pool as a sequence of states, where each state represents the total number of requests in the system.

We will establish how arrival and completion events move the system between states and derive the aggregate completion rate:

$$
\mu_n=\min(n,c)\mu
$$

This provides the foundation for deriving steady-state probabilities and, ultimately, Erlang-C.