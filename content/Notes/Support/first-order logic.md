[Introduction to first order logic](https://youtu.be/ARywou8HLQk?si=RrHT_dfxN4xvu7yU)

First-Order Logic (also known as Predicate Logic) extends [[first-order logic#^aeed27|Propositional Logic]] to express statements involving variables and properties.
## What is a Predicate?

A predicate is a statement involving **variables** that is neither true nor false until specific values are assigned to those variables.

Examples:

• x > 3
• x + y = z
## Structure of Predicate Statements

A statement in predicate logic consists of two parts:
1. Subject: The variable or object being discussed (e.g., x).
2. Predicate: The property or condition assigned to the subject (e.g., is greater than 3).
## Notation and Propositional Functions

Predicates are denoted using propositional functions like P(x):
• P represents the predicate condition ("is greater than 3").
• x represents the subject or variable.
## Evaluating Truth Values

Substituting specific values for variables converts the predicate into a proposition with a distinct truth value:

• P(4): "4 > 3" evaluates to True.
• P(2): "2 > 3" evaluates to False.
## Quantifiers

First-Order Logic uses quantifiers (such as universal and existential quantifiers) to express statements about groups of objects.


------------------------------------------------------------
#### Propositional Logic 
^aeed27

Propositional logic evaluates entire statements as simple, indivisible units (atoms) that are either true or false.

| Feature                  | Propositional Logic                                               | Predicate Logic                                                                       |
| ------------------------ | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Basic Unit               | Atomic propositions ($P, Q, R$)                                   | Predicates $P(x)$, Terms/Variables ($x, y$), Constants ($a, b$)                       |
| Expressive Power         | Low (treats statements as "black boxes")                          | High (can model properties, relations, and sets)                                      |
| Quantification           | None                                                              | Supported ($\forall$ "for all", $\exists$ "there exists")                             |
| Decidability             | **Decidable** (truth tables can check any formula in finite time) | **Undecidable** (no general algorithm can determine if an arbitrary formula is valid) |
| Computational Complexity | NP Complete(SAT Problem)                                          | Semi-decidable / Higher complexity                                                    |
**Propositional logic is a proper subset of predicate logic**.



