[[Handbook of Satisfiability - Second Edition - Armin Biere, Marijn Heule, Hans van Maaren, Toby Walsh - ( WeLib.org )-529-591.pdf]]

## The core definition

> **Definition 13.3.3 (Group Isomorphism).** Let ⟨G, ∗⟩ and ⟨G′, ∗′⟩ be two groups. An isomorphism of G with G′ is a one-to-one function φ mapping G **onto** G′ such that
> 
> φ(x ∗ y) = φ(x) ∗′ φ(y) for all x, y ∈ G

If such a φ exists, G and G′ are **isomorphic groups**, written G ≅ G′.

Two things must hold simultaneously:

- **φ is a bijection** (one-to-one and onto) — a pure relabeling of elements, nothing more, nothing less.
- **φ is structure-preserving** — it doesn't matter whether you combine elements first and then map, or map first and then combine.