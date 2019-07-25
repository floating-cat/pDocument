Order > PartialOrder > Eq

# Functor
map = andThen (Haskell: compose)
> That’s a common Haskell-vs-Scala difference.
In Haskell, to help with point-free programming, the “data” argument usually comes last. For instance, I can write map f . map g . map h and get a list transformer, because the argument order is map f list. (Incidentally, map is an restriction of fmap to the List functor).
In Scala instead, the “data” argument is usually the receiver. That’s often also important to help type inference, so defining map as a method on functions would not bring you very far: think the mess Scala type inference would make of (x => x + 1) map List(1, 2, 3).

> We can think of fmap as] a function that takes a function and returns a new function that’s just like the old one, only it takes a functor as a parameter and returns a functor as the result. It takes an a -> b function and returns a function f a -> f b. This is called lifting a function.
laws:
fa.map(identity) <-> fa
fa.map(f).map(g) <-> fa.map(f andThen g)


:k Int
Int and every other types that you can make a value out of are called a proper type and denoted with a * symbol (read “type”). This is analogous to the value 1 at value-level. Using Scala’s type variable notation this could be written as A.
:k -v Option
proper type, 1st-order-kinded type, higher-kinded type
HKT is a type constructor that accepts another type constructor. This is analogous to a higher-order function, and thus called higher-kinded type.

We say things like “is X a monoid?” to mean “can X form a monoid under some operation?

# Semigroupal
not related: Functor[List].map(List(1, 2, 3, 4)) ({(_: Int) * (_:Int)}.curried)

Just (3 *) and a functor value of Just 5, and we want to take out the function from Just(3 *) and map it over Just 5.
def product[A, B](fa: F[A], fb: F[B]): F[(A, B)]
Semigroupal defines product function, which produces a pair of (A, B) wrapped in effect F[_] out of F[A] and F[B].

associativity law:
(F.product(fa, F.product(fb, fc)), F.product(F.product(fa, fb), fc))
(x |+| y) |+| z = x |+| (y |+| z)

# Apply
//  <*>
def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
(3.some, 5.some) mapN { _ - _ }
*>,<* (productL, productR), mapN
def ap2[A, B, Z](ff: F[(A, B) => Z])(fa: F[A], fb: F[B]): F[Z]
Apply[Option].tuple2(1.some, 2.some)

(not related) 9.some, none[Int]

composition law:
def applyComposition[A, B, C](fa: F[A], fab: F[A => B], fbc: F[B => C]): IsEq[F[C]] = {
  val compose: (B => C) => (A => B) => (A => C) = _.compose
  fbc.ap(fab.ap(fa)) <-> fa.ap(fab.ap(fbc.map(compose)))
}

# Applicative >  Apply > Semigroupal
def pure[A](x: A): F[A]
def sequenceA[F[_]: Applicative, A](list: List[F[A]]): F[List[A]] (with pure and mapN)

laws
identity: pure id <*> v = v
homomorphism: pure f <*> pure x = pure (f x)
interchange: u <*> pure y = pure ($ y) <*> u
Cats defines another law:
fa.map(f) <-> fa.ap(F.pure(f)) => map = ap + pure

# Monoid
laws
associativity (x |+| y) |+| z = x |+| (y |+| z)
left identity Monoid[A].empty |+| x = x
right identity x |+| Monoid[A].empty = x

Value classes
The newtype keyword in Haskell is made exactly for these cases when we want to just take one type and wrap it in something to present it as another type. We can use value classes for simple examples.

scala> none[String] |+| "andy".some
res4: Option[String] = Some(andy)

Like traffic laws dictating that we drive on one side within a patch of land, some laws are convenient if everyone follows them.
For the sake of learning, I think it’s ok to start out with cargo cult. We all learn language through imitation and pattern recognition.

Foldable
foldMap： def foldMap[A, B](fa: F[A])(f: A => B)(implicit B: Monoid[B]): B

# FlatMap
Cats breaks down the Monad typeclass into two typeclasses: FlatMap and Monad.
@typeclass trait FlatMap[F[_]] extends Apply[F]
def mproduct[B](f: A => F[B]): F[(A, B)]
def >>=[B](f: A => F[B]): F[B]
def >>[B](fb: F[B]): F[B]

laws:
associativity: (m flatMap f) flatMap g === m flatMap { x => f(x) flatMap {g} }
Cats defines two more laws:
fab.ap(fa) <-> fab.flatMap(f => fa.map(f)) => ap = flatmap + map
def kleisliAssociativity[A, B, C, D](f: A => F[B], g: B => F[C], h: C => F[D], a: A): IsEq[F[D]] = {
  val (kf, kg, kh) = (Kleisli(f), Kleisli(g), Kleisli(h))
  ((kf andThen kg) andThen kh).run(a) <-> (kf andThen (kg andThen kh)).run(a)
}

# Monad
incorrect because of operator precedence:
Monad[Option].pure(Pole(0, 0)) >>= {_.landLeft(1)} >> none[Pole] >>= {_.landRight(1)}

