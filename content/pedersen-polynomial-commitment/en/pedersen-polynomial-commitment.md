# Polynomial Commitments Via Pedersen Commitments

A polynomial commitment is a mechanism by which a prover can convince a verifier a polynomial $p$ has an evaluation $y = p(x)$ at point $x$ without revealing anything about $p$. The sequence is as follows:

1. The prover sends to the verifier a *commitment* $C$ to the polynomial, "locking in" their polynomial.
2. The verifier responds with a value $u$ they want the polynomial evaluated at.
3. The prover responds with $y$ and $\pi$, where $y$ is the evaluation of $p(u)$ and $\pi$ is proof that the evaluation was correct.
4. The verifier checks $C$, $u$, $y$, $\pi$ and accepts or rejects that the evaluation of the polynomial was valid.

This commitment scheme does not require a trusted setup. However, the communication overhead is $O(n)$ as the prover must send a commitment for each coefficient in their polynomial.

## The Steps to Committing to the Polynomial
### Prover commits each polynomial coefficient
The prover can commit to the polynomial by creating a [Pedersen Commitment](https://www.rareskills.io/post/pedersen-commitment) of each coefficient. For a Pedersen Committment, the prover and verifier need to agree on two elliptic curve points with unknown discrete logs. We will use $G$ and $B$.

For example, if we have polynomial

$$p = c_0+c_1x+c_2x^2$$

We can create a Pedersen commitment for each coefficient. We will need three blinding terms $\gamma_0$, $\gamma_1$, $\gamma_2$. For convenience, any scalar used for blinding will use a lower-case Greek letter. We always use the elliptic curve point $B$ for the blinding term. Our commitments are produced as follows:

$$
\begin{align*}
C_0=c_0G+\gamma_0B \\
C_1=c_1G+\gamma_1B \\
C_2=c_2G+\gamma_2B \\
\end{align*}
$$

The prover sends the tuple $(C_0, C_1, C_2)$ to the verifier.

### Verifier chooses $u$
The verifier chooses their value for $u$ and sends that to the prover.

### Prover evaluates the polynomial and computes the proof
The prover compute the original polynomial as:

$$
y = c_0 + c_1u + c_2u^2
$$

### Prover evaluates the blinding terms
The proof that the evaluation was done correctly is given by the following polynomial, which uses the blinding terms multiplied by the associated power of $u$. The reason for this will be explained later.

$$
\pi = \gamma_0 + \gamma_1u+\gamma_2u^2
$$

The prover sends $(y, \pi)$ to the verifier. Note that the prover is only sending field elements (scalars) not elliptic curve points.

**The proof is simply the sum of the blinding terms for each coefficient multiplied by the powers of $u$ for that coefficient.**

## Verification step
The verifier runs the following check:

$$
C_0+C_1u+C_2u^2\stackrel{?}{=}yG+\pi B
$$

If the prover was honest, the $y$ will be the sum of the polynomial coefficients multiplied by successive powers of $u$, and $\pi$ will be the sum of the blinding terms multiplied by the powers of $u$.

## Why the verification step works
If we expand the elliptic curve points to their underlying values, we see the equation is balanced:

$$
\begin{align*}
C_0 + C_1u + C_2u^2 &= yG + \pi B \\
(c_0G + \gamma_0B) + (c_1G + \gamma_1B)u + (c_2G + \gamma_2B)u^2 &= (c_0 + c_1u + c_2u^2)G + (\gamma_0 + \gamma_1u + \gamma_2u^2)B \\
c_0G + \gamma_0B + c_1Gu + \gamma_1Bu + c_2Gu^2 + \gamma_2Bu^2 &= (c_0 + c_1u + c_2u^2)G + (\gamma_0 + \gamma_1u + \gamma_2u^2)B \\
c_0G + c_1Gu + c_2Gu^2 + \gamma_0B + \gamma_1Bu + \gamma_2Bu^2 &= (c_0 + c_1u + c_2u^2)G + (\gamma_0 + \gamma_1u + \gamma_2u^2)B \\
(c_0 + c_1u + c_2u^2)G + (\gamma_0 + \gamma_1u + \gamma_2u^2)B &= (c_0 + c_1u + c_2u^2)G + (\gamma_0 + \gamma_1u + \gamma_2u^2)B \\
\end{align*}
$$

In a sense, the prover is evaluating the polynomial using the coefficients to the polynomial and their choice of $u$. This will produce the evaluation of the original polynomial plus the blinding terms of the polynomial.

The proof of correct evaluation is that the prover is able to separate the blinding terms from evaluation of the polynomial -- even though the prover does not know the discrete logs of $yG$ and $\pi B$.

## Why the prover cannot cheat
Cheating on the prover's part means they don't honestly evaluate $y = p(u)$ but still tries to pass the final evaluation step.

Without loss of generality, let's say the prover sends the correct commitments for the coefficients $C_0, C_1, C_2$.

We say without loss of generality because there is a mismatch between the coefficients sent in the commitments and the coefficients used to evaluate the polynomial.

To do so, the prover sends $y'$ where $y' \neq c_0 + c_1u + c_2u^2$.

Using the final equation from the previous section, we see that the prover must satisfy:

$$
(c_0 + c_1u + c_2u^2)G+(\gamma_0 + \gamma_1u+\gamma_2u^2)B=y'G+(\gamma_0 + \gamma_1u+\gamma_2u^2)B
$$

The $G$ terms of the equation are clearly unbalanced. The other "lever" the prover can pull is adjusting the $\pi$ that they send.

$$
(c_0 + c_1u + c_2u^2)G+(\gamma_0 + \gamma_1u+\gamma_2u^2)B=y'G + \boxed{\pi'}B
$$

Since $y' \neq c_0 + c_1u + c_2u^2$ the malicious prover must rebalance the equation by picking a term $\pi'$ that accounts for the mismatch in the $G$ terms. The prover can try to solve for $\pi'$ with 

$$
\pi'B = (c_0 + c_1u + c_2u^2)G+(\gamma_0 + \gamma_1u+\gamma_2u^2)B - y'G
$$

But solving this equation requires the malicious prover to know the discrete logs of $G$ and $B$.

For example, if we know that the discrete log of $B$ is $b$ and the discrete log of $G$ is $g$, then we can compute $\pi'$ as

$$
\begin{align*}
\pi'b = (c_0 + c_1u + c_2u^2)g+(\gamma_0 + \gamma_1u+\gamma_2u^2)b - y'g \\
\\
\pi' = \frac{(c_0 + c_1u + c_2u^2)g+(\gamma_0 + \gamma_1u+\gamma_2u^2)b - y'g}{b}
\end{align*}
$$

But again, this is not possible because computing the discrete log of $B$ and $G$ is infeasible.

## What the verifier learns
The verifier learns that the commitments $C_0, C_1, C_2$ represent valid commitments to a polynomial that is at most degree 2, and that $y$ is the value of the polynomial evaluated at $u$.