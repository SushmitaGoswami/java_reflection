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

## Constructors

### Find all the constructor of a class
1. **using class.getConstructors()** - This method returns all the public and non public constructors.
2. **using class.getDeclaredConstructors()** - This method returns only public constructors.

### How to access a private constructor of a class?
```
1. class.getDeclaredConstructors(args) -- will give all the public and private constructors.
2. constructor.setAccesible(true) -- for private constructor to be accesible via reflection.
3. constructor.newInstance(args)
```
For example, Configuration classs/object in Spring or ServerConfiguration in web server application uses this feature.

### How to access a package private class?
```
1. class.getDeclaredConstructors(args) -- will give all the public and private constructors.
2. constructor.setAccesible(true) -- for private constructor to be accesible via reflection.
```
Following are the some of the scenarios where java reflection comes handy for a package private classes.
1. This is used by the ObjectMapper implementation for example GSON,Jersey.
2. This feature is useful to create object of package private classes from the external packages. For example, dependency injection of Spring uses this feature.

## Fields
Using the Fields class, it is possible to get the name and type of the fields and also the runtime value of any field of an object.
```
1. class.getDeclaredFields() - get all the fields (both private and public). But it never gives the inherited fields.
2. class.getFields() - get all the public fields. It returns the inherited fields too.
3. get static variable (alternative approach) - class.getDeclaredField("field_name").get(null).
```

### Synthetic fields
A synthetic field is a compiler-created field that links a local inner class to a block's local variable or reference type parameter. The compiler synthesizes certain hidden fields and methods in order to implement the scoping of names.

For example,
1. On calling the getDeclaredField method on an inner class, by default compiler will create an object with name this$0
2. On calling the getDeclaredField method on an enum, by default compiler will create an object of name $VALUES.


### Settings field value dynamically
Using reflection, it is possible to set field values in runtime. Following are some of the uses cases of this functionality.
1. Network protocol deserializer 
2. Object deserializer
3. ORM - Object Relational Mapping.
4. Application File parser (e.g. property file parser)

**Note**: This functionality shouldn't be used with static variable as this might cause some expected outputs.



## Array
Using reflection, it is possible to analysize an array elements in runtime.
```
1. class.isArray() - It returns true the class object is of type Array.
2. class.getComponentType - It returns the type of array elements.
3. Array class - It contains several static helper method to analyze ar array elements in runtime.
```

### Creation of array in Runtime.
It is possible to create and set value of an array in Runtime.
```
1. Array.newInstance() -- creates an array of element
2. Array.set() -- set the value of an element at runtime.
```

## Methods
```
1. class.getDeclaredMethods() - All methods except the inherited one.
2. class.getMethods() - All public methods including the inherited methods.
```
### Method properties
1. class.getName() -- method name
2. class.getReturnType -- return type
3. class.getParameterTypes & class.getParameterCounts -- return no of parameters.
4. class.getExceptionTypes - exception returned by the method.

**Real life examples** :- getters validation in test framework.

### How to get a method by name and parameter types?
```
1. class.getMethod : -- all the methods in that class
2. class.getDeclaredMethod: all the pulic methods in that class and parent class.
```
**Note**: we can't use method parameter name as those are being removed after compilation. (e.g. replaced names would be arg0, arg1)

**Real life examples** :- setters validation in test framework.

### Method Invocation
1. Method invocation is one of the powerful way of decoupling the business/test logic from the execution logic.
2. We can group and execute methods which have no interface in common. Thus, it is possible to achieve the power of Polymorphism.

All the exception thrown by **method.invoke()** method is wrapped using InvocationTargetException.

**Real life examples** - 
1. Implement polymosrphism mechanism to write a generic method executor for different classes.
2. Implement sample test frameworks.

## Modifiers
Modifiers can be of two types - 
1. **Access Modifier** - a.public, b.private, c.default, d.private-package.
2. **Non Access Modifier** - a.static, b.transient, c. volatile

Following is the way to find the modifiers of different entities.
- class.getModifiers() - 
- method.getModifers() -
- field.getModifiers() -

**Real life use case** - Json Serializer which leaves the transient variable from serialization.

## Annotations
Annotation doesn't add any additional functionalities to a program, rather it attach a meta data to the target it is annotated with. Using different annotation, it is possible to find errors and warnings at compiler level. Most of the cases, annotations are retained at the compiler, but JVM ignores it. 

### Annotations and Reflection
It is possible to analyze annotation in the runtime using reflection. Some of the possible use cases of that are as follows. It allows decoupling of applicaion and reflection logic.

1. Json serializer - It is possible to decouple the actual serialized name of a field from the function argument.
2. Configuration initializers,
3. Program sequence for test frameworks,

### Meta Annotation
Meta annotation attaches additional marking or information to the other annotation. Example of some of the metat annotations are as follows.
1. **@Target** - This annotation specifies to which entity a particular annotation can be attached to. It can be Field, Method, class etc.
2. **@Retention** - This annotation defines the scope and lifetime of an annotation. It can be any of the following three.
     > RetentionPolicy.SOURCE - Annotation will be ignored by the compiler itself.
     > RetentionPolicy.CLASS - Annotation will be retained at the compiler level and discarded by the JVM.
     > RetentionPolicy.RUNTIME - Annotation will be retained at the compiler level and also processed by the JVM.


### Annotation Elements
Annotation elements can be any of the following types
- primitive types
- string
- Date
- Collection<?>
- Class<?> 
- and many others

### Analyze an annotation

Annotation can be obtained and analyzed at runtime using the following method,
- **Target.isAnnotationPresent(Class</?> clz)** - return true/false. 
- **Target.getAnnotation(Class</?> clz)** - Return the attached annotation.

Target can be any of the following
- Method
- Class
- Parameter
- Field etc.

### Repeatable Annotation

If an annotation need to be applied multiple times to an entity, then it needs to be declared as repeatable and declare a container annotation. Following is the example of repeatable annotation.

```
@Target(ElementType.Method)
@Repeatable(Roles.class)
public @interface Role{
  String value();
}

@Retention(RetentionPolicy.RUNTIME)
public @interface Roles{
  Role[] value();
}
```

What happens behind the scene?
```
@Role(value="test1")
@Role(value="test2")
public void test(){
....
}
```
is converted to following snippet by the compiler, i.e it replaces the repetable annotation by the container annotation.

```
@Roles(value={"test1","test2"})
public void test(){
....
}
```

So, only the container annotation is available at runtime.


## Dynamic Proxy
- Dynamic Proxy is a class created at Runtime and it is usually starts with $Proxy.
- Dynamic Proxy implements the methods of interface that we provide while creating the proxy object.
- It intercepts each method call and delegates the call to an instance of InvocationHandler. Instead of the proxy holding the reference to the original object, Invocationhandler has the reference and invoke the method on that object in runtime from invoke() method.

## Advantage over static proxy design pattern
Let's say, we have to design a time measuring proxy which will measure the time taken while executing methods. Now, if there are n no of interfaces and m no of methods in each interface, then we have to write n no of proxy classes and duplicate the delegation calls to the original object in m methods. However, using dynamic proxy, we can implement a single proxy invocation handler to accomplish this.

### Usecases
- Caching Implementation
- Security purposes
- Better resource utilization (e.g. lazy initialization)
- Remote method Invocation


## Disadvantage of Reflection
- A bit slower
- Less Type safety
- Less Code safety
- No Compiler time optimization
- Every edge cases and exception needs to be cared off!

