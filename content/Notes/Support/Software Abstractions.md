[[logic-chapter.pdf]]

## 3 Logic
[[Modeling Language]] has *logic* at its core, and the purpose of these logic is providing fundamental concepts. 
These logic is a [[working logic]] -- it's designed for expressing abstractions -- must be small, simple, expressive and flexible. 

Relational logic -- a logic that combines the quantifiers of [[first-order logic]] with the operators of the [[relational calculus]]. 

## 3.1 Three logics in one

> Example constraint: constraint that an address book, represented by a relation address from names to addresses, maps each name to at most one address might be written 


- Predicate Calculus style [verbose one]
	- There's two kinds of expression: relation names[used as predicates] and tuples[formed from quantified variables]
	- _the example constraint could be expressed in predicate calculus as ..._
	- ```alloy
	  all n: Name, d, d’:
	   Address | n -> d in address and n -> d’ in address implies d = d’
	  ```
	- In here ***address*** is a predicate - a relation name - standing for address(n,d). and a ***tuple*** is depicted as `n->d`, just two variables glued with `->`
 
 - Navigation Expression style [common one]
	 - Expressions denote sets, which are formed by navigating from quantified variables along relations. 
	 - _the example constraint could be expressed in navigation style as ..._
	 - ```alloy
	   all n: Name | lone n.address
	   ```
	  - in here `n.address`, **denotes the set of every address that `n` maps to** 
- Relational Calculus style
	- Expressions denote relations and there are no quantifiers at all.
	- _the example constraint could be expressed in relational calculus as ..._
	- ```alloy
	  no ~address.address - iden
	  ```

## 3.2 Atoms and Relations

### 3.2.1 Atoms

An **[[Atoms in Alloy | atom]]** is the most basic building block of a model. It is:

- **indivisible** — can't be broken into smaller parts
- **immutable** — its properties don't change over time
- **uninterpreted** — no built-in meaning (unlike numbers, which have arithmetic baked in)

Almost nothing in the real world is truly atomic — atoms are a _modeling choice_. If something needs to be divisible/mutable/interpreted, you model that with extra relations, not by giving the atom internal structure.

### 3.2.2 Relations

A **relation** is a set of tuples (rows), where each tuple is a sequence of atoms (columns).

- **size** = number of tuples (rows) — can be 0 or more
- **arity** = number of columns — must be ≥ 1
    - arity 1 = **unary** (a set of atoms)
    - arity 2 = **binary**
    - arity 3 = **ternary**
    - arity ≥ 3 = **multirelation**
- A unary relation with exactly one tuple = a **scalar**.
- A unary relation with **at most one** tuple (empty or singleton) = an **option**.

```
Name = {(N0), (N1), (N2)}                  -- a set
myName = {(N0)}                            -- a scalar
address = {(N0,D0), (N1,D1)}               -- binary relation
addr = {(B0,N0,D0), (B0,N1,D1), (B1,N1,D2)} -- ternary relation
```

**Key uniformity trick:** In Alloy, everything is a relation — scalars are singleton relations, sets are unary relations, tuples are singleton relations too. This means the _same_ operators (like join) work on scalars, sets, and relations without special cases, and there's no such thing as an "undefined value" from partial function application (an unmapped input just gives the empty set).

### 3.2.3 Expressing Structure with Relations

Because atoms are flat and structureless, all structure comes from relations:

- **Composite objects**: give each component its own atom, bind them with a relation (e.g., a key card `C1` linked to keys `K11`, `K12` via relations `fst`/`snd`).
- **Containment**: use a multirelation (e.g., `addr` relating books→names→addresses models each book "containing" a name/address table).
- **Mutation over time**: separate an object's _identity_ from its _value_, and relate identity, value, and time (e.g., `value = {(S0,V0,T0), (S0,V1,T1)}` — stock S0 had value V0 at T0, V1 at T1).
- **Giving atoms properties**: since atoms are uninterpreted, properties come only from relations to other atoms (e.g., an ordering relation `precedes` on sequence numbers).

**Important design constraint — relations are flat (first-order):** a relation can't contain another relation as an entry. You can't directly write a "function from books to (functions from names to sets of addresses)" — you have to flatten it into one ternary relation `addr: Book->Name->Addr`. This loses some expressiveness (e.g., distinguishing "mapped to the empty set" from "not mapped at all") but keeps the logic tractable for automated analysis, and workarounds exist (e.g., introduce extra atoms to represent groupings, as with course prerequisites).

### 3.2.4 Functional and Injective Relations

