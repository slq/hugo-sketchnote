+++
title = "Suprising casts"
description = "So you are geek. Do you know how casts work in Java?"
date = 2017-02-03T23:45:50Z
author = "Slawek Plamowski"
+++

```java
int amount = 0;
amount += 1.5;
```

I expected javac to complain about calling `amount += 1.5;` without an explicit cast. Most of us treat this as `amount = amount + 1.5;` However, it is rather `amount = (int)(amount + 1.5);` That's right, `javac` adds an explicit cast when we use a compound operation in Java. It is described in the Java Language Specification section 15.26.2 "Compound Assignment Operators":

>A compound assignment expression of the form E1 op= E2 is equivalent to E1 = (T)((E1) op (E2)), where T is the type of E1, except that E1 is evaluated only once.

Consider the following examples.

```java
public class CastSurprise {
  public static void main(String... args) {
    // case 1:
    char c1 = 'a' + 5;  // compiles

    // case 2:
    char c2 = 'a';
    c2 = c2 + 5; // does not compile

    // case 3:
    char c3 = 'a';
    c3 += 5; // compiles
  }
}
```

In case 1, javac could determine at static compile time that the value would not lose precision. If instead of 5 we add 500_000, then javac would complain, because we would overflow the 16 bit char type.

In case 2, since we are increasing c2 without an explicit cast, javac cannot guarantee that the result would fit inside 16 bits. It thus forces us to add a cast.

In case 3, the cast to char is added by the compiler. As we saw above, the statement c3 += 5; is equivalent to c3 = (char)(c3 + 5);

### Left-Hand Operands

You can find the definition in `JLS 15.7.1`. Try figure it out before running them:

```java
public class Test1 { // JLS 15.7.1
  public static void main(String... args) {
    int i = 2;
    int j = (i = 3) * i;
    System.out.println(j);
  }
}
```

**Left-Hand Operand Is Evaluated First**

>In the example, the `*` operator has a left-hand operand that contains an assignment to a variable and a right-hand operand that contains a reference to the same variable. The value produced by the reference will reflect the fact that the assignment occurred first.

**OUTPUT**
```
9
```

And now for the second one:

```java
public class Test2 {// JLS 15.7.1
  public static void main(String... args) {
    int a = 9;
    a += (a = 3); // first example
    System.out.println(a);
    int b = 9;
    b = b + (b = 3); // second example
    System.out.println(b);
  }
}
```

>In the example, the two assignment statements both fetch and remember the value of the left-hand operand, which is 9, before the right-hand operand of the addition operator is evaluated, at which point the variable is set to 3.

**OUTPUT**
```
12
12
```

And our last one:

```java
public class Test3 {// JLS 15.7.1
  public static void main(String... args) {
    int j = 1;
    try {
      int i = forgetIt() / (j = 2);
    } catch (Exception e) {
      System.out.println("Now j = " + j);
    }
  }

  static int forgetIt() throws Exception {
    throw new Exception("I'm outta here!");
  }
}
```

>That is, the left-hand operand forgetIt() of the operator / throws an exception before the right-hand operand is evaluated and its embedded assignment of 2 to j occurs.

**OUTPUT**
```
java.lang.Exception: I'm outta here!
Now j = 1
```