laws
left identity: (Monad[F].pure(x) flatMap {f}) === f(x)
right identity: (m flatMap {Monad[F].pure(_)}) === m
associativity: (m flatMap f) flatMap g === m flatMap { x => f(x) flatMap {g} }
Monad is a special kind of a monoid.

# Writer datatype
Whereas the Maybe monad is for values with an added context of failure, and the list monad is for nondeterministic values, Writer monad is for values that have another value attached that acts as a sort of log value.

type Writer[L, V] = WriterT[Id, L, V]
def value[L:Monoid, V](v: V): Writer[L, V] = WriterT.value(v)
def tell[L](l: L): Writer[L, Unit] = WriterT.tell(l)

We’ve also seen that functions are applicative functors. They allow us to operate on the eventual results of functions as if we already had their results. => map2
val f = (_: Int) * 2
val g = (_: Int) + 10
(g map f)(8)
val addStuff: Int => Int = for {
         a <- (_: Int) * 2
         b <- (_: Int) + 10
       } yield a + b

not related:
def getUser(id: Long): UserRepo => User = {
    case repo => repo.get(id)
}

# State
type State[S, A] = StateT[Eval, S, A]
State partially applies StateT with Eval, which emulates a call stack with heap memory to prevent overflow.
not ture: final class StateT[F[_], S, A](val runF: F[S => F[(S, A)]])
def run(initial: S)(implicit F: FlatMap[F]): F[(S, A)]

def pure[S, A](a: A): State[S, A] = State(s => (s, a))
def modify[S](f: S => S): State[S, Unit] = State(s => (f(s), ()))
def inspect[S, T](f: S => T): State[S, T] = State(s => (s, f(s)))
def get[S]: State[S, S] = inspect(identity)

# Validated
fold
Validation is that it is does not form a monad, but forms an applicative functor.
valid[String, String]("event 1 ok")
invalid[String, String]("event 2 failed!")

NonEmptyList

# Ior(逻辑或)
Ior.left short curcuits like the failure values in Xor[A, B] and Either[A, B], but Ior.both accumulates the failure values like Validated[A, B].

# Free monads
data Free f a = Pure a | Free (f (Free f a))
case class Fix[F[_]](f: F[Fix[F]])
sealed abstract class Free[S[_], A] extends Product with Serializable

type FreeMonoid[A] = Free[(A, +?), Unit]
def cons[A](a: A): FreeMonoid[A] =
         Free.liftF[(A, +?), Unit]((a, ()))
val xs = cons(1) flatMap {_ => cons(2)}
def toList[A](list: FreeMonoid[A]): List[A] =
  list.fold(
    { _ => Nil },
    { case (x: A @unchecked, xs: FreeMonoid[A]) => x :: toList(xs) })

# Stack Safty
flatmap => not stack safe
def tailRecM[A, B](a: A)(f: A => F[Either[A, B]]): F[B]

def powWriter2(x: Long, exp: Long): Writer[LongProduct, Unit] =
  FlatMap[Writer[LongProduct, ?]].tailRecM(exp) {
    case 0L      => Writer.value[LongProduct, Either[Long, Unit]](Right(()))
    case m: Long => Writer.tell(LongProduct(x)) >>= { _ => Writer.value(Left(m - 1)) }
  }

# Monadic Functions
Functions that either operate on monadic values or return monadic values as their results (or both!). Such functions are usually referred to as monadic functions.

filterA, foldM
def binSmalls(acc: Int, x: Int): Option[Int] =
  if (x > 9) none[Int] else (acc + x).some
(Foldable[List].foldM(List(2, 11, 3, 1), 0) {binSmalls})

# Kleisli
final case class Kleisli[F[_], A, B](run: A => F[B])
def lift[G[_]](implicit G: Applicative[G]): Kleisli[λ[α => G[F[α]]], A, B] =
  Kleisli[λ[α => G[F[α]]], A, B](a => Applicative[G].pure(run(a)))

val f = Kleisli { (x: Int) => (x + 1).some }
scala> val l = f.lift[List]
l: cats.data.Kleisli[[α]List[Option[α]],Int,Int] = Kleisli(cats.data.Kleisli$$Lambda$3951/723364368@4f76e28c)
scala> List(1, 2, 3) >>= l.run
res2: List[Option[Int]] = List(Some(2), Some(3), Some(4))
(Int => Option[Int]) => (Int => List[Option[Int]])


Monads (technically FlatMap) is that they are fractals, like the above Sierpinski triangle. Each part of a fractal is self-similar to the whole shape.
When you want to find your own monad, keep a lookout for the fractal structure. From there, see if you can implement the flatten function F[F[A]] => F[A].


Kleisli === ReaderT
a bit mind-bending:
type ReaderTOption[A, B] = Kleisli[Option, A, B]
type StateTReaderTOption[C, S, A] =
StateT[ReaderTOption[C, ?], S, A]

A => Option[B]
S => (C => Option(S, A))

