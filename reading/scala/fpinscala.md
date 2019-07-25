# Chapter 1

Consider what programming would be like without the ability to do these things, or with significant restrictions on when and how these actions can occur. It may be difficult to imagine. How is it even possible to write useful programs at all? If we can’t reassign variables, how do we write simple programs like loops? What about working with data that changes, or handling errors without throwing exceptions? How can we write programs that must perform I/O, like drawing to the screen or reading from a file?
The answer is that functional programming is a restriction on how we write programs, but not on what programs we can express.

Separate from the concern of testing, there’s another problem: it’s difficult to reuse buyCoffee. Suppose a customer, Alice, would like to order 12 cups of coffee. Ideally we could just reuse buyCoffee for this, perhaps calling it 12 times in a loop. But as it is currently implemented, that will involve contacting the payment system 12 times, authorizing 12 separate charges to Alice’s credit card! That adds more processing fees and isn’t good for Alice or the coffee shop.

```scala
// before
def buyCoffee(cc: CreditCard, p: Payments): Coffee = {
  val cup = new Coffee()
  p.charge(cc, cup.price)
  cup
}

// after
def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
  val cup = new Coffee()
  (cup, Charge(cc, cup.price))
}
```

Here we’ve separated the concern of creating a charge from the processing or interpretation of that charge. The buyCoffee function now returns a Charge as a value along with the Coffee.

We saw in the case of buyCoffee how we could separate the creation of the Charge from the interpretation or processing of that Charge. In general, we’ll learn how this sort of transformation can be applied to any function with side effects to push these effects to the outer layers of the program. Functional programmers often speak of implementing programs with a pure core and a thin layer on the outside that handles effects.

## Referential transparency and purity

An expression e is referentially transparent if, for all programs p, all occurrences of e in p can be replaced by the result of evaluating e without affecting the meaning of p. A function f is pure if the expression f(x) is referentially transparent for all referentially transparent x.

Referential transparency forces the invariant that everything a function does is represented by the value that it returns, according to the result type of the function. This constraint enables a simple and natural mode of reasoning about program evaluation called the substitution model. When expressions are referentially transparent, we can imagine that computation proceeds much like we’d solve an algebraic equation. We fully expand every part of an expression, replacing all variables with their referents, and then reduce it to its simplest form. At each step we replace a term with an equivalent one; computation proceeds by substituting equals for equals. In other words, RT enables equational reasoning about programs.

```scala
val x = new StringBuilder("Hello")
val y = x.append(", World")
val r1 = y.toString
val r2 = y.toString
```

```scala
val x = new StringBuilder("Hello")
val y = x.append(", World")
val r1 = x.append(", World").toString
val r2 = x.append(", World").toString
```

# Chapter 2

We say that MyModule is its namespace. An object whose primary purpose is giving its members a namespace is sometimes called a module.

It’s a common convention to use names like f, g, and h for parameters to a higher-order function. In functional programming, we tend to use very short variable names, even one-letter names. This is usually because HOFs are so general that they have no opinion on what the argument should actually do. All they know about the argument is its type. Many functional programmers feel that short names make code easier to read, since it makes the structure of the code easier to see at a glance.

We can call the type parameters anything we want—[Foo, Bar, Baz] and [TheParameter, another_good_one] are valid type parameter declarations—though by convention we typically use short, one-letter, uppercase type parameter names like [A,B,C].

We saw how the implementations of polymorphic functions are often significantly constrained, such that we can often simply “follow the types” to the correct implementation.

# Chapter 3

We could call x and xs anything there, but it’s a common convention to use xs, ys, as, or bs as variable names for a sequence of some sort, and x, y, z, a, or b as the name for a single element of a sequence. Another common naming convention is h for the first element of a list (the head of the list), t for the remaining elements (the tail), and l for an entire list.

```scala
// get back the origin list
foldRight(List(1,2,3), Nil:List[Int])(Cons(_,_))
```

```scala
def foldRightViaFoldLeftWithoutReverse[A, B](xs: List[A], z: B)(f: (B, A) => B): B =
  xs.foldLeft(identity[B] _)((g, a) => b => g(f(b, a)))(z)

def foldLeftViaFoldRightWithoutReverse[A, B](xs: List[A], z: B)(f: (B, A) => B): B =
  xs.foldRight(identity[B] _)((a, g) => b => g(f(b, a)))(z)
```

## Loss of efficiency when assembling list functions from simpler components

One of the problems with List is that, although we can often express operations and algorithms in terms of very general-purpose functions, the resulting implementation isn’t always efficient—we may end up making multiple passes over the same input, or else have to write explicit recursive loops to allow early termination.

List is just one example of what’s called an algebraic data type (ADT). (Somewhat confusingly, ADT is sometimes used elsewhere to stand for abstract data type.) An ADT is just a data type defined by one or more data constructors, each of which may contain zero or more arguments. We say that the data type is the sum or union of its data constructors, and each data constructor is the product of its arguments, hence the name algebraic data type.

