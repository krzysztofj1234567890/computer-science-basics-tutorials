# Java features anfter java 8


## Collection Factory Methods

Allow you to create immutable or mutable collections in a simple, concise, and often more readable manner.
- List.of(): Creates an immutable List
- Set.of(): Creates an immutable Set
- Map.of(): Creates an immutable Map

## var - Local Variable Type Inference

Allows you to declare a local variable without explicitly specifying its type
- type of the variable is automatically inferred. You cannot use var without initializing the variable because the type must be deduce


## Switch Expressions

## Records and Record Patterns

Provide a compact syntax for creating classes whose primary purpose is to hold data. 
The main idea behind records is to provide a lightweight data structure for encapsulating immutable data without the need to manually write boilerplate code (such as constructors, equals(), hashCode(), and toString()).

```
public record Person(String name, int age) {
    public String greet() {
        return "Hello, my name is " + name;
    }
}
```

The record Person automatically defines:
- Final fields (name and age).
- A canonical constructor to initialize those fields.
- Getters for each field (name() and age() methods).
- equals(), hashCode(), and toString() methods automatically.

No Boilerplate Code: Java records automatically generate common methods such as equals(), hashCode(), toString(), and accessors for the fields.

Immutability: The fields in a record are immutable, making them ideal for use cases where data integrity is important.

## Sealed Classes

A sealed class allows you to control the inheritance hierarchy, offering more control over class extensions than traditional classes.

## Virtual Threads

## Sequenced Collections

guarantee the order of elements
