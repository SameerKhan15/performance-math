# Queueing Theory for Performance Engineering

## Overview

This module develops queueing theory from first principles, with an emphasis on practical applications in distributed systems and performance engineering.

The objective is to understand not only how to apply queueing formulas, but also **how they are derived, what assumptions they depend on, and how to interpret their results when making architectural and capacity-planning decisions**.

We begin with the M/M/c queueing model and derive Erlang-C, using database connection pools as our primary running example.

## Learning Objectives

By completing this module, we aim to develop the ability to:

1. Model arrival rates, service times, concurrency, utilization, and queue lengths.
2. Understand how requests transition between queueing system states.
3. Derive steady-state probabilities using flow balance.
4. Explain why state probabilities develop a geometric tail once all servers are busy.
5. Derive the Erlang-C waiting probability using probability normalization.
6. Derive the exponential waiting-time distribution and checkout timeout probability.
7. Apply queueing theory to connection-pool sizing and performance analysis.
8. Understand the assumptions and limitations of analytical queueing models.

## Running Example: Database Connection Pool

We use a database connection pool with the following parameters:

| Parameter | Symbol | Value |
|---|---|---|
| Number of connections | $c$ | 4 |
| Request arrival rate | $\lambda$ | 30 requests/sec |
| Mean connection hold time | $S$ | 100 ms |
| Service rate per connection | $\mu = 1/S$ | 10 requests/sec |
| Offered load | $a = \lambda S$ | 3 Erlangs |
| Average utilization | $\rho = a/c$ | 75% |

Each connection services one request at a time. When all connections are occupied, additional requests wait in a queue.

We initially assume an M/M/c model:

- **M:** Poisson request arrivals (exponentially distributed interarrival times).
- **M:** Exponentially distributed connection holding times.
- **c:** A fixed number of parallel connections.
- Unlimited waiting capacity, first-come-first-served service, and steady-state operation with $\rho < 1$.

These assumptions allow us to derive tractable analytical results. Real-world workloads may violate them.

## Learning Path

| Stage | Topic | Notes |
|---|---|---|
| 1 | Queueing fundamentals | [1-queueing-fundamentals.md](1-queueing-fundamentals.md) |
| 2 | State-transition model | [2-state-transition-model.md](2-state-transition-model.md) |
| 3 | Steady-state flow balance | [3-steady-state-flow-balance.md](3-steady-state-flow-balance.md) |
| 4 | Geometric tail | [4-geometric-tail.md](4-geometric-tail.md) |
| 5 | Probability normalization | [5-probability-normalization.md](5-probability-normalization.md) |
| 6 | Erlang-C derivation | [6-erlang-c-derivation.md](6-erlang-c-derivation.md) |
| 7 | Waiting-time distribution | [7-wait-time-distribution.md](7-wait-time-distribution.md) |
| 8 | Connection-pool sizing | [8-connection-pool-sizing.md](8-connection-pool-sizing.md) |

Each stage builds on the preceding stages. The emphasis is on deriving each result rather than memorizing formulas.

## Key Mathematical Relationships

The following relationships will be derived and explained throughout the module.

**Offered load**

$$
a = \lambda S = \frac{\lambda}{\mu}
$$

**Utilization**

$$
\rho = \frac{a}{c}
$$

**Aggregate completion rate in state $n$**

$$
\mu_n = \min(n,c)\mu
$$

**Erlang-C waiting probability**

$$
C(c,a) =
\frac{
\frac{a^c}{c!}\frac{c}{c-a}
}{
\displaystyle\sum_{n=0}^{c-1}\frac{a^n}{n!}
+
\frac{a^c}{c!}\frac{c}{c-a}
}
$$

**Probability of exceeding a connection checkout timeout**

$$
P(W_q > T) = C(c,a)e^{-(c-a)T/S}
$$

These formulas are valid under the applicable M/M/c assumptions, with $a<c$. The timeout expression describes the waiting-time tail in the idealized unlimited-queue model.

## Performance Engineering Applications

The mathematical concepts apply to:

- **Database connection pools:** Connection checkout delays, timeouts, pool fragmentation, and shared connection tiers.
- **Thread and worker pools:** Concurrency sizing, resource utilization, and queueing delays.
- **Admission control:** Preventing overload and protecting latency-sensitive workloads.
- **Distributed services:** Capacity planning, bottleneck identification, and tail-latency analysis.
- **AI inference serving:** Request concurrency, scheduling, throughput, and latency objectives.

## Guiding Principle

**The goal is not to memorize queueing formulas. It is to understand the underlying system behavior well enough to reconstruct the formulas, recognize their limitations, and apply them correctly to real performance problems.**

## Reference Architecture

The primary motivating case study examines the performance and scalability implications of **database connection-pool fragmentation**, where connections are partitioned across application instances. This can lead to connection checkout delays and timeouts even when aggregate database connection capacity remains available. The case study explores how shared connection pooling and multiplexing can improve resource utilization, reduce queueing delays, and increase system resilience.

Its Erlang-C analysis provides a concrete application of the mathematical model developed in this module.