# Genericity
Genericity by value
def triangle4: Unit = {
  println("*")
  println("**")
  println("***")
  println("****")
}
def triangle(side: Int): Unit = {
  (1 to side) foreach { row =>
    (1 to row) foreach { col =>
      println("*")
    }
  }
}
Genericity by type
Genericity by function
Genericity by structure
The typeclass captures the requirements required of types, called typeclass contract. It also lets us list the types providing these requirements by defining typeclass instances.
Genericity by property
As well as signatures of operations, the concept might specify the laws these operations satisfy, and non-functional characteristics such as the asymptotic complexities of the operations in terms of time and space.
Typeclasses with laws fit in here too.
Genericity by stage
This could include code generation and macros.
Genericity by shape
sealed abstract class Fix[S[_], A] extends Serializable {
  def out: S[Fix[S, A]]
}
case class In[S[_], A](out: S[Fix[S, A]]) extends Fix[S, A]

Bifunctor
def bimap[A, B, C, D](fab: F[A, B])(f: A => C, g: B => D): F[C, D]
Implement map in terms of bimap
def map[F[_, _]: Bifunctor, A1, A2](fa: Fix[F[?, A1], A1])(f: A1 => A2): Fix[F[?, A2], A2]

We can implement fold, also known as cata from catamorphism:
def fold[F[_, _]: Bifunctor, A1, A2](fa: Fix[F[?, A1], A1])(f: F[A2, A1] => A2): A2 = {
  val g = (fa1: F[Fix[F[?, A1], A1], A1]) =>
  Bifunctor[F].leftMap(fa1) { (fold(_)(f)) }
  f(g(fa.out))
}
The unfold is also called ana from anamorphism:
def unfold[F[_, _]: Bifunctor, A1, A2](x: A2)(f: A2 => F[A2, A1]): Fix[F[?, A1], A1] =
  Fix.In[F[?, A1], A1](Bifunctor[F].leftMap(f(x))(unfold[F, A1, A2](_)(f)))
We can use these for the datatype being recursive.
sealed trait TreeF[+Next, +A]
object TreeF {
  case class EmptyF() extends TreeF[Nothing, Nothing]
  case class NodeF[Next, A](a: A, left: Next, right: Next) extends TreeF[Next, A]
}
scala> def grow[A: PartialOrder](xs: List[A]): Tree[A] =
          DGP.unfold[TreeF, A, List[A]](xs) {
            case Nil => TreeF.EmptyF()
            case x :: xs =>
              import cats.syntax.partialOrder._
              TreeF.NodeF(x, xs filter {_ <= x}, xs filter {_ > x})
          }
grow: [A](xs: List[A])(implicit evidence$1: cats.PartialOrder[A])Tree[A]
scala> grow(List(3, 1, 4, 2))
res8: Tree[Int] = In(NodeF(3,In(NodeF(1,In(EmptyF()),In(NodeF(2,In(EmptyF()),In(EmptyF()))))),In(NodeF(4,In(EmptyF()),In(EmptyF())))))

# Applicative functors
Monadic applicative functors
Naperian applicative functors
Monoidal applicative functors

A second family of applicative functors, this time non-monadic, arises from constant functors with monoidal targets.
Const
We can derive an applicative functor from any Monoid, by using empty for pure, and |+| for ap. The Const datatype is also called Const in Cats.
final case class Const[A, B](getConst: A) {
  def retag[C]: Const[A, C] =
    this.asInstanceOf[Const[A, C]]
}
When A forms a Semigroup, an Apply is derived, and when A form a Monoid, an Applicative is derived automatically.
scala> Const(2).retag[String => String] ap Const(1).retag[String]
res1: cats.data.Const[Int,String] = Const(3)

Product of functors
final case class Tuple2K[F[_], G[_], A](first: F[A], second: G[A]) {
  def mapK[H[_]](f: G ~> H): Tuple2K[F, H, A] =
    Tuple2K(first, f(second))
}
def map[A, B](fa: Tuple2K[F, G, A])(f: A => B): Tuple2K[F, G, B]
scala> val x = Tuple2K(List(1), 1.some)
x: cats.data.Tuple2K[List,Option,Int] = Tuple2K(List(1),Some(1))
scala> Functor[Lambda[X => Tuple2K[List, Option, X]]].map(x) { _ + 1 }
res0: cats.data.Tuple2K[[+A]List[A],Option,Int] = Tuple2K(List(2),Some(2))

Product of apply functors
scala> val x = Tuple2K(List(1), (Some(1): Option[Int]))
x: cats.data.Tuple2K[List,Option,Int] = Tuple2K(List(1),Some(1))
scala> val f = Tuple2K(List((_: Int) + 1), (Some((_: Int) * 3): Option[Int => Int]))
f: cats.data.Tuple2K[List,Option,Int => Int] = Tuple2K(List($$Lambda$4380/1035222735@235f2ef),Some($$Lambda$4381/1039907394@258b8fcc))
scala> Apply[Lambda[X => Tuple2K[List, Option, X]]].ap(f)(x)
res1: cats.data.Tuple2K[[+A]List[A],Option,Int] = Tuple2K(List(2),Some(3))

Product of applicative functors
scala> Applicative[Lambda[X => Tuple2K[List, Option, X]]].pure(1)
res2: cats.data.Tuple2K[[+A]List[A],Option,Int] = Tuple2K(List(1),Some(1))

