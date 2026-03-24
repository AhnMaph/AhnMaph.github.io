---
title: Hastad's Broadcast Attack
date: 2026-03-24 3:49:00 +0700
tags: [crypto, ctf]
---

# Hastad's Broadcast Attack (RSA)

Note: This attack works well with small public exponent

## Scenario
Suppose Alice sends and unpadded messages $M$ to $3$ people $P1, P2 ,P3$ each using a small public key exponent $e=3$ never change and different moduli $N$ for $i_{th}$ individual. The attack states that as soon as $3>=e$ the messages $M$ is exposed. We can recover $M^3 \pmod{N}$ using Chinese Remainder Theorem. Knowing that $gcd(Ni,Nj)=1$ where $i \ne j$ and  $N1.N2.N3 > M^3$

## Chinese Remainder Theorem
Theorem: Let $m_1,..m_n$ be pairwise coprime $gcd(mi,mj)=1$ with $i \ne j$. Then the system of equations.

![image](https://hackmd.io/_uploads/S1kB9okyWl.png)


has unique solution for $x$ modulo $M$ where $M = m_1...m_n$
Define $b_i = M/m_i$ (the product of all moduli except for $m_i$) and $b_i.b_i \equiv 1 \pmod{m_i}$

The $x$ is found by the unique solution: 
$\boxed{ x\equiv \sum_{i=1}^{n} a_i , b_i , b_i' \pmod{M}}$

## Solution

Now we know the ciphertext $C_i ≡ M^3 \pmod{N_i}   (i=1,2,3)$

$M^3 \equiv C_1 \pmod{N_1}$
$M^3 \equiv C_2 \pmod{N_2}$
$M^3 \equiv C_3 \pmod{N_3}$

Let $N = N1.N2.N3$ , $B_i = N/N_i$
and let $B'_i$ be the modular inverse of $B_i$ modulo $N_i$

$M^3 \pmod{N}$ is calculated as follow equation:

$\boxed{M^3 \equiv \sum_{i=1}^3 C_i, B_i, B_i' \pmod{N},}$

And because $M^3$ is smaller than $N$ hence plaintext = $\sqrt[3]{M^3} = M$

$M \equiv \sqrt[3]{M^3} \pmod{N}$



