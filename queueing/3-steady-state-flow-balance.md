# 03 — Steady-State Flow Balance

## 1. Objective

Derive the **relative probabilities of the states in an M/M/c queue** using steady-state flow balance, without memorizing a probability formula.

By the end of this stage, we should understand:

- What a steady-state probability $P_n$ represents.
- Why we multiply a transition rate by the probability of occupying its source state.
- Why probability flows across neighboring states must balance.
- How to derive $P_1$, $P_2$, $P_3$, and $P_4$ relative to $P_0$.
- Why a factorial appears in the state probabilities up to the number of servers $c$.
- Why this factorial pattern changes when all servers are busy.

We will **not yet calculate the actual value of $P_0$**. That requires probability normalization, which comes in Stage 5.

---

## 2. Running Example: A Four-Connection Pool

Consider a hypothetical database connection pool:

| Quantity | Symbol | Value |
|---|---|---:|
| Parallel connections | $c$ | 4 |
| Average arrival rate | $\lambda$ | 30 requests/sec |
| Mean connection hold time | $S$ | 100 ms = 0.1 sec |
| Service rate of one busy connection | $\mu=1/S$ | 10 completions/sec |
| Offered load | $a=\lambda S=\lambda/\mu$ | 3 Erlangs |
| Utilization | $\rho=a/c$ | 75% |

We model the pool as an **M/M/4 queue**: Poisson arrivals, independent exponentially distributed holding times, four identical servers, an unlimited first-come-first-served queue, and no abandonment. Since $a<c$, the idealized model has a steady-state distribution.

A state $n$ is the **total number of requests in the system**, including both requests holding a connection and requests waiting for one.

For example:

| State $n$ | Busy connections | Waiting requests |
|---|---:|---:|
| 0 | 0 | 0 |
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 3 | 0 |
| 4 | 4 | 0 |
| 5 | 4 | 1 |

From Stage 2, the aggregate service-completion rate in state $n$ is:

$$
\boxed{\mu_n=\min(n,c)\mu}
$$

Therefore, in our example, $\mu_1=10$, $\mu_2=20$, $\mu_3=30$, and $\mu_4=40$ completions/sec.

---

## 3. What Does $P_n$ Mean?

Define:

$$
\boxed{P_n=P(N=n)}
$$

Here $N$ is the number of requests in the system when we observe it at a randomly selected instant **in steady state**.

Two equivalent interpretations are useful:

1. **Random-instant probability:** $P_0$ is the probability that the system is empty when observed at a randomly chosen instant.
2. **Long-run fraction of time:** $P_0$ is the fraction of a sufficiently long observation period that the system spends empty.

For example, if $P_0=0.05$, the system spends approximately 5% of its time empty. Over an hour this corresponds to about 180 seconds **in total**, not necessarily one uninterrupted 180-second interval.

### Why can the system be empty despite 30 arrivals/sec?

$\lambda=30$/sec is an **average arrival intensity**, not a schedule requiring precisely 30 arrivals in every second. With Poisson arrivals, the number of arrivals in a given interval varies, and gaps between arrivals are random.

It is possible for the outstanding requests to finish during a sufficiently long gap, leaving the system in state 0 for a brief interval. Even a 5 ms empty interval contributes to the time fraction $P_0$.

**Do not confuse** $P_0$ with the probability that zero arrivals occur over an entire second. Those are different events.

Also distinguish:

- $P_n$: probability of **being in state $n$**.
- $P(\text{arrival next}\mid n)$: probability of a particular **next transition** given the current state.

The latter was covered in Stage 2. This chapter derives the former.

---

## 4. Physical Intuition Behind Flow Balance

Consider the boundary between state 0 and state 1:

```text
                  arrival: λ = 30/sec
        State 0  -------------------->  State 1
                 <--------------------
                  completion: μ = 10/sec
```

An arrival moves the system from 0 to 1. A completion moves it from 1 to 0.

We **cannot simply equate** $\lambda=30$ and $\mu=10$. Each transition rate applies only while its corresponding source state is occupied.

If the system is empty 10% of the time, the rate of 0 → 1 transitions **averaged over total elapsed time** is:

$$
P_0\lambda=0.10(30)=3\text{ transitions/sec}
$$

If the system spends 30% of its time in state 1, the reverse rate averaged over total elapsed time is:

$$
P_1\mu=0.30(10)=3\text{ transitions/sec}
$$

This explains the distinction:

- $\lambda$ is the arrival **intensity while the transition is possible**.
- $P_0\lambda$ is the **long-run frequency** of transitions from 0 to 1, measured across all elapsed time.

