# Etumos
A programming language combining the best of Julia and Rust

## Motivations
Both the Julia and Rust programming languages are great, truly modern in its usability when compared to some of its predecessors (IMO). The Julia programming language is great due to its flexibility, all the benefits of dynamic polymorphism is realized due to its function dispatch, and the performance is great too. But because of this, the benefits of functional types and type inference at instantiation time is moot, so loads of run time errors. On the other hand, Rust, because of HM, has great static type properties, I especially like the functional types. But the lack of flexibility in its type system makes polymorphism verbose and difficult; I would not like to use macros to handle this (maybe if macros are more structured, rather than operating on strings?). An example of this is array types as defined by the ndarray crate. Dependent types in array dimensions are especially crippling (annoying to extend and use imo). 

### Goals
I would like to make a **statically typed**, *(purely?)* **functional** language with **no GC**, that is on the same order of performance as Rust, C++, etc. I wish systems level languages have the same type system as Haskell, and certain features, such as shared ownership, can be removed to bridge the gap without loss of generality.

Perhaps the closest language to my vision of this one is [Idris 2](https://github.com/idris-lang/Idris2), from which I will probably take a few ideas, such as [QTT](http://www.t-news.cn/Floc2018/FLoC2018-pages/proceedings_paper_665.pdf). In particular, linear dependent types can be used to handle mutable state, without monads. While the blurring of lines between compile time and run time can enable far more flexibility and expressibility in the language. Perhaps enforcing invariants such as commutativity, associativity, etc.

I would also like to test out some ideas in optimization, perhaps having machine learned heuristic functions, but that is in the long term.
