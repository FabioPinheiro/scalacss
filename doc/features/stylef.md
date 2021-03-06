## Functional Styles

Sometimes you want a style that depends on input.

For standalone stylesheets, it's not an issue.
```scala
object MyStyles extends StyleSheet.Standalone {
  import dsl._

  for (i <- 0 to 3)
    s".indent-$i" -
      paddingLeft(i * 2.ex)
}
```

For all other purposes, you want an `A => Style`.
That's what `StyleF` is.

A `StyleF[A]` has two arguments:

1. `A => StyleS`. A function producing a static style.
2. `Domain[A]`. A set of all legal inputs.
  CSS is generated ahead-of-time, before styles are used so its inputs must also be provided ahead-of-time.

Domains can be generated as follows:
```scala
Domain.boolean                                   // Domain[Boolean]
Domain.ofValues(1, 5, 7, 9)                      // Domain[Int]
Domain.ofRange(0 to 4)                           // Domain[Int]
Domain.ofRange(0 to 4).option *** Domain.boolean // Domain[(Option[Int], Boolean)]
```

##### Syntax
```scala
// Styles for each A
styleF(Domain[A])(a => styleS(…))

// Convenience methods for boolean & integer ranges
styleF.bool      (b => styleS(…))
styleF.int(range)(i => styleS(…))

// Manual classname
styleF("manual")(Domain[A])(a => styleS(…))
styleF("manual").bool      (b => styleS(…))
styleF("manual").int(range)(i => styleS(…))
```

##### Example

```scala
object MyInline extends StyleSheet.Inline {
  import dsl._

  // Convenience method: styleF.bool
  val everythingOk =
    styleF.bool(ok => styleS(
      backgroundColor(if (ok) green else red)
    ))

  // Convenience method: styleF.int
  val indent =
    styleF.int(1 to 3)(i => styleS(
      paddingLeft(i * 4.ex)
    ))

  // Full control
  sealed trait Blah
  case object Blah1 extends Blah
  case object Blah2 extends Blah
  case object Blah3 extends Blah
  val blahDomain = Domain.ofValues(Blah1, Blah2, Blah3)
  val blahStyle =
    styleF(blahDomain)(b => styleS(
      ...
    ))
}

```

## Universal Equality

Inputs to a `styleF` are stored in a standard Scala `Map` so they require sensible hashcodes and universal equality.

This won't work:

```scala
class MyBlob(val blobIsHappy: Boolean)

val domain = Domain.ofValues(new Blob(true), new Blob(false))

styleF("blobs")(domain)(blob => styleS(…))
```

In order to make the above example work, we:
* Provide working hashcode & universal equality. Easiest is to make `MyBlob` a `case class`.
* Provide an implicit `UnivEq[MyBlob]` instance. This tells the compiler that `MyBlob` is safe to be stored in a `Map`.

```scala
import japgolly.univeq._

case class MyBlob(blobIsHappy: Boolean)
object MyBlob {
  implicit def univEq: UnivEq[MyBlob] = UnivEq.derive
}

val domain = Domain.ofValues(Blob(true), Blob(false))

styleF("blobs")(domain)(blob => styleS(…))
```