The same interpretation applies to $P_1\mu$.

### Why do the flows balance?

In a steady-state birth-death system, the long-run frequency of crossing a boundary in one direction equals the long-run frequency of crossing it in the other direction. Otherwise, there would be a persistent net movement across that boundary rather than a stationary state distribution.

Therefore:

$$
\boxed{P_0\lambda=P_1\mu}
$$

This is **local (or detailed) balance**. It does not mean that an arrival and a completion happen at the same instant. It means their long-run, probability-weighted transition frequencies match.

The earlier $P_0=0.10$ and $P_1=0.30$ values illustrate this one boundary; they are **not asserted to be the actual steady-state probabilities** of the entire four-connection system.

---

## 5. Deriving $P_1$ Relative to $P_0$

Start from local balance:

$$
P_0\lambda=P_1\mu
$$

Solve for $P_1$:

$$
P_1=\frac{\lambda}{\mu}P_0
$$

Recall the offered load:

$$
a=\frac{\lambda}{\mu}=\lambda S
$$

Thus:

$$
\boxed{P_1=aP_0}
$$

For our running example, $a=3$:

$$
\boxed{P_1=3P_0}
$$

**Interpretation:** The system is three times as likely to have exactly one request as to be empty. This is a *ratio of probabilities*, not a claim that state 1 occurs 300% of the time.

For example, if $P_0$ happened to equal 0.05, then $P_1$ would be 0.15. We have **not** yet determined the true $P_0$.

---

## 6. Deriving $P_2$ Relative to $P_0$

Consider states 1 and 2:

```text
                  arrival: λ = 30/sec
        State 1  -------------------->  State 2
                 <--------------------
                  completion: 2μ = 20/sec
```

In state 2, **two connections are busy**, so the aggregate completion rate is $2\mu$.

Local balance gives:

$$
P_1\lambda=P_2(2\mu)
$$

Solve for $P_2$:

$$
P_2=\frac{\lambda}{2\mu}P_1
$$

Since $a=\lambda/\mu$:

$$
P_2=\frac{a}{2}P_1
$$

Substitute $P_1=aP_0$:

$$
P_2=\frac{a}{2}(aP_0)
$$

Therefore:

$$
\boxed{P_2=\frac{a^2}{2!}P_0}
$$

Numerically, with $a=3$:

$$
P_2=\frac{3^2}{2}P_0=\boxed{4.5P_0}
$$

**Interpretation:** State 2 is 4.5 times as likely as state 0. The ratio between *adjacent* states, $P_2/P_1=3/2$, is smaller than $P_1/P_0=3$ because the departure rate is higher when two connections are busy.

---

## 7. Deriving $P_3$ and Seeing the Factorial

When the system is in state 3, three connections are busy. Local balance between states 2 and 3 gives:

$$
P_2\lambda=P_3(3\mu)
$$

Therefore:

$$
P_3=\frac{a}{3}P_2
$$

Substitute the previous result:

$$
P_3=\frac{a}{3}\left(\frac{a^2}{2!}P_0\right)
$$

Then:

$$
P_3=\frac{a^3}{3\times2\times1}P_0
$$

So:

$$
\boxed{P_3=\frac{a^3}{3!}P_0}
$$

For $a=3$:

$$
P_3=\frac{27}{6}P_0=\boxed{4.5P_0}
$$

Notice that **$P_2=P_3$** in this example. From state 2 to state 3, the arrival rate $\lambda=30$/sec matches the completion rate in state 3, $3\mu=30$/sec. The probability ratio is therefore $P_3/P_2=1$.

### Why does the factorial appear?

Every move to a higher state, while idle connections remain, adds another busy connection. Therefore the aggregate completion rate grows from $\mu$ to $2\mu$ to $3\mu$, and so on.

Successive probability ratios contribute the denominators $1$, $2$, $3$, ...; their product is the factorial:

$$
1\times2\times3=3!
$$

The factorial is not an arbitrary algebraic device. **It reflects the growing number of connections that can independently complete service.**

---

## 8. Deriving $P_4$ and the General Relationship

At state 4, all four connections are busy. Balance between states 3 and 4 gives:

$$
P_3\lambda=P_4(4\mu)
$$

Therefore:

$$
P_4=\frac{a}{4}P_3
$$

Substitute $P_3=(a^3/3!)P_0$:

$$
P_4=\frac{a}{4}\frac{a^3}{3!}P_0
$$

Thus:

$$
\boxed{P_4=\frac{a^4}{4!}P_0}
$$

For $a=3$:

$$
P_4=\frac{81}{24}P_0=\boxed{3.375P_0}
$$