Composition of Applicative
scala> Applicative[List].compose[Option].pure(1)
res3: List[Option[Int]] = List(Some(1))

Product of applicative functions
Kliesli composition will let you compose A => F[B] and B => F[C] using andThen, but note that F stays the same. On the other hand, AppFunc composes A => F[B] and B => G[C].
/**
 * [[Func]] is a function `A => F[B]`.
 */
sealed abstract class Func[F[_], A, B] { self =>
  def run: A => F[B]
  def map[C](f: B => C)(implicit FF: Functor[F]): Func[F, A, C] =
    Func.func(a => FF.map(self.run(a))(f))
}

sealed abstract class AppFunc[F[_], A, B] extends Func[F, A, B] { self =>
  def F: Applicative[F]

  def product[G[_]](g: AppFunc[G, A, B]): AppFunc[[α]Tuple2K[F, G, α], A, B]
}
scala> val f = Func.appFunc { x: Int => List(x.toString + "!") }
f: cats.data.AppFunc[List,Int,String] = cats.data.Func$$anon$6@a6b56dd
scala> val g = Func.appFunc { x: Int => (Some(x.toString + "?"): Option[String]) }
g: cats.data.AppFunc[Option,Int,String] = cats.data.Func$$anon$6@2d0f032f
scala> val h = f product g
h: cats.data.AppFunc[[α]cats.data.Tuple2K[List,Option,α],Int,String] = cats.data.Func$$anon$6@625084a0
scala> h.run(1)
res4: cats.data.Tuple2K[List,Option,String] = Tuple2K(List(1!),Some(1?))

# Traverse
@typeclass trait Traverse[F[_]] extends Functor[F] with Foldable[F] {
  def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): G[F[B]]
  def sequence[G[_]: Applicative, A](fga: F[G[A]]): G[F[A]] =
    traverse(fga)(ga => ga)
}
When m is specialised to the identity applicative functor, traversal reduces precisely (modulo the wrapper) to the functorial map over lists.
List(1, 2, 3) traverse[Id, Int] { (x: Int) => x + 1 }
In the case of a monadic applicative functor, traversal specialises to monadic map, and has the same uses. In fact, traversal is really just a slight generalisation of monadic map.

For a monoidal applicative functor, traversal accumulates values. The function reduce performs that accumulation, given an argument that assigns a value to each element
def reduce[A, B, F[_]](fa: F[A])(f: A => B)(implicit FF: Traverse[F],
                                            BB: Monoid[B]): B = {
  val x = fa traverse { (a: A) =>
    Const[B, Unit]((f(a)))
  }
  x.getConst
}
scala> reduce(List('a', 'b', 'c')) { c: Char => c.toInt }
res3: Int = 294

scala> def contents[F[_], A](fa: F[A])(implicit FF: Traverse[F]): Const[List[A], F[Unit]] =
         {
           val contentsBody: A => Const[List[A], Unit] = { (a: A) => Const(List(a)) }
           FF.traverse(fa)(contentsBody)
         }
contents: [F[_], A](fa: F[A])(implicit FF: cats.Traverse[F])cats.data.Const[List[A],F[Unit]]
scala> contents(Vector(1, 2, 3)).getConst
res0: List[Int] = List(1, 2, 3)
scala> def shape[F[_], A](fa: F[A])(implicit FF: Traverse[F]): Id[F[Unit]] =
         {
           val shapeBody: A => Id[Unit] = { (a: A) => () }
           FF.traverse(fa)(shapeBody)
         }
shape: [F[_], A](fa: F[A])(implicit FF: cats.Traverse[F])cats.Id[F[Unit]]
scala> shape(Vector(1, 2, 3))
res1: cats.Id[scala.collection.immutable.Vector[Unit]] = Vector((), (), ())
scala> def decompose[F[_], A](fa: F[A])(implicit FF: Traverse[F]) =
         Tuple2K[Const[List[A], ?], Id, F[Unit]](contents(fa), shape(fa))
decompose: [F[_], A](fa: F[A])(implicit FF: cats.Traverse[F])cats.data.Tuple2K[[β$0$]cats.data.Const[List[A],β$0$],cats.Id,F[Unit]]
scala> val d = decompose(Vector(1, 2, 3))
d: cats.data.Tuple2K[[β$0$]cats.data.Const[List[Int],β$0$],cats.Id,scala.collection.immutable.Vector[Unit]] = Tuple2K(Const(List(1, 2, 3)),Vector((), (), ()))
scala> d.first
res2: cats.data.Const[List[Int],scala.collection.immutable.Vector[Unit]] = Const(List(1, 2, 3))
scala> d.second
res3: cats.Id[scala.collection.immutable.Vector[Unit]] = Vector((), (), ())
Is it possible to fuse the two traversals into one? The product of applicative functors allows exactly this.
scala> def contentsBody[A]: AppFunc[Const[List[A], ?], A, Unit] =
         appFunc[Const[List[A], ?], A, Unit] { (a: A) => Const(List(a)) }
