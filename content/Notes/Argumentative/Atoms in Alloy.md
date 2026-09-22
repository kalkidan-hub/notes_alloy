An **atom** is the most basic building block of a model. It is:

- **indivisible** — can't be broken into smaller parts
	- Example: A person who has address. 
	- If the *Address* is made of a *Street*, *City* and *Zip*, you don't open up the *Address* atom[coz it's indivisible] and add these entities to it, you rather add new atoms for these entities and add relations of these atoms with the *Address* atom to the model.
	- ```
		  street: Address -> one Street
		  city: Address -> one City
		  zip: Address -> one Zip
	  ```
	  ```alloy
	  sig Address {} -- this is how atom is defined
	  ```
	  ```alloy
	    sig Street {}
		sig City {}
		sig Zip {}
		
		sig Address {
		  street: one Street,
		  city:   one City,
		  zip:    one Zip
		}
	  ```
	  
- **immutable** — its properties don't change over time
	- Example: a bank account balance that changes
	- To model such scenarios, instead of using one atom whose "value" field updates [coz an atom is immutable], you introduce separate atoms for identity and for value change over time and ofc a relation connecting them.
	- ```
	  Account   -- the identity, one atom per account, never changes
	  Balance   -- a set of "balance snapshot" atoms
	  balanceAt: Account -> Balance -> Time
	  ```
	  ```alloy
	    sig Account {}
		sig Balance {}
		sig Time {}
		
		sig BalanceAt {
		  acct: one Account,
		  bal:  one Balance,
		  time: one Time
		}
	  ```
	- To have a collection of thus atoms: `value = {(A0,B0,T0), (A0,B1,T1)}`
	- *This is like having collection of atoms[real physical ones] of same type with different physical properties, like temperature, velocity or so.* 
- **uninterpreted** — no built-in meaning 
	- Example: modeling "even numbers" without built-in arithmetic
	- Alloy atoms have zero built-in meaning — an atom named `N3` doesn't inherently "know" it's the number 3, or that it's odd. If you want atoms to behave like integers with structure (successor, evenness, ordering), you don't bake that into the atom — you add a relation that gives it that interpretation from outside:
	- ```
		Number
		next: Number -> lone Number-- successor relation, imposes an ordering
		even: set Number -- a subset you constrain via 'next'
	  ```
	  ```alloy
		  sig Number {
		  next: lone Number
		  }
		  one sig Zero extends Number {}
			
		  fun even[]: set Number {
			  Zero.*(next.next)
		  }
	  ```
	  