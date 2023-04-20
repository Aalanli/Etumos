---
word: "test"
---
# Etumos
Explorations in programming language design

**Goals**: To make a language that is more expressive, more scalable/composable, as safe as, as performant as its current modern day counterparts. (Rust, Julia, Haskell)

**Definitions** *ad hoc*
Language A is more expressive than language B when on average, it requires less and more readable lines of code to capture a given feature/concept. 

Language A is more readable than language B when on average, it requires less time to understand a concept implemented in the language; requiring less time to be able to predict the outputs of a program.

Language A is safer than language B when on average, there are fewer things to remember to write a program which has the intended behavior.

## Observations
### Rust
I enjoy Rust, far more than C++ anyhow. Probably the most enjoyable aspects of it is the type system. Which ensures correctness by construction and solid memory semantics. Type inferance also improves productivity and scalability. However, there are several aspects which I would change xor wish Rust had. 

1. Trait bounds. Some libraries requires heavy trait bounds to interface with the system. At best, this is very verbose, and not very extensible. At worst, the traits defined are brittle and leaky. Writing generic functions with (many) trait bounds is also very painful. 
	- Just in general, I am dissatisfied with Rust's generics. The lack of const generics, and brittleness of traits makes things uncomfortable some of the time.
```Rust
// wish this was possible
trait A {
	fn foo(&self) -> impl Clone;
}

trait B {
	fn bar(&self, other: &impl A) -> impl B;
}

// I am aware this is possible, but I rarely do this, as large trait bounds,
// with many generic parameters can get truly monsterous.
trait C: A, B {}
impl<T: A, B> C for T {}
```
2. Macros. I wish macros could gather the AST.

```Rust
fn foo(a: i32, b: i32) -> {
	if b == 0 { 0 }
	else if (a % b == 0) { b }
	else { foo(b, a % b) }
}

fn main() {
	dump_ast!(foo);
}

// outputs the body of foo
```

3. Lifetimes. In some cases, lifetimes are too restricting, and unsafe is needed.

```Rust
struct Foo<'a>(Vec<&'a mut f32>);

fn main() {
	let mut a = Foo(Vec::new());
	let mut b = 1.0;
	a.0.push(&mut b);
	// some computations later
	a.0.clear();
	b += 1.0; // error, the type of foo still has the lifetime of b
}
```

```Rust
// not possible, which makes sense, but is frustating
fn wtf<'a, 'b, 'c, 'd, 'e>(v: &'a mut &'b mut &'c mut &'d mut &'e mut [f32]) {
	let h: &mut [f32] = ****v;
}
```

4. Mental overhead. I don't think rust is necessary for most things; I find myself agonizing over the use of Rc/Arc when it is default in some programming languages



### Julia
Julia is a nice language. I like its type system, tools and consistency. However, I don't think unrestricted dynamic dispatch is beneficial long term. Informal interfaces are error prone. GC and runtime does not make it general-purpose enough. 

When I write something in Rust or Haskell, almost always, it runs the first time. That cannot be said for Julia.

Addtionally, like Nim, I wish I could turn off the gc. I wish there were mutable structs on the stack, or at least let mutable structs be flattened in an array.



## Motivations
Both the Julia and Rust programming languages are great, truly modern in its usability when compared to some of its predecessors (IMO). The Julia programming language is great due to its flexibility, all the benefits of dynamic polymorphism is realized due to its function dispatch, and the performance is great too. But because of this, the benefits of functional types and type inference at instantiation time is moot, so loads of run time errors. On the other hand, Rust, because of HM, has great static type properties, I especially like the functional types. But the lack of flexibility in its type system makes polymorphism verbose and difficult; I would not like to use macros to handle this (maybe if macros are more structured, rather than operating on strings?). An example of this is array types as defined by the ndarray crate. Dependent types in array dimensions are especially crippling (annoying to extend and use imo). 

### Goals
I would like to make a **statically typed**, *(purely?)* **functional** language with **no GC**, that is on the same order of performance as Rust, C++, etc. I wish systems level languages have the same type system as Haskell, and certain features, such as shared ownership, can be removed to bridge the gap without loss of generality.

Perhaps the closest language to my vision of this one is [Idris 2](https://github.com/idris-lang/Idris2), from which I will probably take a few ideas, such as [QTT](http://www.t-news.cn/Floc2018/FLoC2018-pages/proceedings_paper_665.pdf). In particular, linear dependent types can be used to handle mutable state, without monads. While the blurring of lines between compile time and run time can enable far more flexibility and expressibility in the language. Perhaps enforcing invariants such as commutativity, associativity, etc.

I would also like to test out some ideas in optimization, perhaps having machine learned heuristic functions, but that is in the long term.

## Rough outline
- There are no types, everything is a 'thing' built up (composed) from primitive 'things'
- structs are simply functions from names to 'things'
- the compiler is a thing exposed to the program
- the compiler 'thing' is primitive

### There are no types
```rust
struct A {
	a: i32,
	b: f64
}

let a = A {a: 0, b: 0.0};
```
Is the same as
```haskell
a "a" = 0
a "b" = 0.0
```
where $A \{\text{a}, \text{b}\} \to \{\text{i32}\} \cup \{\text{f64}\}$. 

Structs of structs are simply functions to functions.
Well, actually there are types, but the line between compiled and runtime is significantly blurred. Types are names for sets (with additional structure), and these 'names' are first class values, but erased during compile time.

The compiler defines primitive types $\{\to, (), \emptyset, \text{i32}, \text{i64}, \text{u32},\text{u64},\text{f32},\text{f32},\text{bool}\}$ and some operators on these primitive types $\{\subset, =, +, -, *, \dots\}$

```haskell
A = {
	a: i32, 
	b: f64
}
```
Can be syntactic sugar for $A: \text{i32} \times \text{f64} \to (\text{name} \to P)$ where $P$ is something like the union of i32 and i64.

A preorder can be defined for these types, for example, we have $A \leq \to$, $0 \leq \text{i32}$. 
A arrow exists for $()$ to everything, it is the initial type, while everything as an arrow to $\emptyset$, the terminal type. *(This is important to polymorphism)*

Ordinarily in Rust or C++, if we want compile time variables, we must do this:
```rust
struct A<const N: i32> {
	a: i32,
	b: i64
}
```
```c++
template <int N>
struct A {
	int a;
	double b;
};
```

But we can simply do this
```haskell
A' = {
	a: i32,
	b: f64,
	N: i32
}
A a b = A' a b 0
```
Of course, we can create these functions as we wish, and they are equivalent to the above. The compiler can prove that certain 'things' are known at compile time.
In this case $A \leq A'$. 

We can define
```haskell
(..) = {
	a: Int,
	b: Int
}
```
Where Int is the parent of all integer types, and $(..) \leq \text{Int}$. Ideally, we would want (..) specialized to specific int variants to also be a subset of their respective integer variants. Ex.
```haskell
(..i32) = {
	a: i32,
	b: i32
}
(..i32) <= i32
```

With this, we can define arrays as a primitive type $[T]: S \to T, S = (a..b) = \{i \in \mathbb{Z}, a \leq i < b\}$.
With the introduction of arrays, we must define a linear type and a special 'bind' type, and 'split' type. 

...
The linear type should be entirely implementable in the code itself via the primitive compiler 'thing'.


## Sources
[Syntax and Semantics of Quantitative Type Theory](http://www.t-news.cn/Floc2018/FLoC2018-pages/proceedings_paper_665.pdf)

[Idris 2: Quantitative Type Theory in Practice](https://arxiv.org/pdf/2104.00480.pdf)