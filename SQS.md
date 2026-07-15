# 📘 Special Quasirandom Structures (SQS)

> **Software:** ATAT (Alloy Theoretic Automated Toolkit)
>
> **Program:** `mcsqs`
>
> **Reference:** A. Zunger et al., Phys. Rev. Lett. 65, 353 (1990)

---

# Table of Contents

1. Why do we need SQS?
2. The Problem with Random Alloys
3. Infinite Random Alloy vs Finite DFT Supercell
4. What is a Special Quasirandom Structure?
5. What does `mcsqs` actually do?
6. Correlation Functions
7. What is the Monte Carlo algorithm optimizing?
8. Why are clusters important?
9. Workflow of `mcsqs`
10. Input Files
11. Output Files
12. Summary

---

# 1. Why Do We Need SQS?

Most engineering alloys are **random solid solutions**.

Examples include

- Ni-Co
- Ni-Fe
- Co-Cr
- High Entropy Alloys (HEAs)

Consider a binary alloy

```text
Ni50Co50
```

At high temperature,

every lattice site has

```text
50 % probability → Ni

50 % probability → Co
```

Therefore,

there is **no long-range ordering**.

The arrangement of atoms is completely random.

Example

```text
Ni  Co  Ni  Ni

Co  Ni  Co  Co

Ni  Ni  Co  Ni

Co  Co  Ni  Co
```

This is called a

> **Random Solid Solution**

---

# 2. The Problem with DFT

Density Functional Theory (DFT)

cannot simulate

```text
10²³ atoms.
```

Normally,

DFT calculations contain

```text
16 atoms

32 atoms

64 atoms

128 atoms
```

only.

The problem is

> **How can such a small periodic cell represent an infinitely large random alloy?**

This is exactly the motivation behind

> **Special Quasirandom Structures (SQS).**

---

# 3. Infinite Random Alloy vs Finite Supercell

A real alloy

```text
10²³ atoms
```

↓

Random

↓

No repeating pattern

↓

Infinite.

---

A DFT calculation

```text
64 atoms
```

↓

Periodic

↓

Repeats forever

↓

Artificial periodicity.

Therefore,

a simple random placement of atoms inside a supercell

does **not**

accurately represent

an infinite random alloy.

---

# 4. What is a Special Quasirandom Structure (SQS)?

A Special Quasirandom Structure is

> **A small periodic supercell whose local atomic arrangement closely reproduces the statistical properties of an infinitely large random alloy.**

Notice

it is

NOT

a perfectly random structure.

Instead,

it is

a carefully optimized structure

that

**behaves statistically like a random alloy.**

---

# 5. What Does "Random" Actually Mean?

Many people think

```text
Random

=

Random positions
```

This is

NOT correct.

Randomness

is defined

through

**correlation functions.**

Two structures may have

```text
50% Ni

50% Co
```

yet

one is random

and

one is ordered.

Therefore,

composition

alone

cannot describe randomness.

---

# 6. Correlation Functions

Instead of asking

> "Is this structure random?"

ATAT asks

> "Do neighboring atoms have the same statistical relationships as an infinite random alloy?"

This is measured using

**Correlation Functions.**

---

## Pair Correlation

Look at every

nearest-neighbor pair.

Example

```text
Ni —— Co

Ni —— Ni

Co —— Co
```

ATAT counts

all such pairs.

---

Then

Second nearest neighbors

↓

Third nearest neighbors

↓

Fourth nearest neighbors

and so on.

---

## Triplet Correlation

Now

instead of

2 atoms,

consider

3 atoms.

Example

```text
Ni

/   \

Co——Ni
```

This forms

a

3-body cluster.

---

## Quadruplet

Similarly,

```text
Ni-----Co

|       |

Co-----Ni
```

is

a

4-body cluster.

---

# 7. What is a Cluster?

A cluster is simply

a group of lattice sites.

Examples

```text
Pair

↓

2 atoms
```

```text
Triplet

↓

3 atoms
```

```text
Quadruplet

↓

4 atoms
```

Clusters are

the basic quantities

used

to measure randomness.

