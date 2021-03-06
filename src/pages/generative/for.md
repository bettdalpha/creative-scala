## For Comprehensions

```tut:invisible
import doodle.core._
import doodle.core.Image._
import doodle.syntax._
import doodle.jvm.Java2DFrame._
import doodle.backend.StandardInterpreter._
```

<div class="callout callout-info">
In addition to the standard imports given at the start of the chapter, in this section we're assuming the following:

```tut:silent
import doodle.random._
```
</div>

Scala provides some special syntax, called a *for comprehension*, that makes it simpler to write long sequences of `flatMap` and `map`.

For example, the code for `randomConcentricCircles` has a call to `flatMap` and `map`.

```tut:silent:invisible
def randomAngle: Random[Angle] =
  Random.double.map(x => x.turns)

def randomColor(s: Normalized, l: Normalized): Random[Color] =
  randomAngle map (hue => Color.hsl(hue, s, l))

val randomPastel = randomColor(0.7.normalized, 0.7.normalized)

def randomCircle(r: Double, color: Random[Color]): Random[Image] =
  color map (fill => Image.circle(r) fillColor fill)
```

```tut:silent:book
def randomConcentricCircles(count: Int, size: Int): Random[Image] =
  count match {
    case 0 => Random.always(Image.empty)
    case n => 
      randomCircle(size, randomPastel) flatMap { circle =>
        randomConcentricCircles(n-1, size + 5) map { circles =>
          circle on circles
        }
      }
  }
```

This can be replaced with a for comprehension.

```tut:silent:book
def randomConcentricCircles(count: Int, size: Int): Random[Image] =
  count match {
    case 0 => Random.always(Image.empty)
    case n => 
      for {
        circle  <- randomCircle(size, randomPastel) 
        circles <- randomConcentricCircles(n-1, size + 5)
      } yield circle on circles 
  }
```

The for comprehension is often easier to read than direct use of `flatMap` and `map`.

A general for comprehension

```tut:book:invisible
val a: Seq[Int] = Seq.empty
val b: Seq[Int] = Seq.empty
val c: Seq[Int] = Seq.empty
val e: Int = 0
```

```tut:book:silent
for {
  x <- a
  y <- b
  z <- c
} yield e
```

translates to:

```tut:book:silent
a.flatMap(x => b.flatMap(y => c.map(z => e)))
```

Which is to say that every `<-`, except the last, turns into a `flatMap`, and the last `<-` becomes a `map`.

For comprehensions are translated by the compiler into uses of `flatMap` and `map`.
There is no magic going on. 
It is just a different way of writing code that would use `flatMap` and `map` that avoids excessive nesting.

Note that the for comprehension syntax is more flexible than what we have presented here.
For example, you can drop the `yield` keyword from a for comprehension and the code will still compile.
It just won't return a result.
We're not going to use any of these extensions in Creative Scala, however.
