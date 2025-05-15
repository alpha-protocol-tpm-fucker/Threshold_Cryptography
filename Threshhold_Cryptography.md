Yes — what you’re describing is almost exactly how modern **threshold-cryptographic key‐management** systems are built.

---

### 1 .  Two independent Shamir sharings for one key-pair

Let

```text
sk ∈ ℤq                         # the private key  
pk = g^sk  (mod p)              # the public key in a DLP group (or EC point)  
```

Treat **`sk`** and **`pk`** as two separate secrets and run a Shamir Secret Sharing (SSS) instance for each:

```text
Ring A (private-key holders)        nA players, threshold tA
  – choose random degree-(tA-1) polynomial  fA(0)=sk
  – give share  si = fA(i)  to player i

Ring B (public-key holders)         nB players, threshold tB
  – choose random degree-(tB-1) polynomial  fB(0)=pk
  – give share  pi = fB(i)  to player i
```

Because the two sharings are independent, the two rings can have **different sizes and thresholds**, and can even live on different physical security domains.

---

### 2 .  Linking the two sharings (optional but common)

If you want anybody to verify that Ring B’s secret really is the public key of Ring A’s secret, embed a public commitment:

```text
commit = g^sk = pk
```

Publish `commit`.
Now Ring B can reconstruct `pk` when its threshold is met and prove equality to `commit`; Ring A can reconstruct `sk` (or run a threshold signature) whose verification key is `commit`.

---

### 3 .  Resharing / refreshing without ever rebuilding the secret

Both rings can reshare their own secret to fresh players or change the threshold by running a **verifiable resharing** protocol:

```text
Given: t-out-of-n Shamir shares {s1 … sn}
Goal: produce t′-out-n′ shares {s′1 … s′n′} of the **same** secret

• Each old player acts as a dealer, sharing his share with the new set
  using Feldman or Pedersen VSS.
• New players Lagrange-interpolate the pieces they receive.
• Old shares can be deleted; the underlying secret never appears.
```

The same works for the `pk` ring (though usually you just publish `pk` instead of sharing it again).

---

### 4 .  Using the private-key ring for threshold operations

Once Ring A holds Shamir shares of `sk`, it can run any threshold primitive:

* **Threshold signatures**: FROST, GG-20, or BLS-t variants.
* **Threshold decryption**: ElGamal / ECIES share-combining.
* **Distributed key generation**: if you want to avoid a trusted dealer altogether.

These protocols never reconstruct `sk`; instead they output a signature or plaintext that anyone can check with `pk`.

---

### 5 .  Putting it together

* Generate or import `sk`; compute `pk`.
* Split `sk` across Ring A with threshold `tA`.
* Optionally split `pk` across Ring B with threshold `tB` (useful if `pk` must stay private until some policy is met).
* Publish (or internally store) a commitment tying the two.
* Allow either ring to reshare its secret at any time via VSS without exposing the underlying key material.

This design is standard in **threshold-RSA**, **threshold-ECDSA (FROST)**, and **DKG + threshold-BLS** systems; the additional public-key sharing ring is simply another SSS layer over a non-sensitive value, so it’s perfectly sound cryptography-wise.