contentsBody: [A]=> cats.data.AppFunc[[β$0$]cats.data.Const[List[A],β$0$],A,Unit]
scala> def shapeBody[A]: AppFunc[Id, A, Unit] =
         appFunc { (a: A) => ((): Id[Unit]) }
shapeBody: [A]=> cats.data.AppFunc[cats.Id,A,Unit]
scala> def decompose[F[_], A](fa: F[A])(implicit FF: Traverse[F]) =
         (contentsBody[A] product shapeBody[A]).traverse(fa)

# Applicative wordcount
scala> type Count[A] = Const[Int, A]
defined type alias Count
scala> def liftInt(i: Int): Count[Unit] = Const(i)
liftInt: (i: Int)Count[Unit]
scala> def count[A](a: A): Count[Unit] = liftInt(1)
count: [A](a: A)Count[Unit]
scala> val countChar: AppFunc[Count, Char, Unit] = appFunc(count)
countChar: cats.data.AppFunc[Count,Char,Unit] = cats.data.Func$$anon$6@6aaf9ffb
scala> val text = ("Faith, I must leave thee, love, and shortly too.\n" +
                  "My operant powers their functions leave to do.\n").toList
text: List[Char] =
List(F, a, i, t, h, ,,  , I,  , m, u, s, t,  , l, e, a, v, e,  , t, h, e, e, ,,  , l, o, v, e, ,,  , a, n, d,  , s, h, o, r, t, l, y,  , t, o, o, .,
, M, y,  , o, p, e, r, a, n, t,  , p, o, w, e, r, s,  , t, h, e, i, r,  , f, u, n, c, t, i, o, n, s,  , l, e, a, v, e,  , t, o,  , d, o, .,
)
scala> countChar traverse text
res0: Count[List[Unit]] = Const(96)

Counting the lines (in fact, the newline characters, thereby ignoring a final ‘line’ that is not terminated with a newline character) is similar.
scala> def testIf(b: Boolean): Int = if (b) 1 else 0
testIf: (b: Boolean)Int
scala> val countLine: AppFunc[Count, Char, Unit] =
         appFunc { (c: Char) => liftInt(testIf(c === '\n')) }
countLine: cats.data.AppFunc[Count,Char,Unit] = cats.data.Func$$anon$6@12119968
scala> countLine traverse text
res1: Count[List[Unit]] = Const(2)

scala> def isSpace(c: Char): Boolean = (c === ' ' || c === '\n' || c === '\t')
isSpace: (c: Char)Boolean
scala> val countWord =
         appFunc { (c: Char) =>
           import cats.data.State.{ get, set }
           for {
             x <- get[Boolean]
             y = !isSpace(c)
             _ <- set(y)
           } yield testIf(y && !x)
         } andThen appFunc(liftInt)
countWord: cats.data.AppFunc[[γ$3$]cats.data.Nested[[A]cats.data.IndexedStateT[cats.Eval,Boolean,Boolean,A],Count,γ$3$],Char,Unit] = cats.data.Func$$anon$6@6b5ac45e
scala> val x = countWord traverse text
x: cats.data.Nested[[A]cats.data.IndexedStateT[cats.Eval,Boolean,Boolean,A],Count,List[Unit]] = Nested(cats.data.IndexedStateT@69f1c34d)
scala> x.value.runA(false).value
res2: Count[List[Unit]] = Const(17)

scala> val countAll = countWord product countLine product countChar
countAll: cats.data.AppFunc[[α]cats.data.Tuple2K[[α]cats.data.Tuple2K[[γ$3$]cats.data.Nested[[A]cats.data.IndexedStateT[cats.Eval,Boolean,Boolean,A],Count,γ$3$],Count,α],Count,α],Char,Unit] = cats.data.Func$$anon$6@38dc0c99
scala> val allResults = countAll traverse text
allResults: cats.data.Tuple2K[[α]cats.data.Tuple2K[[γ$3$]cats.data.Nested[[A]cats.data.IndexedStateT[cats.Eval,Boolean,Boolean,A],Count,γ$3$],Count,α],Count,List[Unit]] = Tuple2K(Tuple2K(Nested(cats.data.IndexedStateT@46be96e4),Const(2)),Const(96))
scala> val charCount = allResults.second
charCount: Count[List[Unit]] = Const(96)
scala> val lineCount = allResults.first.second
lineCount: Count[List[Unit]] = Const(2)
scala> val wordCountState = allResults.first.first
wordCountState: cats.data.Nested[[A]cats.data.IndexedStateT[cats.Eval,Boolean,Boolean,A],Count,List[Unit]] = Nested(cats.data.IndexedStateT@46be96e4)
scala> val wordCount = wordCountState.value.runA(false).value
wordCount: Count[List[Unit]] = Const(17)

Applicative functors have a richer algebra of composition operators, which can often replace the use of monad transformers; there is the added advantage of being able to compose applicative but non-monadic computations.