### Consolidating the results

| State | General expression | With $a=3$ |
|---|---|---|
| 0 | $P_0$ | $P_0$ |
| 1 | $aP_0$ | $3P_0$ |
| 2 | $\dfrac{a^2}{2!}P_0$ | $4.5P_0$ |
| 3 | $\dfrac{a^3}{3!}P_0$ | $4.5P_0$ |
| 4 | $\dfrac{a^4}{4!}P_0$ | $3.375P_0$ |

We have derived the general relationship for states up through the available connection count:

$$
\boxed{P_n=\frac{a^n}{n!}P_0,\qquad 0\le n\le c}
$$

**Scope matters:** This expression holds only while the aggregate completion rate in state $n$ equals $n\mu$—that is, through $n=c$.

---

## 9. The General Flow-Balance Recurrence

For any pair of neighboring states $n-1$ and $n$ in the standard M/M/c birth-death model:

$$
P_{n-1}\lambda=P_n\mu_n
$$

Therefore:

$$
\boxed{P_n=\frac{\lambda}{\mu_n}P_{n-1}}
$$

Before saturation, $\mu_n=n\mu$:

$$
P_n=\frac{a}{n}P_{n-1}
$$

Repeated substitution yields the factorial relationship.

After saturation, $\mu_n=c\mu$ is constant. That means the ratio between successive probabilities becomes constant too. We will derive that **geometric tail** in Stage 4.

---

## 10. What Have We Actually Calculated?

So far, we have **relative state probabilities**, not normalized numerical probabilities.

For example:

$$
P_2=4.5P_0
$$

means:

$$
\frac{P_2}{P_0}=4.5
$$

It does **not** mean $P_2=4.5$ or $P_2=450\%$. Actual probabilities must lie between 0 and 1.

To obtain the numerical values of $P_0,P_1,P_2,\ldots$, we must account for **every possible state**, including states beyond 4, and enforce:

$$
P_0+P_1+P_2+\cdots=1
$$

That step—**probability normalization**—comes after we derive the geometric tail.

---

## 11. Worked Review and Conceptual Checks

**Check A — Relative probability.** If $\lambda=20$/sec and $\mu=10$/sec, what is $P_1$ in terms of $P_0$?

$$
P_1=\frac{20}{10}P_0=2P_0
$$

The system is twice as likely to have exactly one request as to be empty.

**Check B — Why does $P_2$ differ from $P_1$?** In our running example, $P_1=3P_0$ but $P_2=4.5P_0$, not $9P_0$.

That is because the state-2 aggregate completion rate is $2\mu$, not $\mu$.

**Check C — Why are $P_2$ and $P_3$ equal here?** Their adjacent-state ratio is:

$$
\frac{P_3}{P_2}=\frac{\lambda}{3\mu}=\frac{30}{30}=1
$$

**Check D — Why does the factorial stop growing beyond state 4?** Because a four-connection pool never has more than four busy connections. At $n>4$, the departure rate stays at $4\mu$, even as the waiting queue grows.

---

## 12. Key Takeaways

1. **State probability $P_n$** is the probability of observing exactly $n$ requests in the system in steady state; it can also be interpreted as a long-run time fraction.
2. A constant *average* arrival rate does not imply uniformly spaced requests. Random arrival gaps can allow the system to become empty.
3. A **probability-weighted transition rate**, such as $P_0\lambda$, gives the long-run frequency of a transition averaged over all elapsed time.
4. **Local flow balance** equates these long-run frequencies across neighboring state boundaries.
5. The resulting recurrence is $P_n=(\lambda/\mu_n)P_{n-1}$.
6. Before all connections are busy, each additional connection increases the aggregate completion rate. This generates the **factorial** in $P_n=(a^n/n!)P_0$.
7. These expressions give **relative probabilities**. Actual numerical probabilities require normalization over every possible state.
8. Once all connections are busy, the aggregate completion rate stops increasing, setting up a different probability pattern beyond $n=c$.

### Central result of Stage 3

$$
\boxed{P_n=\frac{a^n}{n!}P_0,\qquad 0\le n\le c}
$$

---

## 13. Next Stage

**[4 — Geometric Tail](4-geometric-tail.md)**

We'll examine states $n>c$ where all servers are already busy. Since the aggregate completion rate is now constant, we'll show how the state-probability ratios become constant:

$$
\frac{P_{n+1}}{P_n}=\frac{a}{c}=\rho,\qquad n\ge c
$$

This creates the geometric tail of the M/M/c state distribution and prepares us to calculate $P_0$ through probability normalization.