For a binary relation `r`:

- **functional** (a "function") — maps each atom to **at most one** other atom
- **injective** — maps **at most one** atom to each atom (i.e., no two inputs share an output)

A relation can be neither, either, or both. The empty binary relation is always both functional and injective (vacuously).

### 3.2.5 Domain and Range

- **domain** = set of atoms in the _first_ column
- **range** = set of atoms in the _last_ column
- Defined even for relations of arity > 2 (domain = first column, range = last column, regardless of how many columns are in between).

```
address = {(N0,D0), (N1,D1), (N2,D1)}
domain(address) = {N0,N1,N2}
range(address) = {D0,D1}
```

---

## 3.3 Snapshots

A **snapshot** is the graphical way to depict a particular value of sets/relations: nodes = atoms, labeled arcs = tuples (labeled by which relation they belong to). Sets can be shown either as a label on a node or as a labeled contour around several nodes.

For **multirelations**, you can't draw an arc with 3+ endpoints, so you **project** out a column: move the target column to the front, then split the relation into one sub-relation per atom in that column. E.g., projecting the ternary `addr` (book, name, address) onto `Book` gives you one binary graph per book — `B0.addr` is the name→address graph for book `B0` specifically. This is literally how the Alloy Analyzer displays instances of ternary+ relations.

---

## 3.4 Operators

### 3.4.1 Constants

Three built-in constants:

|Constant|Meaning|
|---|---|
|`none`|the empty set|
|`univ`|the universal set (every atom in the model)|
|`iden`|the identity relation — relates every atom to itself|

These are **not** parameterized by type — `univ` and `iden` always range over _every_ atom in the whole model, which is why you almost always see them restricted, e.g. `s <: iden` for "identity relation limited to set `s`." Forgetting to restrict `iden` is a classic bug source (e.g. `iden in r` accidentally also asserts `r` maps _every_ atom in the universe).

### 3.4.2 Set Operators

Apply to any pair of relations of the **same arity**:

|Op|Meaning|
|---|---|
|`+`|union|
|`&`|intersection|
|`-`|difference|
|`in`|subset (or membership, if the left side is a tuple/scalar)|
|`=`|equality|

```
Alias + Group        -- names that are aliases or groups
Alias & RecentlyUsed  -- aliases that were recently used
Name - RecentlyUsed   -- names not recently used
RecentlyUsed in Alias -- "everything recently used is an alias" (a formula, true/false)
```

Note: `in` is deliberately ambiguous between _membership_ (scalar/tuple ∈ set/relation) and _subset_ (set/relation ⊆ set/relation) — both read naturally as "in."

### 3.4.3 Relational Operators

These are the operators where **tuple structure matters** — the real power of the logic.
 
**Arrow / product `->`**: concatenates every tuple of `p` with every tuple of `q`.

```
n->d = {(N0,D0)}                       -- builds a single tuple
Name->Addr = {all combinations}         -- cross product of two sets
```

**Dot join `.`**: the core operator. To join tuple `s1->...->sm` with `t1->...->tn`: if `sm = t1`, the result is `s1->...->sm-1->t2->...->tn` (the matching atom is dropped); otherwise the join is empty. Applied relation-to-relation, `p.q` unions the join of every tuple-pair.

- Binary·binary = ordinary relational composition.
- If `p`,`q` are functions, `p.q` is a function too (function composition).
- If `s` is a set and `r` a binary relation, `s.r` = the **image** of `s` under `r` ("navigation" — follow `r` forward from every member of `s`).
- `r.s` = the image going **backward**.
- Higher-arity joins are common with multirelations: `b.addr` (scalar `.` ternary) projects out book `b`'s name→address sub-table.
- **Not associative in general** — `(a.b).c` and `a.(b.c)` can differ or one can be ill-formed when arities don't line up.

**Box join `e1[e2]`**: pure syntactic sugar, identical in meaning to `e2.e1`. Exists so field-dereference (dot) and array-style indexing (box) look visually distinct even though they're semantically the same operation. E.g. `address[n]` ≡ `n.address`.

**Transpose `~r`**: reverses each tuple's atom order (binary relations only). `~address` maps addresses back to the names that use them.

- `r` is **symmetric** iff `~r in r`.
- Symmetric closure of `r` = `r + ~r`.
- Useful facts: `s.~r` = `r.s`; `r.~r in iden` says `r` is injective; `~r.r in iden` says `r` is functional.

