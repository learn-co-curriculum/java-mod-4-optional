# Optional

## Learning Goals

- Explain the `Optional` class and its importance
- Use common methods provided by the `Optional` class

## What is Optional?

The `Optional<T>` class was created to better handle the absence or presence of
values. It creates a container to hold a value and provides methods to check the
state of the container. It is much less prone to error than using `null` to
signify the absence or presence of a value.

## Creating Optional Objects

There are three main methods for creating `Optional` objects:

- `empty()`: this creates an empty optional, i.e., it contains a null value.
- `of()`: this is used for creating optionals with non-null values. It throws a
  `Null Pointer Exception` if a null value is passed to the method.
- `ofNullable()`: this method can be used to create an optional if we’re not
  sure if the value passed is null or not.

Here are examples of each of these methods:

```java
import java.util.Optional;

public class Example {
    public static void main(String[] args) {
        Optional<String> emptyBird = Optional.empty();
        Optional<String> bird = Optional.of("Pigeon");
        Optional<String> nullBird = Optional.ofNullable(null);

        System.out.println(emptyBird);
        System.out.println(bird);
        System.out.println(nullBird);
    }
}
```

```
Optional.empty
Optional[Pigeon]
Optional.empty
```

## Using the Optional Objects

### Retrieving Values

There are several ways of gettiing a value from an `Optional` object. The 3 main
ways of getting values from optionals are:

- `get`
- `orElse(T other)`
- `orElseGet(Supplier<? extends T> other)`

We’ll start with the simplest `get` method. This method throws an exception if
it’s called on an empty optional.

```java
public class Example {
    public static void main(String[] args) {
        Optional<String> emptyBird = Optional.empty();
        Optional<String> bird = Optional.of("Pigeon");

        System.out.println(bird.get()); // "Pigeon"
        System.out.println(emptyBird.get()); // throws NoSuchElementException error
    }
}
```

The `orElse` and the `orElseGet` method can be used to retrieve a default value
if the optional is empty. The `orElseGet` method takes in a supplier function
instead of a value like the `orElse` method.

```java
public class Example {
    public static void main(String[] args) {
        Optional<String> pigeon = Optional.of("Pigeon");
        Optional<String> emptyBird = Optional.empty();

        String birdName = pigeon.orElse("no bird");
        System.out.println(birdName); // Pigeon

        String emptyBirdName = emptyBird.orElse("no bird");
        System.out.println(emptyBirdName); // no bird

        String emptyBirdName2 = emptyBird.orElseGet(() -> "no bird");
        System.out.println(emptyBirdName2); // no bird
    }
}
```

### Conditionals with Optional

We can use the following methods to use conditionals with optionals:

- `isPresent()`: returns `true` if the optional has a value and `false`
  otherwise.
- `ifPresent(Consumer<? super T> action)`: runs the given action if a value is
  present, otherwise it does nothing.
- `ifPresntOrElse(Consumer<? super T> action, Runnable emptyAction)`: runs the
  `action` if a value is present, otherwise runs the `emptyAction`.

Let’s look at some examples to understand their usage.

```java
public class Example {
    public static void main(String[] args) {
        Optional<String> pigeon = Optional.of("Pigeon");
        Optional<String> emptyBird = Optional.empty();

        if (pigeon.isPresent()) {
            System.out.println("there's a value!");
        }

        pigeon.ifPresent((value) -> System.out.println("there's a " + value + " in here!"));

        emptyBird.ifPresentOrElse(
                (value) -> System.out.println("this one has a " + value + "!"),
                () -> System.out.println("there's no bird here :(")
        );
    }
}
```

```
there's a value!
there's a Pigeon in here!
there's no bird here :(
```

## Conclusion

The `Optional` class is great for handling values we’re uncertain of. Create an
optional when dealing with possible null values to avoid errors.
