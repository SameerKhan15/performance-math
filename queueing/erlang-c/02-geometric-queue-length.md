# Erlang-C From First Principles — Foundations

## 1. Why We Are Studying This

The goal is to understand connection-pool checkout failures from first principles rather than treating Erlang-C as a black-box formula.

The eventual result we want to understand is:
$$
P(\text{fail})
=
C(c,a)\,
e^{-(c-a)T/S}
$$
where:

- \(c\) = connection-pool size
- \(a\) = offered load
- \(T\) = checkout timeout
- \(S\) = model time-scale parameter
- \(C(c,a)\) = Erlang-C probability that an arriving request must wait

We have **not derived this complete equation yet**.

The purpose of these notes is to capture the foundations established so far.

---

# 2. Basic Model

Consider a connection pool with:

\[
c=3
\]

connections.

We use:
$$
\mu=\frac{1}{S}
$$
as the rate associated with one busy connection.

We deliberately keep \(S\) abstract for now rather than attaching an operational interpretation to it.

If all three connections are busy, their aggregate completion rate is:
$$
3\mu
$$
More generally, with \(c\) busy connections:
$$
\boxed{\text{aggregate completion rate}=c\mu}
$$
---

# 3. Parallelism Matters

Suppose all three connections are busy:

```text
[A] [B] [C]
```

Each connection independently progresses toward its next completion.

Because three connections operate in parallel, the expected interval until the **next completion from any connection** is:
$$
\boxed{\frac{1}{3\mu}}
$$
More generally:
$$
\boxed{\text{mean interval between completions while all }c\text{ are busy}
=\frac{1}{c\mu}}
$$
This is different from having one connection perform three operations sequentially.

---

# 4. Waiting Behind Other Requests

Suppose the pool is full and the queue looks like:

```text
[A] [B] [C]

D -> E -> YOU
```

There are two requests ahead of YOU.

YOU therefore need three completion events before obtaining a connection:

1. first completion admits D
2. second completion admits E
3. third completion admits YOU

While the queue remains nonempty, all three connections stay busy.

Each completion interval therefore has expected duration:
$$
\frac{1}{3\mu}
$$
Hence:
$$
E[W\mid J=2]
=
3\left(\frac{1}{3\mu}\right)
=
\frac{1}{\mu}
$$
where:  
`J = number of waiting requests already ahead of the arriving request`

The general pattern is:
$$
\boxed{
E[W\mid J=j]
=
\frac{j+1}{3\mu}
}
$$
for a three-connection pool.

For a general pool size \(c\):
$$
\boxed{
E[W\mid J=j]
=
\frac{j+1}{c\mu}
}
$$

Examples for \(c=3\):
$$
J=0
\quad\Rightarrow\quad
E[W]=\frac{1}{3\mu}
$$
$$
J=1
\quad\Rightarrow\quad
E[W]=\frac{2}{3\mu}
$$
$$
J=2
\quad\Rightarrow\quad
E[W]=\frac{3}{3\mu}
=\frac{1}{\mu}
$$
---

# 5. Queue States

Once all three connections are busy, define queue states according to the number of requests waiting.

For clarity, let:
$$
Q_j
$$
denote the long-run probability of observing the system with all three connections busy and exactly \(j\) requests waiting.

Therefore:
$$
Q_0
=
P(\text{3 busy connections, 0 waiting})
$$
$$
Q_1
=
P(\text{3 busy connections, 1 waiting})
$$
$$
Q_2
=
P(\text{3 busy connections, 2 waiting})
$$
and so on.

For example:

```text
Queue state 0

[A] [B] [C]

queue: empty
```

versus:

```text
Queue state 1

[A] [B] [C]

queue: D
```

---

# 6. What Moves the Queue Up?

Suppose the system is currently in queue state 0:

```text
[A] [B] [C]

queue: empty
```

A new request arrives.

Because all three connections are occupied, the request cannot immediately acquire one:

```text
[A] [B] [C]

queue: D
```

