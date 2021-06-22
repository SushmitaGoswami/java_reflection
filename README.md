# java_reflection


## Reflection entrypoint
Class<?> is an entry point to get the insight of an application code.
  1. Runtime type
  2. Interface that it implements.
  3. Methods & Fields defined
  4. etc.

### How to get an object of Class<?> ?
1. **using .getClass() method on object** - Not applicable to primitive types. Only applicable to object.
2. **add .class to a Class** - Useful when we don't have an object of that class. Also applicable to the primitive types.
3. **Class.forName()** - It requires to provide the fully qualified class name. However, it is the least safest option among this three. But there are some situation, where it is the only option. For example, declaring a bean using xml method of inversion control in spring. It also doesn't support the primitive types.

### Find all the constructor of a class
1. **using class.getConstructors()** - This method returns all the public and non public constructors.
2. **using class.getDeclaredConstructors()** - This method returns only public constructors.


