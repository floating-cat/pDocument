https://free.cofree.io

# How Trampoline Works in Scala
sealed trait TailRec[A] {
  def map[B](f: A => B): TailRec[B] = flatMap(f andThen (Return(_)))
  def flatMap[B](f: A => TailRec[B]): TailRec[B] = FlatMap(this, f)
}

final case class Return[A](a: A) extends TailRec[A]
final case class Suspend[A](resume: () => TailRec[A]) extends TailRec[A]
final case class FlatMap[A, B](sub: TailRec[A], k: A => TailRec[B]) extends TailRec[B]

It should now be obvious why this technique is called “trampoline”: during the execution of run(fac(5)), we keep jumping back and forth between run and fac.

The key reason why trampolined functions are stack-safe is because Suspend is lazy, in other words, the recursion happens in a lazy structure. Generally speaking, lazy recursions tend to be stack-safe, even if they are not tail recursions. For example, the following function:

def func(x: Int): Stream[Int] = x #:: func(x + 1)
is stack safe, even though it is not tail recursive, because the tail of a Stream is lazy. Calling

println(func(1).take(n).toList)
with a large n will not cause StackOverflowError. The execution simply trampolines between func and take.

# The fix Combinator in Scalaz
def fix[A](f: (=> A) => A): A = {
  lazy val a: A = f(a)
  a
}
fix 使用一个伪递归的函数 f，可以获得到这个递归的函数。f f 也可以获得改函数。
Y f = f (Y f)

val gac: (=> Int => Int) => Int => Int =
  fac => n => if (n == 0) 1 else n * fac.apply(n - 1)


# Stream, Laziness and Stack Safety
def take(n: Int): Stream[A] = this match {
  case Cons(h, t) if n > 1 => cons(h(), t().take(n - 1))
  case Cons(h, _) if n == 1 => cons(h(), empty)
  case _ => empty
}

In take, the recursion happens in the stream constructor cons, which is lazy in both parameters, so we can start evaluaing cons without evaluating h() or t().take(n - 1).

In fact, this is exactly how trampolining works.

Suppose we have an infinite stream s: Stream[Int] = 1, 2, 3,... and let’s see what happens when we call s.take(5).toList.

Since s matches the first case in take, we call cons(h(), Stream(2, 3,...).take(4)), without evaluating h() and t().toList, which returns Cons(() => h(), () => Stream(2, 3,...).take(4)). 

Evaluating tail() means evaluating Stream(2, 3,...).take(4). Similar as before, it is evaluated into Cons(() => h(), () => Stream(3, 4,...).take(3)). The execution continues till we have taken 5 elements from the original stream. During this process, we alternate between the execution of take and toList, in other words, we are trampoling.

This is also why it doesn’t create an intermediate list when we call Stream(1, 2, 3, 4).map(_ + 10).filter(_ % 2 == 0).toList. It basically trampolines between map, filter and toList, and processes one element at a time.

# Free Monoids and Free Monads, Free of Category Theory
“Free” in free monoids and free monads is generally taken to mean “structure free”, although I’d like to think of it as “free of unnecessary information loss”. In other words, a “free something” should be the “something” that preserves the most information, or the most general “something”, among all “something”s.

The set associated with monoid is called its underlying set. When we talk about the most general monoid, we need to specify a particular set, as in the most general monoid for Int, the most general monoid for Boolean, etc.

sealed trait List[A]
final case class Nil[A]() extends List[A]
final case class Cons[A](head: A, tail: List[A]) extends List[A]

Change types to functors in the definition of List to get a free monad:
sealed trait Free[F[_], A]
final case class Return[F[_], A](a: A) extends Free[F, A]
final case class Bind[F[_], A](ffa: F[Free[F, A]]) extends Free[F, A]

Another free monad definition:
sealed trait Free[F[_], A]
final case class Return[F[_], A](a: A) extends Free[F, A]
final case class Suspend[F[_], A](s: F[A]) extends Free[F, A]
final case class FlatMap[F[_], A, B](s: Free[F, A], f: A => Free[F, B]) extends Free[F, B]

The advantage of this definition is that it is a monad for any type constructor F (also known as “freer monad”), since it is basically equivalent to a free monad combined with a free functor. On the other hand, in the previous definition, Free[F, A] is a monad only if F is a functor.

The join operation on a monad is comparable to the combine operator on a monoid. 
https://stackoverflow.com/a/13388966/2331527
Here's an even simpler answer: A Monad is something that "computes" when monadic context is collapsed by join :: m (m a) -> m a (recalling that >>= can be defined as x >>= y = join (fmap y x)). This is how Monads carry context through a sequential chain of computations: because at each point in the series, the context from the previous call is collapsed with the next.

A free monad satisfies all the Monad laws, but does not do any collapsing (i.e., computation). It just builds up a nested series of contexts. The user who creates such a free monadic value is responsible for doing something with those nested contexts, so that the meaning of such a composition can be deferred until after the monadic value has been created.