The system has moved:
$$
0\rightarrow1
$$
If requests arrive at rate:
$$
\lambda
$$
then arrivals push the queue upward at rate:
$$
\boxed{\lambda}
$$
---

# 7. What Moves the Queue Down?

Now suppose the system is in queue state 1:

```text
[A] [B] [C]

queue: D
```

One of the three busy connections completes.

D immediately acquires the available connection:

```text
[A] [D] [C]

queue: empty
```

The system has moved:
$$
1\rightarrow0
$$
Because all three connections were busy, completions occur at aggregate rate:
$$
\boxed{3\mu}
$$
Thus:
$$
\boxed{\lambda\text{ pushes queue length upward}}
$$
while:
$$
\boxed{3\mu\text{ pushes queue length downward}}
$$
For a general pool of size \(c\), the downward rate while the pool remains saturated is:
$$
c\mu
$$
---

# 8. Steady-State Flow Balance

Suppose the system has been operating long enough to reach statistical equilibrium.

Consider the boundary between queue states 0 and 1.

If the system occupies queue state 0 for fraction \(Q_0\) of the time and arrivals occur at rate \(\lambda\), then the long-run flow from state 0 into state 1 is:
$$
Q_0\lambda
$$
Similarly, if the system occupies queue state 1 for fraction \(Q_1\) of the time and completions occur at rate \(3\mu\), then the long-run flow from state 1 back into state 0 is:
$$
Q_1(3\mu)
$$
At steady state, these flows must balance:
$$
\boxed{
Q_0\lambda
=
Q_1(3\mu)
}
$$
This is the key **steady-state flow-balance equation**.  
**Steady-state flow balance** is a foundational principle in Queueing Theory, stating that once a queueing system reaches long-term mathematical equilibrium,  
the rate at which entities enter a specific state must equal the rate at which they leave it. This concept allows engineers to solve for long term probabilities of a system's behavior,  
such as predicting wait times or queue lengths.  
`Total Rate of Flow INTO State n = Total Rate of Flow OUT of State n`  
---

# 9. Concrete Flow-Balance Example

Suppose:
$$
Q_0=0.20
$$
$$
\lambda=24/\text{s}
$$
and:
$$
3\mu=30/\text{s}
$$
Then the long-run transition rate from state 0 to state 1 is:
$$
0.20\times24
=
4.8/\text{s}
$$
For equilibrium, the reverse transition rate must also equal:
$$
4.8/\text{s}
$$
Therefore:
$$
Q_1\times30=4.8
$$
which gives:
$$
Q_1=0.16
$$
So:
$$
\boxed{
0.20(24)
=
0.16(30)
=
4.8/\text{s}
}
$$
This means:

- `0 -> 1` transitions occur at 4.8/sec on average
- `1 -> 0` transitions occur at 4.8/sec on average

There are therefore 9.6 total boundary crossings/sec if both directions are counted, but **steady-state balance concerns equality of the two directional flows**.

An important intuition follows:

> A state with a faster exit rate needs to be occupied for less time to generate the same long-run flow.

---

# 10. Utilization Appears Naturally

Starting with:
$$
Q_0\lambda=Q_1(3\mu)
$$
rearrange:
$$
\frac{Q_1}{Q_0}
=
\frac{\lambda}{3\mu}
$$
Define utilization:
$$
\boxed{
\rho=\frac{\lambda}{3\mu}
}
$$
Therefore:
$$
\boxed{
Q_1=\rho Q_0
}
$$
For example, if:
$$
\rho=0.8
$$
then:
$$
Q_1=0.8Q_0
$$
---

# 11. Apply the Same Balance to Every Queue Level

Consider states 1 and 2.

The same mechanisms operate:
$$
1\xrightarrow{\lambda}2
$$
and:
$$
2\xrightarrow{3\mu}1
$$
Therefore steady-state balance gives:
$$
Q_1\lambda=Q_2(3\mu)
$$
so:
$$
Q_2=\rho Q_1
$$
Since:
$$
Q_1=\rho Q_0
$$
we obtain:
$$
Q_2=\rho^2Q_0
$$
Similarly:
$$
Q_3=\rho Q_2=\rho^3Q_0
$$
and in general:
$$
\boxed{
Q_j=Q_0\rho^j
}
$$
---

