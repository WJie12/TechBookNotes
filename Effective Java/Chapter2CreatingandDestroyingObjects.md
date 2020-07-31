# Chapter 2: Creating and Destroying Objects

- When and how to create objects
- When and how to avoid creating objects
- How to ensure objects are destroyed in a timely manner
- How to manage any cleanup actions that must precede their destruction

## Item 1: Consider static factory methods instaed of constrctors

The ways for a class to allow a client to obtain an insatnce of itself:

- provide a public constructor
- provide a public static factory method

A class can provide its clients with static factory methods instead of, or in addition to, constructors.

```Java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE: Boolean.False;
}
```

Providing a static factory method instaed of a public constructor:

- Advantages

  - Unlike constructors, static factory methods have names

  - Static factory methods are not required to create a new object each time they're invoked.

    - Return the same object from repeated invocations -> Allow classes to maintain strict control over what instances exist at any time (instance controlled classes)

      - Guarantee that it is a singleton or noninstantiable
      - Allow an immutable class to make the guarantee that no two equal insatnces exits: `a.euqals(b)` if and only if `a==b`. Then its clients can use the  `==` instaed of `equals(Object)` which result in imroved performance. (Enum types provide the guarantee)
  
  - Can return an object of any subtype of their return type
    - Allow great flexibility in choosing the class of the returned object
    - An API can return objects without making their classed public
    - The class of an object returned can vary form invocation to invocation
    - The class of the object returned by a static factory method need not even exist at the time the class containing the methods is written
      - form the basis of `service provider frameworks` (JDBC)
        - A service provider framework is a system in which multiple service providers implement a service, and the system makes the implementations available to its clients, decoupling them from the implementations.
        - components:
          - service interface (`Connection`): providers implement
          - provider registration API (`DriverMnager.registerDriver`): system uses to register implementations, giving clients access to them
          - service access API (`DriverManager.getConnection`): clients use to obtain an instance of the service
            - The service access API is the “flexible static factory” that forms the basis of the service provider framework; allow but do not require the client to specify some criteria for choosing a provider (returns a default implementation)
          - (optional) service provider interface (`Driver`): providers implement to create instances of their service implementation
    - Reduce the verbosity of creating parameterized type instances

```Java
// Constructor
Map<String, List<String>> m = 
    new HashMap<String, list<String>>();

// Static factories
// type inference
public static <K, V> HashMap<K, V> newInstacne() {
    return new HashMap<K, V>();
}
Map<String, List<String>> m = HashMap.newInstacne()
```

- Disadvantages
  - Classes without public or protected constructors cannot be subclassed
    - The same is true for nonpublic classes returned by public static factories
    - Arguably this can be a blessing in disguise, as it encourages programmers to use composition instead of inheritance
  - Not readily distinguishable from other static methods
    - Can reduce the disadvantage by drawing attention to static factories in class or interface comments, and by adhering to common naming conventions.
    - Common names for static factory methods:
      - `valueOf`：Returns an instance that has the same value as its parameters. Such static factories are effectively type-conversion methods.
      - `of`: A concise alternative to valueOf, popularized by EnumSet
      - `getInstance`: Returns an instance that is described by the parameters but cannot be said to have the same value. In the case of a singleton, getInstance takes no parameters and returns the sole instance
      - `newInstance`: Like `getInstance`, except that `newInstance` guarantees that each instance returned is distinct from all others
      - `getType`: Like `getInstance`, but used when the factory method is in a different class. `Type` indicates the type of object returned by the factory method.
      - `newType`: Like `newInstance`, but used when the factory method is in a different class. `Type` indicates the type of object returned by the factory method

## Item 2: Consider a builder when faced with many constructor parameters

- Static factories and constructirs do not sacle well to large numbers of optional parameters

- Telescoping constructors pattern: provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.
  - It works, but it is hard to write client code when there are many parameters, and harder still to read it.

```Java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
  private final int servingSize; // (mL) required
  private final int servings; // (per container) required
  private final int calories; // optional
  private final int fat; // (g) optional
  private final int sodium; // (mg) optional
  private final int carbohydrate; // (g) optional
  public NutritionFacts(int servingSize, int servings) {
    this(servingSize, servings, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories) {
    this(servingSize, servings, calories, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat) {
    this(servingSize, servings, calories, fat, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat, int sodium) {
    this(servingSize, servings, calories, fat, sodium, 0);
  }
  public NutritionFacts(int servingSize, int servings,
  int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize = servingSize;
    this.servings = servings;
    this.calories = calories;
    this.fat = fat;
    this.sodium = sodium;
    this.carbohydrate = carbohydrate;
  }
}

NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

- JavaBeans Pattern: call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest
  - It is easy to create instances, and easy to read the resulting code
  - A JavaBean may be in an inconsistent state partway through its construction
  - precludes the possibility of making a class immutable, and requires added effort on the part of the programmer to ensure thread safety

```Java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
  // Parameters initialized to default values (if any)
  private int servingSize = -1; // Required; no default value
  private int servings = -1; // " " " "
  private int calories = 0;
  private int fat = 0;
  private int sodium = 0;
  private int carbohydrate = 0;
  public NutritionFacts() { }
  // Setters
  public void setServingSize(int val) { servingSize = val; }
  public void setServings(int val) { servings = val; }
  public void setCalories(int val) { calories = val; }
  public void setFat(int val) { fat = val; }
  public void setSodium(int val) { sodium = val; }
  public void setCarbohydrate(int val) { carbohydrate = val; }
}

NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

- Builder pattern: Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters
and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is immutable. The builder is a static member class (Item 22) of the class it builds.
  - Advantages
    - easy to write and read; simulates named optional parameters as found in Ada and Python
    - Compared with constructors, builders can have multiple varags paramters
    - Builder pattern is flexible. A single builder can be used to build multiple objects. The builder can fill in some fields automatically, such as a serial number that automatically increases each time an object is created.
    - A builder whose parameters have been set makes a fine Abstract Factory
      - a client can pass such a builder to a method to enable the method to create one or more objects for the client
      - need a type to represent the builder. For release 1.5 and later, a single generic type (Item 26) suffices for all builders
      - Methods that take a Builder instance would typically constrain the builder’s type parameter using a bounded wildcard type
      - Tradtional Abstract Factory implementation in Java has been the Class object, with the newInstance method playing the part of the build method, `Class.newInstance` breaks compile-time exception checking.
    - Disadvantages
      - must first create its builder before create an object
      - more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters. (but parameters may add in the future)

```Java
// Builder Pattern
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;
  public static class Builder {
    // Required parameters
    private final int servingSize;
    private final int servings;
    // Optional parameters - initialized to default values
    private int calories = 0;
    private int fat = 0;
    private int carbohydrate = 0;
    private int sodium = 0;
    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings = servings;
    }
    public Builder calories(int val){
      calories = val;
      return this;
    }
    public Builder fat(int val){
      fat = val;
      return this;
    }
    public Builder carbohydrate(int val){
      carbohydrate = val;
      return this;
    }
    public Builder sodium(int val){
      sodium = val;
      return this;
    }
    public NutritionFacts build() {
      return new NutritionFacts(this);
    }
  }
  private NutritionFacts(Builder builder) {
    servingSize = builder.servingSize;
    servings = builder.servings;
    calories = builder.calories;
    fat = builder.fat;
    sodium = builder.sodium;
    carbohydrate = builder.carbohydrate;
  }
}

NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
calories(100).sodium(35).carbohydrate(27).build();
```

```Java
// A builder for objects of type T
public interface Builder<T> {
  public T build();
}

//builds a tree using a client-provided Builder instance to build each node
Tree buildTree(Builder<? extends Node> nodeBuilder) { ... }
```

- Summary
  - the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if most of those parameters are optional
  - Client code is much easier to read and write with builders than with the traditional telescoping constructor pattern, and builders are much safer than JavaBeans.

## Item 3: Enforce the singleton property with a private constructor or an enum type

- A singleton is simply a class that is instantiated exactly once
- Making a class a singleton can make it difficult to test its clients

- 3 ways to implement singletons
  - a final field
    - can guarantees only one instance will exist once initialized
    - a privileged client can invoke the private constructor reflectively (Item 53) with the aid of the `AccessibleObject.setAccessible` method. If you need to defend against this attack, modify the constructor to make it throw an exception if it’s asked to create a second instance.
  - a static factory method
    - All calls return the same object reference
    - declarations make it clear that the class is a singleton
    - it gives you the flexibility to change your mind about whether the class should be a singleton without changing its API.
  - make a enum type with one element
    - functionally equivalent to the public field approach
    - more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks.
    - a single-element enum type is the best way to implement a singleton.

```Java
// Singleton with public final field
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public void leaveTheBuilding() { ... }
}

// Singleton with static factory
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }
  public void leaveTheBuilding() { ... }
}

// Enum singleton - the preferred approach
public enum Elvis {
  INSTANCE;
  public void leaveTheBuilding() { ... }
}
```

- To make a singleton class that is implemented using first two of the previous
approaches serializable, you have to declare all instance fields transient and provide a readResolve method (Item 77) to maintain the singleton guarantee.

```Java
// readResolve method to preserve singleton property
private Object readResolve() {
  // Return the one true Elvis and let the garbage collector
  // take care of the Elvis impersonator.
  return INSTANCE;
}
```

## Item 4: Enforce noninstantiability with a private constructor

- Utility classes which is just a grouping of static methods and static fields were not designed to be instantiated: an instance would be nonsensical.
- However, the compiler provides a public, parameterless default constructor which is indistinguisable from any other to a user.
- Attempting to enforce noninstantiability by making a class abstract does not work.
  - The class can be subclassed and the subclass instantiated
  - misleads the user into thinking the class was designed for inheritance
- a class can be made noninstantiable by including a private constructor
  - The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class.
  - This idiom is mildly counterintuitive, as the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment.
  - this idiom also prevents the class from being subclassed

```Java
// Noninstantiable utility class
public class UtilityClass {
  // Suppress default constructor for noninstantiability
  private UtilityClass() {
    throw new AssertionError();
  }
... // Remainder omitted
}
```






