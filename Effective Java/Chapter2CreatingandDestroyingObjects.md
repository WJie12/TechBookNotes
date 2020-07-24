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