# TraverseEmpty
final class EmptyOps[F[_], A](val fa: F[A]) extends AnyVal {
  def traverseFilter[G[_] : Applicative, B](f: A => G[Option[B]])(implicit F: TraverseEmpty[F]): G[F[B]] = F.traverseFilter(fa)(f)
  def filterA[G[_]: Applicative](f: A => G[Boolean])(implicit F: TraverseEmpty[F]): G[F[A]] = F.filterA(fa)(f)
  def mapFilter[B](f: A => Option[B])(implicit F: FunctorEmpty[F]): F[B] = F.mapFilter(fa)(f)
  def collect[B](f: PartialFunction[A, B])(implicit F: FunctorEmpty[F]): F[B] = F.collect(fa)(f)
  def filter(f: A => Boolean)(implicit F: FunctorEmpty[F]): F[A] = F.filter(fa)(f)
}
filterA is a more generalized (or weaker) version of filterM. Instead of requiring a Monad[G] it needs Applicative[G].
scala> List(1, 2, 3) filterA { x => List(true, false) }
res7: List[List[Int]] = List(List(1, 2, 3), List(1, 2), List(1, 3), List(1), List(2, 3), List(2), List(3), List())

# Eval
sealed abstract class Eval[+A] extends Serializable { self =>
  def value: A
  def memoize: Eval[A]
}
object Eval extends EvalInstances {
  def now[A](a: A): Eval[A] = Now(a)
  def later[A](a: => A): Eval[A] = new Later(a _)
  def always[A](a: => A): Eval[A] = new Always(a _)
  def defer[A](a: => Eval[A]): Eval[A] = new Eval.Call[A](a _) {}

  val Unit: Eval[Unit] = Now(())
  val True: Eval[Boolean] = Now(true)
  val False: Eval[Boolean] = Now(false)
  val Zero: Eval[Int] = Now(0)
  val One: Eval[Int] = Now(1)
}

def foldRight[A, B](fa: List[A], lb: Eval[B])(f: (A, Eval[B]) => Eval[B]): Eval[B] = {
  def loop(as: List[A]): Eval[B] =
    as match {
      case Nil => lb
      case h :: t => f(h, Eval.defer(loop(t)))
    }
  Eval.defer(loop(fa))
}
object OddEven1 {
  def odd(n: Int): Eval[String] = Eval.defer {even(n - 1)}
  def even(n: Int): Eval[String] =
    Eval.now { n <= 0 } flatMap {
      case true => Eval.now {"done"}
      case _    => Eval.defer { odd(n - 1) }
    }
}


Use Id[_] for F[_]

# SemigroupK
combineK (<+>)

Option[A] can form a Semigroup only when the type parameter A forms a Semigroup.
scala> Foo("x").some |+| Foo("y").some
<console>:33: error: value |+| is not a member of Option[Foo]
       Foo("x").some |+| Foo("y").some
The Semigroup will combine the inner value of the Option whereas SemigroupK will just pick the first one.
scala> 1.some |+| 2.some
res4: Option[Int] = Some(3)
scala> 1.some <+> 2.some
res5: Option[Int] = Some(1)

law:
F.combineK(F.combineK(a, b), c) <-> F.combineK(a, F.combineK(b, c))

# MonoidK
@typeclass trait MonoidK[F[_]] extends SemigroupK[F] { self =>
  def empty[A]: F[A]
}
laws:
def monoidKLeftIdentity[A](a: F[A]): IsEq[F[A]] =
  F.combineK(F.empty, a) <-> a
def monoidKRightIdentity[A](a: F[A]): IsEq[F[A]] =
  F.combineK(a, F.empty) <-> a

# Alternative
@typeclass trait Alternative[F[_]] extends Applicative[F] with MonoidK[F]
Alternative on its own does not introduce any new methods or operators.
It’s more of a weaker (thus better) Applicative version of MonadPlus that adds filter on top of Monad.
law:
def alternativeRightAbsorption[A, B](ff: F[A => B]): IsEq[F[B]] =
  (ff ap F.empty[A]) <-> F.empty[B]
def alternativeLeftDistributivity[A, B](fa: F[A], fa2: F[A], f: A => B): IsEq[F[B]] =
  ((fa |+| fa2) map f) <-> ((fa map f) |+| (fa2 map f))
def alternativeRightDistributivity[A, B](fa: F[A], ff: F[A => B], fg: F[A => B]): IsEq[F[B]] =
  ((ff |+| fg) ap fa) <-> ((ff ap fa) |+| (fg ap fa))
There’s an open question by Yoshida-san on whether the last law is necessary or not.

# Basic category theory
A arrow f in this cateogry consists of three things:
a set A, called the domain of the arrow,
a set B, called the codomain of the arrow,
a rule assigning to each element a in the domain, an element b in the codomain. This b is denoted by f ∘ a (or sometimes ’f(a)‘), read ’f of a‘.
I get that a map can be more general than Function1[A, B] but it’s ok for this category.

An arrow in which the domain and codomain are the same object is called an endomorphism.
An arrow, in which the domain and codomain are the same set A, and for each of a in A, f(a) = a, is called an identity arrow.

