# Methods Common to All Objects

- *Object* is designed primarily for extension. All of its nonfinal methods (equals, hashCode, toString, clone and finalize) have explicit general contracts because they are designed to be overridden.

- This chapter is about how to override the nonfinal Object methods.

## Item 8: Obey the general contract when overriding *equals*

- Not to override the *equals* method, in which case each instance of the class is equal only to itself. If:
  - Each instance of the class is inherently unique
  - You don not care whether the class provides a "logical equality" test
  - A superclass has already overridden *equals*, and the superclass behavior is appropriate for this class
  - The class is private or package-private, and you care certain that its *equals* method will never be invoked
    -  Arguably, the *equals* method should be overridden to throw a error
- Override *equals* when 
  -  a class has a notion of logical equality that differs from mere object identity
  -  a superclass has not already overridden equals to implement the desired behavior

- *value* classes in generally 
  - overriding the equals method necessary to satisfy programmer expectations (find out whether instances are logically equivalent)
  - it enables instances to serve as map keys or set elements with predictable, desirable behavior.
- A class uses instances control to ensure that at most one object exists with each value (such as Enum class) does not require the *equals* method to be overriden
- When override *equals* method you should adhere to its general contract
  - Reflexive: For any non-null reference value x, `x.equals(x)` must return true. 
  - Symmetric: For any non-null reference values x and y, `x.equals(y)` must return true if and only if `y.equals(x)` returns true. 
    -  Once you’ve violated the equals contract, you simply don’t know how other objects will behave when confronted with your object. 
  - Transitive: For any non-null reference values x, y, z, if `x.equals(y)` returns true and `y.equals(z)` returns true, then `x.equals(z)` must return true. 
    - There is no way to extend an instantiable class and add a value component while preserving the equals contract, 
  - Consistent: For any non-null reference values x and y, multiple invocations of `x.equals(y)` consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified. 
  - For any non-null reference value x, `x.equals(null)` must return false.

## Item 9: Always override *hashCode* when you override equals

- You must override *hashCode* in every class that overrides *euqals*.

- The key provision that is violated when you fail to override *hashCode* is the second one: equal objets must have equal hash codes

- Hash function recipe:

  - Store some constant nonzero value, say, 17, in an int variable called result. 
- For each significant field f in your object (each field taken into account by the equals method, that is), do the following: 
  
  - Compute an int hash code c for the field: 
  
    - If the field is a boolean, compute (f ? 1 : 0) . 
  
    - If the field is a byte, char, short, or int, compute (int) f. 
  
    - If the field is a long, compute (int) (f ^ (f >>> 32)).
  
    - If the field is a float, compute Float.floatToIntBits(f).
  
    - If the field is a double, compute Double.doubleToLongBits(f), and then hash the resulting long.
  
    - If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required, compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, return 0 (or some other constant, but 0 is traditional).
  
    -  If the field is an array, treat it as if each element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine these values per step 2.b. If every element in an array field is significant, you can use one of the Arrays.hashCode methods added in release 1.5.
  - Combine the hash code c computed in step 2.a into result as follows:        result = 31 * result + c;  
- Return result. 
  - When you are finished writing the hashCode method, ask yourself whether equal instances have equal hash codes. Write unit tests to verify your intuition! If equal instances have unequal hash codes, figure out why and fix the problem.
- Do not be tempted to exclude significant parts of an object from the hash code computation to improve performance.