---

# 8. What is mcsqs Actually Trying to Match?

Suppose

an infinite random alloy

has

Nearest-neighbor pair correlation

```text
0
```

Second-neighbor pair correlation

```text
0
```

Triplet correlation

```text
0
```

Quadruplet correlation

```text
0
```

Then

`mcsqs`

tries

to build

a finite supercell

whose

correlations are

as close as possible

to these

ideal random values.

It

does **NOT**

minimize energy.

It minimizes

the difference

between

```text
Current Correlations

and

Ideal Random Correlations.
```

---

# 9. What is the Monte Carlo Algorithm Doing?

Initially,

a supercell is created.

Example

```text
Ni

Co

Ni

Co

Ni

...
```

Now

ATAT

selects

two atoms

randomly.

Example

```text
Ni

↓

Co

Co

↓

Ni
```

They are exchanged.

---

After swapping,

ATAT calculates

all

pair,

triplet,

and higher-order correlations

again.

If

the new structure

is closer

to

the ideal random alloy,

the swap is accepted.

Otherwise,

it is rejected

(or sometimes accepted according to the Monte Carlo probability).

This procedure

is repeated

thousands

or even millions

of times.

---

# 10. What Does mcsqs Actually Observe?

The algorithm is continuously observing

- Pair correlations
- Triplet correlations
- Quadruplet correlations
- Higher-order cluster correlations

It is **NOT** observing

- Energy
- Forces
- Stress
- Magnetization
- Electronic structure

These quantities are calculated later

using DFT.

---

# 11. Objective Function

The objective function can be simplified as

```text
Error

=

Σ |Current Correlation

−

Target Correlation|
```

where

Current Correlation

↓

Computed from the trial structure.

Target Correlation

↓

Correlation of an infinite random alloy.

The goal is

```text
Error → Minimum
```

A perfect SQS would have

```text
Error = 0
```

although

this is usually impossible

for small supercells.

---

# 12. Why Are Short-Range Clusters More Important?

Atoms interact

most strongly

with

their nearest neighbours.

Therefore,

matching

nearest-neighbour

correlations

is much more important

than matching

very distant neighbours.

For this reason,

`mcsqs`

assigns

larger weights

to

short-range clusters.

---

# 13. Workflow of mcsqs

```text
Define random alloy

↓

Generate clusters

↓

Construct supercell

↓

Randomly assign atoms

↓

Calculate correlation functions

↓

Compare with ideal random alloy

↓

Swap atoms

↓

Recalculate correlations

↓

Improve objective function

↓

Repeat until convergence

↓

Output Best SQS
```

---

# 14. Input Files

## rndstr.in

Defines

- lattice
- composition
- partially occupied sites

This file describes

the

**ideal random alloy**

NOT

the final structure.

---

## clusters.out

Contains

all clusters

that must

be matched.

Generated using

```bash
mcsqs -2=6 -3=4
```

or

```bash
corrdump
```

---

# 15. Output Files

## bestsqs.out

Final optimized SQS structure.

This file is used

for DFT calculations.

---

## bestcorr.out

Contains

| Cluster | SQS | Random | Difference |
|----------|----:|-------:|-----------:|

This tells

how well

the SQS

matches

the ideal random alloy.

---

## mcsqs.log

Monte Carlo history.

---

# 16. Key Points

✔ SQS is **not** a random arrangement of atoms.

✔ SQS is **not** generated by minimizing energy.

✔ SQS is generated by matching correlation functions.

✔ Correlation functions describe the statistical arrangement of atoms.

✔ The Monte Carlo algorithm repeatedly swaps atoms to improve the correlation match.

✔ The final structure statistically behaves like an infinite random alloy.

---

# Final Definition

> A **Special Quasirandom Structure (SQS)** is a finite periodic supercell generated by optimizing atomic arrangements so that its pair, triplet, and higher-order correlation functions closely match those of an infinite perfectly random alloy. The `mcsqs` program in ATAT achieves this by using a Monte Carlo atom-swapping algorithm that minimizes the difference between the supercell correlations and the target random-state correlations.