Data for a category consists of the four ingredients:
objects: A, B, C, …
arrows: f: A => B
identity arrows: 1A: A => A
composition of arrows
The identity laws:
If 1A: A => A, g: A => B, then g ∘ 1A = g
If f: A => B, 1B: B => B, then 1A ∘ f = f
The associative law:
If f: A => B, g: B => C, h: C => D, then h ∘ (g ∘ f) = (h ∘ g) ∘ f

# Arrow
As we saw, an arrow (or morphism) is a mapping between a domain and a codomain. Another way of thinking about it, is that its an abstract notion for things that behave like functions.
In Cats, an Arrow instance is provided for Function1[A, B], Kleisli[F[_], A, B], and Cokleisli[F[_], A, B].
@typeclass trait Arrow[F[_, _]] extends Split[F] with Strong[F] with Category[F] { self =>
  def lift[A, B](f: A => B): F[A, B]
}
@typeclass trait Category[F[_, _]] extends Compose[F] { self =>
  def id[A]: F[A, A]
}
@typeclass trait Compose[F[_, _]] { self =>
  @simulacrum.op("<<<", alias = true)
  def compose[A, B, C](f: F[B, C], g: F[A, B]): F[A, C]
  @simulacrum.op(">>>", alias = true)
  def andThen[A, B, C](f: F[A, B], g: F[B, C]): F[A, C]
}
(f >>> g)(2)
(f <<< g)(2)
@typeclass trait Strong[F[_, _]] extends Profunctor[F] {
  def first[A, B, C](fa: F[A, B]): F[(A, C), (B, C)]
  def second[A, B, C](fa: F[A, B]): F[(C, A), (C, B)]
}
scala> val f_first = f.first[Int]
f_first: ((Int, Int)) => (Int, Int) = cats.instances.Function1Instances$$anon$3$$Lambda$4931/1847944899@6e03b360
scala> f_first((1, 1))
res2: (Int, Int) = (2,1)
scala> val f_second = f.second[Int]
f_second: ((Int, Int)) => (Int, Int) = scala.Function1$$Lambda$2547/1273946735@2aa149a9
scala> f_second((1, 1))
res3: (Int, Int) = (1,2)
@typeclass trait Split[F[_, _]] extends Compose[F] { self =>
  def split[A, B, C, D](f: F[A, B], g: F[C, D]): F[(A, C), (B, D)]
}
scala> (f split g)((1, 1))
res4: (Int, Int) = (2,100)

Isomorphism
Definitions: An arrow f: A => B is called an isomorphism, or invertible arrow, if there is a map g: B => A, for which g ∘ f = 1A and f ∘ g = 1B. An arrow g related to f by satisfying these equations is called an inverse for f. Two objects A and B are said to be isomorphic if there is at least one isomorphism f: A => B.

Another kind of example one often sees in mathematics is categories of structured sets, that is, sets with some further “structure” and functions that “preserve it,” where these notions are determined in some independent way.

A partially ordered set or poset is a set A equipped with a binary relation a ≤A b such that the following conditions hold for all a, b, c ∈ A:

reflexivity: a ≤A a
transitivity: if a ≤A b and b ≤A c, then a ≤A c
antisymmetry: if a ≤A b and b ≤A a, then a = b
An arrow from a poset A to a poset B is a function m: A => B that is monotone, in the sense that, for all a, a’ ∈ A,

a ≤A a’ implies m(a) ≤A m(a’).
As long as the functions are monotone, the objects will continue to be in the category, so the “structure” is preserved.

Functor
Definition 1.2. A functor
F: C => D
between categories C and D is a mapping of objects to objects and arrows to arrows, in such a way that.

F(f: A => B) = F(f): F(A) => F(B)
F(1A) = 1F(A)
F(g ∘ f) = F(g) ∘ F(f)

Monoid as categories
A monoid is a category with just one object. The arrows of the category are the elements of the monoid. In particular, the identity arrow is the unit element u. Composition of arrows is the binary operation m · n for the monoid.

Mon
In detail, a homomorphism from a monoid M to a monoid N is a function h: M => N such that for all m, n ∈ M,

h(m ·M n) = h(m) ·N h(n)
h(uM) = uN
Since a monoid is a category, a monoid homomorphism is a special case of functors.

Grp
Definition 1.4 A group G is a monoid with an inverse g-1 for every element g. Thus, G is a category with one object, in which every arrow is an isomorphism.
trait Group[@sp(Int, Long, Float, Double) A] extends Any with Monoid[A] {
  def inverse(a: A): A
}
assert((1 |+| 1.inverse) === Monoid[Int].empty)

Forgetful functor
We’ve seen the term homomorphism a few times, but it’s possible to think of a function that doesn’t preserve the structure. Because every group G is also a monoid we can think of a function f: G => M where f loses the inverse ability from G and returns underlying monoid as M. Since both groups and monoids are categories, f is a functor.
We can extend this to the entire Grp, and think of a functor F: Grp => Mon. These kinds of functors that strips the structure is called forgetful functors. If we try to express this using Scala, you would start with A: Group, and somehow downgrade to A: Monoid as the ruturn value.