**Transitive closure `^r`**: smallest relation ⊇ `r` that is transitive. Computed as `r + r.r + r.r.r + ...`. Represents **reachability** — `^r` relates `a` to `b` iff there's a path of length ≥1 from `a` to `b` in the graph of `r`. Classic use: `no ^r & iden` says `r` is acyclic.

**Reflexive-transitive closure `*r`**: `*r = ^r + iden` — reachability including zero-length paths (every atom reaches itself). Caveat: because `iden` ranges over the _whole_ universe, `*r` on its own picks up "irrelevant" self-loops for atoms unrelated to `r`'s domain/range — this usually washes out naturally when `*r` appears in a navigation expression (`friends.*(b.addr)`), since irrelevant atoms just don't show up in the join.

**Domain/range restriction `<:` / `:>`**: filter a relation by its first/last element.

- `s <: r` = tuples of `r` **starting** in `s`
- `r :> s` = tuples of `r` **ending** in `s`

```
Group <: address   -- entries in address that start with a group
address :> Addr    -- entries in address that end with an address
```

**Override `++`**: `p ++ q` = union, but any tuple in `p` sharing a starting element with a tuple in `q` is dropped in favor of `q`'s version. Models "insert/update" semantics for maps and variable assignment.

```
homeAddress ++ workAddress   -- prefer work address, fall back to home address
p ++ q  ≡  p - (domain(q) <: p) + q
```

Common use: modeling a state update, e.g. `m' = m ++ k->v` for inserting key `k`↦value `v` into map `m`.

**Precedence**: unary ops (closure, transpose) bind tighter than binary ops; product-like ops (dot, arrow) bind tighter than sum-like ops (`+`, `-`, `&`). All operators are left-associative.

---

## 3.5 Constraints

### 3.5.1 Logical Operators

|Word form|Symbol form|Meaning|
|---|---|---|
|`not`|`!`|negation|
|`and`|`&&`|conjunction|
|`or`|`\|`|disjunction|
|`implies`|`=>`|implication|
|`iff`|`<=>`|bi-implication|

- `a != b` ≡ `not a = b`.
- `F implies G else H` ≡ `(F and G) or (not F and H)`.
- Chained `implies...else...implies...else` reads as a cascading if/elseif/else.
- Shorthand: `{F G H}` ≡ `F and G and H` (juxtaposition = conjunction).
- `implies`/`else` also works between **expressions**, not just formulas: `C implies E1 else E2` picks `E1` or `E2` depending on `C` (a conditional expression, like a ternary operator).

### 3.5.2 Quantification

Form: `Q x: e | F` — `Q` a quantifier, `x` a variable bound over expression `e`, `F` the body.

|Quantifier|Meaning|
|---|---|
|`all x: e \| F`|F holds for every x in e|
|`some x: e \| F`|F holds for at least one x in e|
|`no x: e \| F`|F holds for no x in e|
|`lone x: e \| F`|F holds for at most one x ("less than or equal to one")|
|`one x: e \| F`|F holds for exactly one x|

- Multiple variables: `one x: e, y: e | F` or shared-bound shorthand `one x, y: e | F`.
- `disj` keyword forces distinctness: `all disj x, y: e | F`.
- Quantifiers also apply directly to expressions as a shorthand for size checks: `some e` (e non-empty), `no e` (e empty), `lone e` (≤1 tuple), `one e` (exactly 1 tuple) — e.g. `some Name` says the set Name is non-empty.

