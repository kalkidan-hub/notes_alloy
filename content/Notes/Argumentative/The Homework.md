

> Problem: An Alloy model for cyclic list: 

> Solution: 
> ```
> alloy
> 	   abstract sig Node { nxt: lone Node }
		one sig List { hd: Node }
		fact {
		Node = List.hd.*nxt
		}
		pred Cyclic[l: List] {
		l.hd in l.hd.^nxt
		}
> ```


> Problem with the proposed solution: 
> It introduces symmetry instances. For instance when `run {Cyclic[List]} for 13` produces these [[Isomorphic|isomorphic]] instances:
> ![[Pasted image 20260807112817.png|166]]    ![[Pasted image 20260807112845.png|141]]

> Solution: 
> Introducing [[Total Ordering]] to the model. 
> Hence the model would look like this ...

```open util/ordering[Node] as ord

abstract sig Node { nxt: lone Node }
one sig List { hd: Node }

fact {
  Node = List.hd.*nxt
}

pred Cyclic[l: List] {
  l.hd in l.hd.^nxt
}

// Symmetry break: hd is the first atom in the order,
// and nxt follows the order, wrapping from last back to first.
fact SymmetryBreak {
  List.hd = ord/first
  all n: Node - ord/last | n.nxt = ord/next[n]
  ord/last.nxt = ord/first
}

run { Cyclic[List] } for 6
```

