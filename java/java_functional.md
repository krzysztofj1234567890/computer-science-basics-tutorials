# Java Functional Programming

Basically, functional programming is a style of writing computer programs that treat computations as evaluating mathematical functions.

In mathematics, a function is an expression that relates an input set to an output set.

Importantly, the output of a function depends only on its input. More interestingly, we can compose two or more functions together to get a new function.

A programming language is said to have first-class functions if it treats functions as first-class citizens. This means that functions are allowed to support all operations typically available to other entities. 

Higher-order functions are capable of receiving functions as arguments and returning a function as a result.

Example (https://www.baeldung.com/java-functional-programming)

Instead of using this:
```
Collections.sort(numbers, new Comparator<Integer>() {
    @Override
    public int compare(Integer n1, Integer n2) {
        return n1.compareTo(n2);
    }
});
```
Use this:
```
Collections.sort(numbers, (n1, n2) -> n1.compareTo(n2));
```

Behind the syntactic sugar of lambda expressions, Java still wraps these into functional interfaces. So, Java treats a lambda expression as an Object, which is the true first-class citizen in Java.

## Immutability

Immutability is one of the core principles of functional programming, and it refers to the property that an entity can’t be modified after being instantiated.

rules:
- All fields of an immutable data structure must be immutable.
- This must apply to all the nested types and collections (including what they contain) as well.
- There should be one or more constructors for initialization as needed.
- There should only be accessor methods, possibly with no side effects.

## Referential Transparency

We call an expression referentially transparent if replacing it with its corresponding value has no impact on the program’s behavior.

## Functional Programming Techniques

### Function Composition

Function composition refers to composing complex functions by combining simpler functions.

```
Function<Double, Double> log = (value) -> Math.log(value);
Function<Double, Double> sqrt = (value) -> Math.sqrt(value);
Function<Double, Double> logThenSqrt = sqrt.compose(log);
logger.log(Level.INFO, String.valueOf(logThenSqrt.apply(3.14)));
// Output: 1.06
Function<Double, Double> sqrtThenLog = sqrt.andThen(log);
logger.log(Level.INFO, String.valueOf(sqrtThenLog.apply(3.14)));
// Output: 0.57
```

### Monads

Monad is an abstraction that allows structuring programs generically. So, a monad allows us to wrap a value, apply a set of transformations, and get the value back with all transformations applied.

There are three laws that any monad needs to follow — left identity, right identity and associativity

In Java, there are a few monads that we use quite often, such as __Optional__ and __Stream__:
```
Optional.of(2).flatMap(f -> Optional.of(3).flatMap(s -> Optional.of(f + s)))
```
Why do we call Optional a monad?

Optional allows us to wrap a value using the method of and apply a series of transformations. We’re applying the transformation of adding another wrapped value using the method flatMap.

### Currying

Currying is a mathematical technique of converting a function that takes multiple arguments into a sequence of functions that take a single argument.

In functional programming, it gives us a powerful composition technique where we don’t need to call a function with all its arguments.

Moreover, a curried function does not realize its effect until it receives all the arguments.

```
Function<Double, Function<Double, Double>> weight = gravity -> mass -> mass * gravity;

Function<Double, Double> weightOnEarth = weight.apply(9.81);
logger.log(Level.INFO, "My weight on Earth: " + weightOnEarth.apply(60.0));

Function<Double, Double> weightOnMars = weight.apply(3.75);
logger.log(Level.INFO, "My weight on Mars: " + weightOnMars.apply(60.0));
```

### Recursion

Recursion is another powerful technique in functional programming that allows us to break down a problem into smaller pieces. 

Instead of this:
```
Integer factorial(Integer number) {
    return (number == 1) ? 1 : number * factorial(number - 1);
}
```
Notice that we’re making the recursive call before calculating the result at each step or in words at the head of the calculation. 
So, this style of recursion is also known as __head recursion__.

A drawback of this type of recursion is that every step has to hold the state of all previous steps until we reach the base case. This is not really a problem for small numbers, but holding the state for large numbers can be inefficient.

A solution is a slightly different implementation of the recursion known as __tail recursion__. 
Here we ensure that the recursive call is the last call a function makes.

do this:
```
Integer factorial(Integer number, Integer result) {
    return (number == 1) ? result : factorial(number - 1, result * number);
}
```


## Lambda

```
parameter -> expression
OR
(parameter1, parameter2) -> expression
OR
(parameter1, parameter2) -> { code block }
```

### Using Lambda Expressions

#### Runnable
Instead of:
```
public class GFGRun implements Runnable { 
    public void run() { 
        System.out.println( "Running in Runnable thread");
    } 
} 
public class GFG {
    public static void main(String[] args) { 
        new Thread(new GFGRun()).start(); 
    } 
}
```
Use anonymous function (lambda):
```
public class GFG {
    public static void main(String[] args) {
        Runnable r = new Runnable() {
            public void run() {
                System.out.println( "Running in Runnable thread");
            }
        };
 
        r.run();
        System.out.println( "Running in main thread");
    }
}
```
Or better do this:
```
public class GFG {
    public static void main(String[] args) {
        Runnable r  = ()-> System.out.println( "Running in Runnable thread");
 
        r.run();
 
        System.out.println( "Running in main thread");
    }
}
```

#### Iterator

Instead this:
```
public class GFG {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(11, 22, 33, 44, 55, 66, 77, 88, 99, 100);
 
        // External iterator, for Each loop
        for (Integer n : numbers) {
            System.out.print(n + " ");
        }
    }
}
```

Do this:
```
public class GFG {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(11, 22, 33, 44, 55, 66, 77, 88, 99, 100);
 
        // Internal iterator
        numbers.forEach(number -> System.out.print( number + " "));
    }
}
```

And better:
```
public class GFG {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(11, 22, 33, 44, 55, 66, 77, 88, 99, 100);
 
        System.out.println( numbers.stream()
                .filter(number -> number % 2 == 0)
                .mapToInt(e -> e * 2)
                .sum());
    }
}
```

From the above code, we are not mutating any variable. Instead, we are transforming the data from one function to another. This is another difference between Imperative and Declarative. Not only this but also in the above code of declarative style, every function is a pure function and pure functions don’t have side effects

# Java Streams

https://www.baeldung.com/java-8-streams

Stream API is used to process collections of objects. A stream in Java is a sequence of objects that supports various methods that can be pipelined to produce the desired result.

Streams in Java make data processing more efficient by supporting functional-style operations.

Features:
- A stream is not a data structure instead it takes input from the Collections, Arrays or I/O channels.
- Streams don’t change the original data structure, they only provide the result as per the pipelined methods.
- Each intermediate operation is lazily executed and returns a stream as a result, hence various intermediate operations can be pipelined. Terminal operations mark the end of the stream and return the result.

There are two types of Operations in Streams:
- Intermediate Operations
- Terminate Operations

Benefits:
- No Storage
- Pipeline of Functions
- Laziness
- Can be infinite
- Can be parallelized
- Can be created from collections, arrays, Files Lines, Methods in Stream, IntStream etc.

Intermediate Operations:
- map(): The map method is used to return a stream consisting of the results of applying the given function to the elements of this stream.
- filter(): The filter method is used to select elements as per the Predicate passed as an argument.
- sorted(): The sorted method is used to sort the stream.
- flatMap(): The flatMap operation in Java Streams is used to flatten a stream of collections into a single stream of elements. 
- distinct (): Removes duplicate elements.

Terminal Operations:
- collect(): The collect method is used to return the result of the intermediate operations performed on the stream.
- forEach(): The forEach method is used to iterate through every element of the stream.
- reduce(): The reduce method is used to reduce the elements of a stream to a single value
- count(): Returns the count of elements in the stream.