One might object that algebraic data types violate encapsulation by making public the internal representation of a type. In FP, we approach concerns about encapsulation differently—we don’t typically have delicate mutable state which could lead to bugs or violation of invariants if exposed publicly. Exposing the data constructors of a type is often fine, and the decision to do so is approached much like any other decision about what the public API of a data type should be.

We do typically use ADTs for situations where the set of cases is closed (known to be fixed).

```scala
// fold for tree
def fold[A, B](t: Tree[A])(f1: A => B)(f2: (B, B) => B): B = t match {
  case x: Leaf[A] => f1(x.value)
  case x: Branch[A] => f2(fold(x.left)(f1)(f2), fold(x.right)(f1)(f2))
}
```

The map function lets us operate on values of type Option[A] using a function of type A => B, returning Option[B]. Another way of looking at this is that map turns a function f of type A => B into a function of type Option[A] => Option[B]. This tells us that any function that we already have lying around can be transformed (via lift) to operate within the context of a single Option value.

map2: (F[A], F[B]) => ()A, B => C) = F[C]
map2 = flatMap + map
map3 = flatMap + flatMap + map
(this is also how `for-comprehension` works in Scala)

sequence: List[F[A]] => F[List[A]]
traverse: List[A] => (A => F[B]) => F[List[B]]
sequence = traverse + identity

This is a clear instance where it’s not appropriate to define the function in the OO style. This shouldn’t be a method on List (which shouldn’t need to know anything about F), and it can’t be a method on F.

```scala
// this is inefficient, since it traverses the list twice, so we create traverse
def parseInts(a: List[String]): Option[List[Int]] =
  sequence(a map (i => Try(i.toInt)))
```

# Chapter 5

A value of type () => A is a function that accepts zero arguments and returns an A. In general, the unevaluated form of an expression is called a thunk, and we can force the thunk to evaluate the expression and get a result.

## Formal definition of strictness

If the evaluation of an expression runs forever or throws an error instead of returning a definite value, we say that the expression doesn’t terminate, or that it evaluates to bottom. A function f is strict if the expression f(x) evaluates to bottom for all x that evaluate to bottom.

As a final bit of terminology, we say that a non-strict function in Scala takes its arguments by name rather than by value.

```scala
// this code will actually compute expensive(x) twice
val x = Cons(() => expensive(x), tl)
val h1 = x.headOption
val h2 = x.headOption
```
We typically avoid this problem by defining smart constructors, which is what we call a function for constructing a data type that ensures some additional invariant or provides a slightly different signature than the “real” constructors used for pattern matching.

# Separating program description from evaluation

A major theme in functional programming is separation of concerns. We want to separate the description of computations from actually running them. We’ve already touched on this theme in previous chapters in different ways. For example, first-class functions capture some computation in their bodies but only execute it once they receive their arguments. And we used Option to capture the fact that an error occurred, where the decision of what to do about it became a separate concern. With Stream, we’re able to build up a computation that produces a sequence of elements without running the steps of that computation until we actually need those elements.

More generally speaking, laziness lets us separate the description of an expression from the evaluation of that expression. This gives us a powerful ability—we may choose to describe a “larger” expression than we need, and then evaluate only a portion of it.

We don’t fully instantiate the intermediate stream that results from the map. It’s exactly as if we had interleaved the logic using a special-purpose loop. For this reason, people sometimes describe streams as “first-class loops” whose logic can be combined using higher-order functions like map and filter.

## Infinite streams

```scala
val ones: Stream[Int] = Stream.cons(1, ones)
```

# Corecursion

```scala
def unfold[A, S](z: S)(f: S => Option[(A, S)]): Stream[A]
```

Option is used to indicate when the Stream should be terminated, if at all. The function unfold is a very general Stream-building function.
The unfold function is an example of what’s sometimes called a corecursive function. Whereas a recursive function consumes data, a corecursive function produces data. And whereas recursive functions terminate by recursing on smaller inputs, corecursive functions need not terminate so long as they remain productive, which just means that we can always evaluate more of the result in a finite amount of time. The unfold function is productive as long as f terminates, since we just need to run the function f one more time to generate the next element of the Stream. Corecursion is also sometimes called guarded recursion, and productivity is also sometimes called cotermination.

Using unfold to define constant and ones means that we don’t get sharing as in the recursive definition val ones: Stream[Int] = cons(1, ones). The recursive definition consumes constant memory even if we keep a reference to it around while traversing it, while the unfold-based implementation does not. Preserving sharing isn’t something we usually rely on when programming with streams, since it’s extremely delicate and not tracked by the types. For instance, sharing is destroyed when calling even xs.map(x => x)

