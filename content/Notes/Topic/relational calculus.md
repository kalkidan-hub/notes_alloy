> [!note] Relational Calculus
> Relational calculus is a declarative, non-procedural logic language used to describe what data to retrieve or what conditions must hold, rather than how to compute them.


>While relational algebra uses explicit operations (like join, project, select) to construct results, relational calculus expresses constraints using **predicates** and **quantifiers** ($\exists$, $\forall$).

There are two flavors: 

**Tuple Relational Calculus (TRC):** Variables represent tuples.

_Example:_ $$\{ t \mid t \in \text{User} \land t.\text{age} > 21 \} $$
- **Domain Relational Calculus (DRC):** Variables represent domain elements (attributes).
    
    _Example:_ 
    $$\{ \langle n, a \rangle \mid \exists a ( \text{User}(n, a) \land a > 21 ) \}$$

> Tuple being the row and attribute being the column.