# 12. Why Queue Length Has a Geometric Shape

The sequence:
$$
Q_0,\;
Q_0\rho,\;
Q_0\rho^2,\;
Q_0\rho^3,\ldots
$$
has a geometric shape because every additional queue position multiplies the previous state's probability by the **same constant ratio**:
$$
\boxed{
\frac{Q_{j+1}}{Q_j}=\rho
}
$$
For example, with:
$$
\rho=0.8
$$
the relative multipliers are:
$$
1,\;0.8,\;0.64,\;0.512,\ldots
$$
These are **not probabilities by themselves**.

They indicate each state's probability relative to \(Q_0\).

If:
$$
Q_0=0.20
$$
then:
$$
Q_1=0.20(0.8)=0.16
$$
$$
Q_2=0.20(0.8^2)=0.128
$$
$$
Q_3=0.20(0.8^3)=0.1024
$$
The reason for the geometric structure is physical, not merely algebraic:

> Once the pool is saturated, every additional queue level has the same upward transition rate \(\lambda\) and the same downward transition rate \(3\mu\).

Therefore the ratio between neighboring state probabilities remains constant.

---

# 13. What We Have Established So Far

We now understand the following chain from first principles:
$$
\boxed{
\text{arrival rate }\lambda
}
$$
pushes queue length upward, while:
$$
\boxed{
\text{aggregate completion rate }c\mu
}
$$
pushes it downward.

Steady-state balance between neighboring saturated queue states gives:
$$
Q_j\lambda
=
Q_{j+1}c\mu
$$
therefore:
$$
\frac{Q_{j+1}}{Q_j}
=
\frac{\lambda}{c\mu}
=
\rho
$$
and hence:
$$
\boxed{
Q_j=Q_0\rho^j
}
$$
So the saturated queue-length states have a geometric shape.

Separately, if an arriving request has \(j\) requests ahead of it, then it requires \(j+1\) completions before obtaining a connection, giving expected waiting time:
$$
\boxed{
E[W\mid J=j]
=
\frac{j+1}{c\mu}
}
$$
These two ideas are the foundation required to derive the Erlang-C waiting-time result.

---

# 14. What We Have NOT Derived Yet

We should not jump ahead and treat the following results as proven.

The next stages are:

### A. Normalize the geometric queue states

We need to derive the exact conditional probability:
$$
P(J=j\mid W>0)
$$
which will lead to:
$$
P(J=j\mid W>0)
=
(1-\rho)\rho^j
$$
### B. Move from expected wait to a waiting-time distribution

So far we understand:
$$
E[W\mid J=j]
$$
but Erlang-C timeout analysis needs:
$$
P(W>T\mid J=j)
$$
### C. Average over all possible queue lengths

We will combine:
$$
P(J=j\mid W>0)
$$
with:
$$
P(W>T\mid J=j)
$$
using the law of total probability.

### D. Derive the conditional waiting-time tail

The target is:
$$
\boxed{
P(W>T\mid W>0)
=
e^{-(c\mu-\lambda)T}
}
$$
### E. Rewrite using \(a\) and \(S\)

Eventually:
$$
c\mu-\lambda
=
\frac{c-a}{S}
$$
giving:
$$
\boxed{
P(W>T\mid W>0)
=
e^{-(c-a)T/S}
}
$$
### F. Derive Erlang-C itself

We still need to derive:
$$
C(c,a)=P(W>0)
$$
which answers:

> What is the probability that an arriving request encounters all \(c\) connections busy and therefore has to wait?

Finally:
$$
P(\text{fail})
=
P(W>0)
P(W>T\mid W>0)
$$
becomes:
$$
\boxed{
P(\text{fail})
=
C(c,a)e^{-(c-a)T/S}
}
$$
That will complete the original connection-pool timeout argument.