we introduced non-strictness as a fundamental technique for implementing efficient and modular functional programs. Non-strictness can be thought of as a technique for recovering some efficiency when writing functional code, but it’s also a much bigger idea—non-strictness can improve modularity by separating the description of an expression from the how-and-when of its evaluation. Keeping these concerns separate lets us reuse a description in multiple contexts, evaluating different portions of our expression to obtain different results. We weren’t able to do that when description and evaluation were intertwined as they are in strict code.

# Chapter 6
Because the state updates are performed as a side effect, these methods aren’t referentially transparent. And as we know, this implies that they aren’t as testable, composable, modular, and easily parallelized as they could be.

Note that what’s important here is not this specific example, but the general idea. In this case, the bug is obvious and easy to reproduce. But we can easily imagine a situation where the method is much more complicated and the bug far more subtle. The more complex the program and the subtler the bug, the more important it is to be able to reproduce bugs in a reliable way.

We return the random number and the new state, leaving the old state unmodified. In effect, we separate the concern of computing what the next state is from the concern of communicating the new state to the rest of the program.

An efficiency loss comes with computing next states using pure functions, because it means we can’t actually mutate the data in place. (Here, it’s not really a problem since the state is just a single Long that must be copied.) This loss of efficiency can be mitigated by using efficient purely functional data structures. It’s also possible in some cases to mutate the data in place without breaking referential transparency.

As you write more functional programs, you’ll sometimes encounter situations like this where the functional way of expressing a program feels awkward or tedious. Does this imply that purity is the equivalent of trying to write an entire novel without using the letter E? Of course not. Awkwardness like this is almost always a sign of some missing abstraction waiting to be discovered.
When you encounter these situations, we encourage you to plow ahead and look for common patterns that you can factor out. Most likely, this is a problem that others have encountered, and you may even rediscover the “standard” solution yourself. Even if you get stuck, struggling to puzzle out a clean solution yourself will help you to better understand what solutions others have discovered to deal with similar problems.

Our functions has a type of the form RNG => (A, RNG) for some type A. Functions of this type are called state actions or state transitions because they transform RNG states from one to the next. These state actions can be combined using combinators, which are higher-order functions that we’ll define in this section. Since it’s pretty tedious and repetitive to pass the state along ourselves, we want our combinators to pass the state from one action to the next automatically.

```scala
def map2[A, B, C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
  rng => {
    val (n, rng2) = ra(rng)
    val (n2, rng3) = rb(rng2)
    (f(n, n2), rng3)
}
```

```scala
def nonNegativeLessThan(n: Int): Rand[Int] =
  map(nonNegativeInt) { _ % n }
```

This will certainly generate a number in the range, but it’ll be skewed because Int.MaxValue may not be exactly divisible by n. So numbers that are less than the remainder of that division will come up more frequently. When nonNegativeInt generates numbers higher than the largest multiple of n that fits in a 32-bit integer, we should retry the generator and hope to get a smaller number.

```scala
def nonNegativeLessThan(n: Int): Rand[Int] = { rng =>
  val (i, rng2) = nonNegativeInt(rng)
  val mod = i % n
  if (i + (n-1) - mod >= 0)
    (mod, rng2)
  else nonNegativeLessThan(n)(rng)
}
```

But it would be better to have a combinator that does this passing along for us. Neither map nor map2 will cut it. We need a more powerful combinator, flatMap.

We can reimplement map and map2 in terms of flatMap. The fact that this is possible is what we’re referring to when we say that flatMap is more powerful than map and map2.

```scala
type State[S,+A] = S => (A,S)
// or
// type State[S,+A] = S => (A,S)
```

Here State is short for computation that carries some state along, or state action, state transition, or even statement (see the next section).

The representation doesn’t matter too much. What’s important is that we have a single, general-purpose type, and using this type we can write general-purpose functions for capturing common patterns of stateful programs.

# Purely functional imperative programming

We’d run a state action, assign its result to a val, then run another state action that used that val, assign its result to another val, and so on. It looks a lot like imperative programming.

In the imperative programming paradigm, a program is a sequence of statements where each statement may modify the program state. That’s exactly what we’ve been doing, except that our “statements” are really State actions, which are really functions. As functions, they read the current program state simply by receiving it in their argument, and they write to the program state simply by returning a value.

Aren’t imperative and functional programming opposites?

Absolutely not. Remember, functional programming is simply programming without side effects. Imperative programming is about programming with statements that modify some program state, and as we’ve seen, it’s entirely reasonable to maintain state without side effects.

Functional programming has excellent support for writing imperative programs, with the added benefit that such programs can be reasoned about equationally because they’re referentially transparent.

```scala
def modify[S](f: S => S): State[S, Unit] =
  for {
    s <- get
    _ <- set(f(s))
  } yield ()
```

modify: S => S => State[S, Unit]