**Classic gotcha** (worth remembering): `one x: Line | cache' = cache - x` does **not** mean "pick one line to remove" — it means "there's exactly one line for which this equation holds," which is false if the cache is empty (since _any_ x would satisfy it vacuously... actually it fails because multiple x's would satisfy it). The correct formulation for "some line can be removed" is `some x: Line | cache' = cache - x`. Don't confuse the **quantifier** with the **multiplicity** of the bound variable — `lone p: some X | F` and `some p: lone X | F` mean very different things.

### 3.5.3 Higher-Order Quantification

Quantified variables need not be scalars — they can be sets or even multirelations. This makes the logic "higher-order." Alloy allows it (e.g. `all s, t: set univ | s + t = t + s` — union is commutative), but the Analyzer generally **cannot automatically check** higher-order formulas except when a technique called **Skolemization** can eliminate the higher-order quantifier by turning it into a free variable.

### 3.5.4 Let Expressions

`let x = e | A` — textually substitutes `e` for `x` everywhere in `A`. Purely a shorthand for factoring out a repeated/complex subexpression — **not recursive** (a `let`-bound variable can't refer to itself or to a `let` binding later in the same chain).

### 3.5.5 Comprehensions

`{x1: e1, x2: e2, ..., xn: en | F}` builds a **new relation** containing every tuple `x1->x2->...->xn` for which `F` holds, with each `xi` ranging over set `ei`. Each `ei` must be a set (unary), not a higher-arity relation.

```
{n: Name | no n.^address & Addr}       -- names that resolve to no real address
{n: Name, a: Addr | n->a in ^address}  -- flattened multilevel lookup table
```

---

## 3.6 Declarations and Multiplicity Constraints

### 3.6.1 Declarations

`relation-name : expression` declares a relation name and bounds its possible values (a subset constraint). Used for quantified variables and (in full Alloy) signature fields.

```
address: Name->Addr                 -- address maps names to addresses
addr: Book->Name->Addr              -- collection of address books
```

Multiple declarations of the same relation can carry different amounts of information — a "stronger" declaration narrows the possible shape further.

### 3.6.2 Set Multiplicities

When the bounding expression is a **set** (unary), you can prefix it with a multiplicity keyword:

|Keyword|Meaning|
|---|---|
|`set`|any number (0+)|
|`one`|exactly one|
|`lone`|zero or one|
|`some`|one or more|

Omitting the keyword on a set-valued bound = implicitly `one` (i.e., declares a **scalar**).

```
RecentlyUsed: set Name    -- a subset
senderAddress: Addr       -- a scalar (implicit "one")
senderName: lone Name     -- an option
receiverAddresses: some Addr -- non-empty subset
```

### 3.6.3 Relation Multiplicities

When the bound is itself built with `->`, multiplicities can appear _inside_ it:

```
r: A m->n B
```

means: each member of `A` maps to `n` members of `B`, and `m` members of `A` map to each member of `B`.

```
r: A->one B      -- function with domain A
r: A one->B      -- injective relation with range B
r: A->lone B     -- partial function over A
r: A one->one B  -- bijection (injective function) between A and B
r: A some->some B -- both sides non-empty on every entry
```

This is just shorthand for two nested `all`/multiplicity quantifications — terser and more readable than spelling them out.

### 3.6.4 Declaration Formulas

The same declaration syntax, but with `in` instead of `:`, is used to **constrain an already-declared or arbitrary expression** rather than introduce a new name:

```
Alias <: address in Alias->lone Addr   -- each alias maps to ≤1 address
```

### 3.6.5 Nested Multiplicities

Multiplicities can nest inside a larger bounding expression:

```
r: A->(B m->n C)   -- for each tuple in A, the B->C sub-relation has multiplicity m->n
r: (A m->n B)->C   -- for each tuple in C, the A->B sub-relation has multiplicity m->n
```

---

## 3.7 Cardinality and Integers

- `#r` — the number of tuples in relation `r` (an integer).
- Integer arithmetic operators: `plus`, `minus`, `mul`, `div`, `rem`.
- Integer comparisons: `=`, `<`, `>`, `=<`, `>=` (note: `=<` not `<=`).

```
all g: Group | #g.address > 1     -- every group has more than one address
```

- `e.sum` — sums a **set of integers** `e` into a single integer.
- `sum x: e | ie` — sums the integer expression `ie` over every `x` drawn from set `e`.

```
all g: split.Group | #g.address = (sum g': g.split | #g'.address)
-- a group's total address count = sum of its subgroups' counts
```

- Design note: arithmetic operators (`=<`, `.plus[...]`, etc.) applied to **sets** of integers act on their **sums**, not element-wise — but plain `=` between two integer sets still checks the sets are identical, not just sum-equal. This creates a subtlety: `S =< T and T =< S` doesn't imply `S = T`, only that their sums are equal.
- `plus`, `minus`, etc. are really predefined ternary relations, so `x.plus[1]` is literally just dot/box join notation, equivalent to `plus[x,1]`.

---

## Quick cross-reference: operator precedence (rough, left-to-right within a tier)

1. Unary: `~` `^` `*` (transpose, closures)
2. Product-like binary: `.` `->` `[]` (join, arrow, box)
3. Sum-like binary: `+` `-` `&` (union, difference, intersection)
4. Comparisons: `in` `=` (and integer comparisons)
5. Logical: `not/!` → `and/&&` → `or/\|\|` → `implies/=>` → `iff/<=>`
6. Quantifiers bind outermost, scoping over everything to their right.

_(Full precedence table is in Appendix B of the book — this is just the shape of it.)_