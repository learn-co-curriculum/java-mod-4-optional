# Optional

## Learning Goals

- Describe the use of `null` to represent the absence of a value.
- Define the [java.util.Optional](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) class and its methods for handling the absence or presence of a value.
- Refactor a `null` implementation to use `Optional`.

## What is Optional?

The `Optional<T>` class was created to better handle a variable with an optional value.
It creates a container to hold a value and provides methods to check the
state of the container. It is much less prone to error than using `null` and
presents an object-oriented way to handle optional values.

## The issue with `null`

Every variable declared with a class type may potentially store a `null` value.  

Consider an apartment rental company that charges an extra monthly fee for parking.
The fee is based on the vehicle category: car, truck, or van.
The `Vehicle` class has a field to store the category, along with other fields not shown (VIN, etc).

```java
public enum VehicleCategory {
        TRUCK, VAN, CAR
}

public class Vehicle {
  private VehicleCategory category;

  public Vehicle(VehicleCategory category) {this.category = category;}

  public VehicleCategory getCategory() {return category;}
}
```

The `Tenant` class below stores fields for `name` and `vehicle`.
Every tenant has a name, but not everyone has vehicle. The two constructors allow for the possibility of a tenant not owning a vehicle.
If the first constructor `Tenant(String name)` is called,
the `vehicle` variable will remain the default initialized value `null`.
In general, we assume `null` is not passed into a method through a parameter variable.
Thus, each tenant will have a non-null `name`, but `vehicle` may be null.

```java
public class Tenant {
    private String name;
    private Vehicle vehicle;

    public Tenant(String name) {
        this.name = name;
    }

    public Tenant(String name, Vehicle vehicle) {
        this.name = name;
        this.vehicle = vehicle;
    }

    public String getName() {return name;}
  
    public Vehicle getVehicle() {return vehicle; }
}
```

While we can comment the methods to indicate `getName()` is expected to return a value
but `getVehicle()` may return a null, there wasn't a programmatic way to
indicate a potentially absent value until `Optional` was introduced in Java 8.

Every class that uses the `Tenant` class must explicitly test for `null` when calling `getVehicle()`.
In the example below, the `printVehicleFee()` method performs a null check on 
the value returned from `getVehicle()` prior to getting the vehicle category.

```java
public class Main1 {

  public static void printVehicleFee(Tenant tenant) {
    Vehicle vehicle = tenant.getVehicle();
    if (vehicle != null) {
      VehicleCategory category = vehicle.getCategory();
      double fee = (category == VehicleCategory.CAR) ? 50.00 : 75.50 ;
      System.out.println(String.format("%s %s fee:$%.2f", tenant.getName(), category, fee ));
    }
    else {
      System.out.println(String.format("%s no vehicle fee", tenant.getName() ));
    }
  }

  public static void main(String[] args) {
    printVehicleFee(new Tenant("Pat Smith"));
    printVehicleFee(new Tenant("Havi Jones", new Vehicle(VehicleCategory.CAR)));
    printVehicleFee( new Tenant("Mina Wilson", new Vehicle(VehicleCategory.TRUCK)));
  }
}
```

As another example, consider the `findByName()` method below.
The method uses the `name` parameter to find the corresponding tenant in an array, returning `null` if a match is not found.
The `main` method must perform a null check on the result returned from calling `findByName()`.
While we could add comments to the `findByName()` method,
there is nothing explicit in the method signature itself that warns of a possible null return.

```java
import java.util.Scanner;

public class Main2 {

  public static final Tenant[] residents = {
          new Tenant("Pat Smith"),
          new Tenant("Havi Jones", new Vehicle(VehicleCategory.CAR)),
          new Tenant("Mina Wilson", new Vehicle(VehicleCategory.TRUCK))
  };

  public static Tenant findByName(String name) {
    for (Tenant tenant: residents ) {
      if (tenant.getName().equals(name))
        return tenant;
    }
    return null;
  }

  public static void main(String[] args) {
    //lookup by name
    Scanner scanner = new Scanner(System.in);
    System.out.println("enter name:");
    String name = scanner.nextLine();
    Tenant tenant = findByName(name);

    if (tenant != null) {
      System.out.println("found " + name);
    }
    else {
      System.out.println("didn't find " + name);
    }
  }
}
```