# Initial and terminal objects
When a definition relies only on category theoretical notion (objects and arrows), it often reduces down to a form “given a diagram abc, there exists a unique x that makes another diagram xyz commute.” Commutative in this case mean that all the arrows compose correctly. Such defenition is called a universal property or a universal mapping property (UMP).
Some of the notions have a counterpart in set theory, but it’s more powerful because of its abtract nature. Consider making the empty set and the one-element sets in Sets abstract.

Definition 2.9. In any category C, an object
0 is initial if for any object C there is a unique morphism
1 is terminal if for any object C there is a unique morphism

Uniquely determined up to isomorphism
As a general rule, the uniqueness requirements of universal mapping properties are required only up to isomorphisms. Another way of looking at it is that if objects A and B are isomorphic to each other, they are “equal in some sense.” To signify this, we write A ≅ B.
Proposition 2.10 Initial (terminal) objects are unique up to isomorphism.
Proof. In fact, if C and C’ are both initial (terminal) in the same category, then there’s a unique isomorphism C => C’. Indeed, suppose that 0 and 0’ are both initial objects in some category C; the following diagram then makes it clear that 0 and 0’ are uniquely isomorphic.
An interest aspect of abstract construction is that they can show up in different categories in different forms.
In Sets, the empty set is initial and any singleton set {x} is terminal.
scala> def absurd[A]: Nothing => A = { case _ => ??? }
absurd: [A]=> Nothing => A
In a poset, an object is plainly initial iff it is the least element, and terminal iff it is the greatest element.
This kind of makes sense, since in a poset we need to preserve the structure using ≤.

scala> def unit[A](a: A): Unit = ()
unit: [A](a: A)Unit
This makes Unit a terminal object in the category of Sets, but note that we can define singleton types all we want in Scala using object.

Product
Given sets A and B, the cartesian product of A and B is the set of ordered pairs
A × B = {(a, b)| a ∈ A, b ∈ B}
fst ∘ (a, b) = a
snd ∘ (a, b) = b
Definition 2.15. In any category C, a product diagram for the objects A and B consists of an object P and arrows p1 and p2 satisfying the following UMP:
Given any diagram of the form
there exists a unique u: X => P, making the diagram commute, that is, such that x1 = p1 u and x2 = p2 u.
“There exists unique u” is a giveaway that this definition is an UMP.
Proposition 2.17 Products are unique up to isomorphism.

Duality
The opposite (or “dual”) category Cop of a category C has the same objects as C, and an arrow f: C => D in Cop is an arrow f: D => C in C. That is, Cop is just C with all of the arrows formally turned around.
If we take the concept further, we can come up with “dual statement” Σ* by substituting any sentence Σ in the category theory by replacing the following:
f ∘ g for g ∘ f
codomain for domain
domain for codomain
Since there’s nothing semantically important about which side is f or g, the dual statement also holds true as long as Σ only relies on category theory. In other words, any proof that holds for one concept also holds for its dual. This is called the duality principle.
Another way of looking at it is that if Σ holds in all C, it should also hold in Cop, and so Σ* should hold in (Cop)op, which is C.
Recall proof for “the initial objects are unique up to isomorphism.”
If you flip the direction of all arrows in the above diagram, you do get a proof for terminals.

Coproduct
One of the well known duals is coproduct, which is the dual of product. Prefixing with “co-” is the convention to name duals.
Since coproducts are unique up to isomorphism, we can denote the coproduct as A + B, and [f, g] for the arrow u: A + B => X.
The “coprojections” i1: A => A + B and i2: B => A + B are usually called injections, even though they need not be “injective” in any sense.
Similar to the way products related to product type encoded as scala.Product, coproducts relate to the notion of sum type, or disjoint union type.

Algebraic datatype
To encode A + B might be using sealed trait and case classes.
Either datatype as coproduct

scala> type |:[+A1, +A2] = Either[A1, A2]
defined type alias $bar$colon
Cats provides a typeclass called cats.Inject that represents injections i1: A => A + B and i2: B => A + B.
scala> val a = Inject[String, String |: Int].inj("a")
a: String |: Int = Left(a)
scala> val one = Inject[Int, String |: Int].inj(1)
one: String |: Int = Right(1)
scala> Inject[String, String |: Int].prj(a)
res6: Option[String] = Some(a)
scala> Inject[String, String |: Int].prj(one)
res7: Option[String] = None
scala> val StringInj = Inject[String, String |: Int]
StringInj: cats.Inject[String,String |: Int] = cats.InjectInstances$$anon$2@3e67ae9
scala> val IntInj = Inject[Int, String |: Int]
IntInj: cats.Inject[Int,String |: Int] = cats.InjectInstances$$anon$3@6a9628bb
scala> val b = StringInj("b")
b: String |: Int = Left(b)
scala> val two = IntInj(2)
two: String |: Int = Right(2)
scala> two match {
         case StringInj(x) => x
         case IntInj(x)    => x.show + "!"
       }
res8: String = 2!
The reason I put colon in |: is to make it right-associative. This matters when you expand to three types.

There’s a datatype in Cats called EitherK[F[_], G[_], A], which is an either on type constructor.
