If you would like to _update_ tuple values in Scala, like this:

```scala
var tuple = (1, "test")
tuple._2 = "new"
```

It would not be possible. Scala will not compile this kind of code.

The reason is that you cannot reassign tuple values.  They're intentionally immutable: once you have created a tuple. This is very useful for writing correct code.

But what if you want a different tuple?  That's where the `copy` method comes in:

```scala
val tuple = (1, "test")
val another = tuple.copy(_2 = "new")
```

or if you really want to use a `var` to contain the tuple:

```scala
var tuple = (1, "test")
tuple = tuple.copy(_2 = "new")
```

As an alternative, if you really, really want your values to change individually, you can use a `case class` instead (probably with an `implicit` conversion so you can get a tuple when you need it) :

```scala
case class Doublet[A,B](var _1: A, var _2: B) {}
implicit def doublet_to_tuple[A,B](db: Doublet[A,B]) = (db._1, db._2)
val doublet = Doublet(1, "test")
doublet._2 = "new"
```

Nevertheless, I do not recommend this kind of code :smile:. It usually means you have poor design and should think twice how to achieve what you want without modyfying tuples. 

See [stackoverflow](http://stackoverflow.com/questions/7142838/in-scala-how-can-i-reassign-tuple-values) for explanation.