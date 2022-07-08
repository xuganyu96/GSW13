# GSW13

## Setup
The setup function takes a security parameter $\lambda$ and returns the parameters for the encryption scheme, which includes:
- $n \in \mathbb{Z}$ is an integer lattice dimension
- $q \in \mathbb{Z}$ is an odd prime
- $\chi \subset \mathbb{Z}$ is some random distribution
- $m \in \mathbb{Z}$ is the sample size for the public key

Furthermore, we denote $l = \lceil \log_2{q} \rceil = \lfloor \log_2{q}\rfloor + 1$ and $N = (n+1) \cdot l$

## Secret Key
To generate the secret key, we first generate an LWE secret $\vec{s} = (1, -\vec{t}) \in \mathbb{Z}_q^{(n+1)}$ where $\vec{t} \leftarrow \mathbb{Z}_q^n$ is uniformly sampled, then output:

$$
\text{sk} = \vec{v} = \text{PowersOf2}(\vec{s}) \in \mathbb{Z}_q^N
$$

## Public key
The public key is the LWE problem, which is constructed by adding noise to the output of a series of randomly generated linear equation. More specifically:

1. Uniformly sample $B \leftarrow \mathbb{Z}_q^{m \times n}$
2. Sample the noise $\vec{e} \leftarrow \chi^m$
3. Compute the noisy result $\vec{b} = B\vec{t} + \vec{e}$, where $\vec{t}$ is the LWE secret in the secret key

Finally, output the public key:

$$
\text{pk} = A = \lbrack \vec{b} \hspace{3pt} \hspace{3pt} B\rbrack \in \mathbb{Z}_q^{m \times (n+1)}
$$

## Encryption
To encrypt the plaintext $\mu \in \mathbb{Z}_q$ using the public key $A \in \mathbb{Z}_q^{m \times (n+1)}$:

$$
C = \text{Flatten}(\mu I_N + \text{BitDecomp}(RA)) \in \mathbb{Z}_2^{N \times N}
$$

where $R \in \mathbb{Z}_2^{N \times m}$ is uniformly sampled.

## Decryption
Given ciphertext $C \in \mathbb{Z}_2^{N \times N}$ (we can safely assume that all homomorphic evaluation will include `Flatten`, so the ciphertext will always be small in magnitude) and the secret key $\vec{v} \in \mathbb{Z}_q^N$, the plaintext $\mu \in \mathbb{Z}_q$ can be recovered using the following relationship

$$
C \cdot \vec{v} = \mu \cdot \vec{v} + R \cdot \vec{e}
$$

where because $R \in \mathbb{Z}_2^{N \times N}$, the magnitude of coefficients of $R \cdot \vec{e}$ is the same as the magnitude of the original noise.

## Helper functions
### Bit decomposition
The point of bit composition is to break a vector in $\mathbb{Z}_q^n$ into a vector with small coefficients $\mathbb{Z}_2^{n \cdot \lceil \log{q} \rceil}$

First, observe that for each integer in the ring $a \in \mathbb{Z}_q$ there is a unique binary representation:

$$
a = \sum_{j=0}^{\lfloor \log{q} \rfloor} 2^j \cdot a_j, \hspace{5pt} a_j \in \mathbb{Z}_2
$$

Thus, for $\vec{a} = (a_1, a_2, \ldots, a_n) \in \mathbb{Z}_q^n$, we can define bit decomposition:

$$
\text{BitDecomp}(\vec{a}) = (a_{1, 0}, a_{1, 1}, \ldots, a_{1, \lfloor \log{q} \rfloor}, \ldots, a_{n, 0}, a_{n, 1}, \ldots, a_{n, \lfloor \log{q} \rfloor}) \in \mathbb{Z}_2^{n \cdot \lceil \log{q} \rceil}
$$

Where for each $i$,  $a_i = \sum_{j=0}^{\lfloor \log{q} \rfloor} 2^j \cdot a_{i, j}$

### Inverse of bit decomposition
The inverse of bit decomposition takes the binary representation and constructs the original vector in $\mathbb{Z}_q^n$. Let $\vec{a}^\prime = (a_{1, 0}, a_{1, 1}, \ldots, a_{1, \lfloor \log{q} \rfloor}, \ldots, a_{n, 0}, a_{n, 1}, \ldots, a_{n, \lfloor \log{q} \rfloor}) \in \mathbb{Z}_2^{n \times \lceil \log{q} \rceil}$:

$$
\text{BitDecomp}^{-1}(\vec{a}^\prime) = (\sum_{j=0}^{\lfloor \log{q} \rfloor}2^j \cdot a_{1, j}, \ldots, \sum_{j=0}^{\lfloor \log{q} \rfloor}2^j \cdot a_{n, j}) \in \mathbb{Z}_q^n
$$

It is worth pointing out that even if the input is not strictly a binary vector $\vec{a}^\prime \in \mathbb{Z}_q^{n \cdot \lceil \log{q} \rceil}$, the function is still well defined

### Powers of 2
Given vector $\vec{b} = (b_1, b_2, \ldots, b_n) \in \mathbb{Z}_q^n$:

$$
\text{PowersOf2}(\vec{b}) = (b_1, 2b_1, \ldots, 2^{\lfloor \log{q}\rfloor}b_1, \ldots, b_n, 2b_n, \ldots, 2^{\lfloor \log{q}\rfloor}b_n) \in \mathbb{Z}_q^{n \cdot \lceil \log{q} \rceil}
$$

### Flatten
The function flatten is constructed by composing bit decomposition and its inverse

$$
\text{Flatten}(\vec{a}^\prime) = \text{BitDecomp}(\text{BitDecomp}^{-1}(\vec{a}^\prime))
$$

### Important identity
These four functions are very useful because they allow vectors to be transformed with small coefficients while preserving dot products, which are used to decrypt:

1. "bit decomposition" and "powers of 2" cancel each other out in inner product:

$$
\langle \text{BitDecomp}(\vec{a}), \text{PowersOf2}(\vec{b}) \rangle = \langle \vec{a}, \vec{b}\rangle
$$

2. Applying "powers of 2" to one side is equivalent to applying "inverse bit decomposition" on the other side:

$$
\langle \vec{a}^\prime, \text{PowersOf2}(\vec{b}) \rangle = \langle \text{BitDecomp}^{-1}(\vec{a}^\prime), \vec{b}\rangle
$$

3. "Flatten" does not affect the inner product:

$$
\langle \vec{a}^\prime, \text{PowersOf2}(\vec{b}) \rangle = \langle \text{Flatten}(\vec{a}^\prime), \text{PowersOf2}(\vec{b})\rangle
$$

Each of the functions can be applied to matrices $\mathbb{Z}_q^{m \times n}$ by applying to each row vector separately, as well. Correspondingly, the indentites are written with matrix multiplication:

$$
\text{BitDecomp}(A) \cdot \text{PowersOf2}(\vec{b}) = A \cdot \vec{b}
$$

$$
A^\prime \cdot \text{PowersOf2}(\vec{b}) = \text{BitDecomp}^{-1}(A^\prime) \cdot \vec{b} = \text{Flatten}(A^\prime) \cdot \text{PowersOf2}(\vec{b})
$$