## Creating Optional Objects

The `Optional<T>` class provides a class-level mechanism for representing optional values.

There are three main methods for creating `Optional` objects:

- `empty()`: this creates an empty optional, i.e., it contains a null value.
- `of()`: this is used for creating optionals with non-null values and throws a
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
    Optional<String> notNullBird = Optional.ofNullable("Robin");

    System.out.println(emptyBird);
    System.out.println(bird);
    System.out.println(nullBird);
    System.out.println(notNullBird);
  }
}
```

The program prints:

```
Optional.empty
Optional[Pigeon]
Optional.empty
Optional[Robin]
```

## Using the Optional Objects

### Retrieving Values

There are several ways of getting a value from an `Optional` object. The 3 main
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
- `ifPresent(Consumer<? super T> action)`: if a value is present, invoke the specified consumer with the value, otherwise does nothing.
- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`: if a value is present, invoke the specified consumer with the value, otherwise perform the runnable action.
  - A lambda expression that implements `Consumer` takes a parameter, while `Runnable` expects no parameter. 

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

The parameter `value` in the lambda expression is the value stored in the optional,
i.e. the value from calling `Optional.get()`.
In the example above, it would be the string "Pigeon".  

```
there's a value!
there's a Pigeon in here!
there's no bird here :(
```

## Refactoring a null-based implementation to use `Optional`

Let's fix the `Vehicle` class to address the possibility that `getVehicle()` may return a value
that is not an instance of class `Vehicle`.  

### Creating objects with `Optional.ofNullable()`

We adapt the `getVehicle()` return type to `Optional<Vehicle>`.
Since the `vehicle` field may or may not be null, the method returns `Optional.ofNullable(vehicle)`.

```java
import java.util.Optional;

public class Tenant {
    private String name;
    private Vehicle vehicle;

    public Tenant(String name) {
        this.name = name;
    }

    public Tenant(String name, Vehicle vehicle) {
        this.name = name;
        this.vehicle = vehicle;
    }

    public String getName() {
        return name;
    }

    public Optional<Vehicle> getVehicle() {
        return Optional.ofNullable(vehicle);
    }
}
```

### Calling `isPresent()`

Any client that calls the `getVehicle()` method is responsible for checking for the presence of a value.
We evolve the `printVehicleFee()` method to handle the optional value returned from `getVehicle()`:

```java
import java.util.Optional;

public class Main1 {

  public static void printVehicleFee(Tenant tenant) {
    Optional<Vehicle> vehicle = tenant.getVehicle();
    if (vehicle.isPresent()) {
      VehicleCategory category = vehicle.get().getCategory();
      double fee = (category == VehicleCategory.CAR) ? 50.00 : 75.50 ;
      System.out.println(String.format("%s %s fee:$%.2f", tenant.getName(), category, fee ));
    }
    else {
      System.out.println(String.format("%s no vehicle fee", tenant.getName() ));
    }
  }

  public static void main(String[] args) {
    printVehicleFee(new Tenant("Pat Smith"));
    printVehicleFee(new Tenant("Havi Jones", new Vehicle(VehicleCategory.CAR)));
    printVehicleFee( new Tenant("Mina Wilson", new Vehicle(VehicleCategory.TRUCK)));
  }
}
```

Let's step through the changes from the original null-based version:

| Code                                                      | Description                                                                                                               |
|-----------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| `Optional<Vehicle> vehicle = tenant.getVehicle();`        | The value returned from `getValue`<br> is assigned to an `Optional<Vehicle>` variable.                                    | 
| `if (vehicle.isPresent()) {`                              | We call `vehicle.isPresent()`<br> rather than testing for null.                                                           | 
| `VehicleCategory category = vehicle.get().getCategory();` | `vehicle.get()` retrieves the `Vehicle`<br> object contained in the optional.<br>That object is used to get the category. | 


### Calling `ifPresentOrElse`

We can also rewrite the method to use `ifPresentOrElse` rather than a conditional statement: 

```java
   public static void printVehicleFee(Tenant tenant) {
        Optional<Vehicle> vehicle = tenant.getVehicle();
        vehicle.ifPresentOrElse(
                (v) -> {
                    VehicleCategory category = v.getCategory();
                    double fee = (category == VehicleCategory.CAR) ? 50.00 : 75.50 ;
                    System.out.println(String.format("%s %s fee:$%.2f", tenant.getName(), category, fee ));
                    },
                () -> {System.out.println(String.format("%s no vehicle fee", tenant.getName() ));}
        );
    }
```

The parameter `v` passed into the lambda expression is the value contained in the optional, which is an instance of `Vehicle`.
Thus, we invoke `getCategory()` directly on the object passed into the function.

### Creating objects with `Optional.of()` and `Optional.empty()`

We will also evolve the `findByName(String)` method to return an `Optional<Tenant>` rather than returning null.
The method uses `Optional.of()` when a matching tenant is found, and `Optional.empty()` when there is not a matching tenant.

```java
import java.util.Scanner;
import java.util.Optional;

public class Main2 {

  public static final Tenant[] residents = {
          new Tenant("Pat Smith"),
          new Tenant("Havi Jones", new Vehicle(VehicleCategory.CAR)),
          new Tenant("Mina Wilson", new Vehicle(VehicleCategory.TRUCK))
  };

  public static Optional<Tenant> findByName(String name) {
    for (Tenant tenant: residents ) {
      if (tenant.getName().equals(name))
        return Optional.of(tenant);
    }
    return Optional.empty();
  }

  public static void main(String[] args) {
    //lookup by name
    Scanner scanner = new Scanner(System.in);
    System.out.println("enter name:");
    String name = scanner.nextLine();
    Optional<Tenant> tenant = findByName(name);

    if (tenant.isPresent())
      System.out.println("found " + name);
    else
      System.out.println("didn't find " + name);
  }
}
```

| Code                                                        | Description                                         |
|-------------------------------------------------------------|-----------------------------------------------------|
| `public static Optional<Tenant> findByName(String name) { ` | Return type is `Optional<Tenant>`                   | 
| `return Optional.of(tenant);`                               | return an `Optional<Tenant>` with a non null value  | 
| `return Optional.empty()`                                   | return an `Optional<Tenant>` with a null value      | 
| `Optional<Tenant> tenant = findByName(name);`               | Store the return value in an `Optional<Tenant>`     | 
| `if (tenant.isPresent()) {`                                 | Call `isPresent()` versus testing for null          | 


We can also rewrite the `main` method to use `IfPresentOrElse` to test the optional value.  
The first lambda expression requires parameter `(t)` to hold the non-null tenant value, 
even though the parameter variable is not used in the lambda body.

```java
    public static void main(String[] args) {
        //lookup by name
        Scanner scanner = new Scanner(System.in);
        System.out.println("enter name:");
        String name = scanner.nextLine();
        Optional<Tenant> tenant = findByName(name);

        tenant.ifPresentOrElse(
                (t) -> System.out.println("found " + name),
                () -> System.out.println("didn't find " + name)
        );
    }
```


## Summary

The `Optional` class provides an object-oriented way of managing null values.
A method return type should use `Optional` if it is likely to return a null value.
The client that calls the method is responsible for using the `Optional` methods to check for the absence or presence of the value.

## Resources
[java.util.Optional](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)