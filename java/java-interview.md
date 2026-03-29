1. what is the difference between final, finalize and finalyy() ?
- final -> is keyword it can be usdalong mthod variable and classes.
1. method: when it used laong with method means method cannot be override
2. variable: when it used of variable then variable value can be change later
3. class: when it used on the class means class cannot be inherited later.
- finally -> is used with try() block and it will alywas executed whether or not exception in the code.
- finalize -> which is calle db garbage collector before performing the garbage collection.

2. What is Java? 
- Java is a high-level programming language that was developed by James Gosling in the year 1982.
- It is based on the principles of object-oriented programming and can be used to develop large-scale applications. 

3. Why is Java a platform independent language?
- Java language was developed so that it does not depend on any hardware or software because the compiler compiles the code and then converts it
- to platform-independent byte code which can be run on multiple systems.
- The only condition to run that byte code is for the machine to have a runtime environment (JRE) installed in it

4. Difference between Heap and Stack Memory in java. And how java utilizes this.
- Stack memory is the portion of memory that was assigned to every individual program. And it was fixed. On the other hand, Heap memory is the portion that was not allocated to the java program but
- it will be available for use by the java program when it is required, mostly during the runtime of the program.
Java Utilizes this memory as - 

- When we write a java program then all the variables, methods, etc are stored in the stack memory.
- And when we create any object in the java program then that object was created in the heap memory. And it was referenced from the stack memory.
- Example- Consider the below java program:
```java
class Main {
   public void printArray(int[] array){
       for(int i : array)
           System.out.println(i);
   }
   public static void main(String args[]) {
       int[] array = new int[10];
       printArray(array);
   }
}
```
For this java program. The stack and heap memory occupied by java is -
<img width="924" height="577" alt="Screenshot 2026-03-28 at 11 20 33 AM" src="https://github.com/user-attachments/assets/ea239ae8-7f2a-4e28-9c1a-b144aa267e8a" />


5. How is Java different from C++?
- C++ is only a  compiled language, whereas Java is compiled as well as an interpreted language.
- Java programs are machine-independent whereas a c++ program can run only in the machine in which it is compiled. 
- C++ allows users to use pointers in the program. Whereas java doesn’t allow it. Java internally uses pointers. 
- C++ supports the concept of Multiple inheritances whereas Java doesn't support this. And it is due to avoiding the complexity of name ambiguity that causes the diamond problem.

6. Pointers are used in C/ C++. Why does Java not make use of pointers?
- Pointers are quite complicated and unsafe to use by beginner programmers. Java focuses on code simplicity, and the usage of pointers can make it challenging.
- Pointer utilization can also cause potential errors. Moreover, security is also compromised if pointers are used because the users can directly access memory with the help of pointers.
- Thus, a certain level of abstraction is furnished by not including pointers in Java. Moreover, the usage of pointers can make the procedure of garbage collection quite slow and erroneous.
- Java makes use of references as these cannot be manipulated, unlike pointers.

7. What do you understand by an instance variable and a local variable?
- Instance variables are those variables that are accessible by all the methods in the class.
- They are declared outside the methods and inside the class. These variables describe the properties of an object and remain bound to it at any cost.
- All the objects of the class will have their copy of the variables for utilization. If any modification is done on these variables, then only that instance will be impacted by it,
- and all other class instances continue to remain unaffected.

8. What are the default values assigned to variables and instances in java?
-  There are no default values assigned to the variables in java. We need to initialize the value before using it.
-  Otherwise, it will throw a compilation error of (Variable might not be initialized). 
-  But for instance, if we create the object, then the default value will be initialized by the default constructor depending on the data type. 
-  If it is a reference, then it will be assigned to null. 
-  If it is numeric, then it will assign to 0.
-  If it is a boolean, then it will be assigned to false. Etc.

9. What do you mean by data encapsulation?
- Data Encapsulation is an Object-Oriented Programming concept of hiding the data attributes and their behaviours in a single unit.
- It helps developers to follow modularity while developing software by ensuring that each object is independent of other objects by having its own methods,
- attributes, and functionalities.
- It is used for the security of the private properties of an object and hence serves the purpose of data hiding.

10. Tell us something about JIT compiler.
JIT stands for Just-In-Time and it is used for improving the performance during run time. It does the task of compiling parts of byte code having similar functionality at the same time thereby reducing the amount of compilation time for the code to run.
The compiler is nothing but a translator of source code to machine-executable code. But what is special about the JIT compiler? Let us see how it works:
First, the Java source code (.java) conversion to byte code (.class) occurs with the help of the javac compiler.
Then, the .class files are loaded at run time by JVM and with the help of an interpreter, these are converted to machine understandable code.
JIT compiler is a part of JVM. When the JIT compiler is enabled, the JVM analyzes the method calls in the .class files and compiles them to get more efficient and native code. It also ensures that the prioritized method calls are optimized.
Once the above step is done, the JVM executes the optimized code directly instead of interpreting the code again. This increases the performance and speed of the execution.

<img width="736" height="458" alt="Screenshot 2026-03-28 at 12 19 27 PM" src="https://github.com/user-attachments/assets/a0bed504-2d33-4129-9fb5-799064edb4a2" />

11. == and .equals() are different in Java:
- == checks identity (for objects) and exact value (for primitives).
- .equals() checks logical equality based on class implementation.
- Summary:
- Primitives: use == for value comparison.
- Objects: == compares references (same object or not).
- Objects: .equals() compares content/value if overridden; otherwise behaves like reference comparison.
- Wrapper classes and String override .equals() to compare values.
- For custom classes, override both equals() and hashCode() for value-based equality.

12. Is it possible that the ‘finally’ block will not be executed? If yes then list the case.
- Yes. It is possible that the ‘finally’ block will not be executed. The cases are-
- Suppose we use System.exit() in the above statement.
- If there are fatal errors like Stack overflow, Memory access error, etc.

13. Can the static methods be overridden?
- No! Declaration of static methods having the same signature can be done in the subclass but run time polymorphism can not take place in such cases.
- Overriding or dynamic polymorphism occurs during the runtime, but the static methods are loaded and looked up at the compile time statically. Hence, these methods cant be overridden.

14. What is the main objective of garbage collection?
- The main objective of this process is to free up the memory space occupied by the unnecessary and unreachable objects during the Java program execution by deleting thos unreachable objects.
- This ensures that the memory resource is used efficiently, but it provides no guarantee that there would be sufficient memory for the program execution.

15. What is a ClassLoader?
- Java Classloader is the program that belongs to JRE (Java Runtime Environment). The task of ClassLoader is to load the required classes and interfaces to the JVM when required. 
- Example- To get input from the console, we require the scanner class. And the Scanner class is loaded by the ClassLoader.

A ClassLoader in Java is the JVM component that loads .class bytecode into memory and turns it into runtime Class objects.

Main points:

Loads classes only when needed (dynamic loading).
Follows parent delegation (asks parent first), which avoids duplicate/core class override issues.
Built-in loaders:
Bootstrap (core JDK classes)
Platform (JDK platform modules)
Application (your app classpath)
You can create a custom class loader for plugins/modules or special loading rules.

16. What part of memory - Stack or Heap - is cleaned in garbage collection process?
Heap.

Example - 

class Rectangle{
int length = 5;
     int breadth = 3;
}
Object for this Rectangle class - Rectangle obj1 = new Rectangle();

Shallow copy - The shallow copy only creates a new reference and points to the same object. Example - For Shallow copy, we can do this by -
Rectangle obj2 = obj1;
Now by doing this what will happen is the new reference is created with the name obj2 and that will point to the same memory location.

Deep Copy - In a deep copy, we create a new object and copy the old object value to the new object. Example -
Rectangle obj3 = new Rectangle();
Obj3.length = obj1.length;
Obj3.breadth = obj1.breadth;

## Java Intermediate Interview Questions

17. Apart from the security aspect, what are the reasons behind making strings immutable in Java?
1. String Pool Optimization
- Java stores string literals in the String Pool to reuse objects and save memory. This sharing works safely because String is immutable.
2. Thread Safety in Multithreading
- Immutable strings can be shared across threads without synchronization, reducing concurrency bugs and making code simpler.
3. Safe Keys in HashMap/Hashtable
- Strings are widely used as map keys. If a key changed after insertion, hash-based lookup would fail. Immutability keeps hash/value stable, so retrieval remains reliable.

18. How would you differentiate between a String, StringBuffer, and a StringBuilder?
- Storage area: In string, the String pool serves as the storage area. For StringBuilder and StringBuffer, heap memory is the storage area.
- Mutability: A String is immutable, whereas both the StringBuilder and StringBuffer are mutable.
- Efficiency: It is quite slow to work with a String. However, StringBuilder is the fastest in performing operations. The speed of a StringBuffer is more than a String and less than - a StringBuilder. (For example appending a character is fastest in StringBuilder and very slow in String because a new memory is required for the new String with appended character.)
- Thread-safe: In the case of a threaded environment, StringBuilder and StringBuffer are used whereas a String is not used. However, StringBuilder is suitable for an environment - - - with a single thread, and a StringBuffer is suitable for multiple threads.
Syntax:

```java
// String
String first = "InterviewBit";
String second = new String("InterviewBit");
// StringBuffer
StringBuffer third = new StringBuffer("InterviewBit");
// StringBuilder
StringBuilder fourth = new StringBuilder("InterviewBit");
```
19. Using relevant properties highlight the differences between interfaces and abstract classes.
Availability of methods: Only abstract methods are available in interfaces, whereas non-abstract methods can be present along with abstract methods in abstract classes.
Variable types: Static and final variables can only be declared in the case of interfaces, whereas abstract classes can also have non-static and non-final variables.
Inheritance: Multiple inheritances are facilitated by interfaces, whereas abstract classes do not promote multiple inheritances.
Data member accessibility: By default, the class data members of interfaces are of the public- type. Conversely, the class members for an abstract class can be protected or private also.
Implementation: With the help of an abstract class, the implementation of an interface is easily possible. However, the converse is not true;

20. What is a Comparator in java?
Consider the example where we have an ArrayList of employees like( EId, Ename, Salary), etc. Now if we want to sort this list of employees based on the names of employees. Then that is not possible to sort using the Collections.sort() method. We need to provide something to the sort() function depending on what values we have to perform sorting. Then in that case a comparator is used.

 # how hashtable works internally ?
HashTable + Bucket (Reference)
A hash table stores data as key -> value.
It uses a hash function to convert a key into an array index.
That array index is called a bucket.
In Java HashMap
HashMap has an internal array (table[]).
Each array slot is one bucket.
Each bucket stores entries (key, value, hash, next), not just keys.
How put(key, value) works
Compute hash from key.
Convert hash to bucket index.
Go to that bucket.
If key already exists, update value.
If not, add new entry in that bucket.
How get(key) works
Compute hash from key.
Jump to bucket index.
Check entries in that bucket.
Match key using equals().
Return value.
Collision
If different keys map to the same bucket, that is a collision.
HashMap handles collisions by keeping multiple entries in the same bucket.
Internally this is a linked structure, and for many collisions it can become a tree.
Your understanding (corrected)
Yes, it is like:

an array of buckets,
each bucket can hold a list/tree of key-value entries,
after reaching the bucket, map checks entries to find the exact key.

21. HashSet and TreeSet both store unique elements, but differ in behavior:

HashSet

Backed by hash table.
No insertion/sorted order guarantee.
Allows one null.
Average O(1) add/remove/search.
TreeSet

Backed by red-black tree.
Keeps elements sorted (natural order or Comparator).
Usually does not allow null (can throw NullPointerException).
O(log n) add/remove/search.
Use HashSet for fast lookups, TreeSet when you need sorted data.

22. Why is the character array preferred over string for storing confidential information?

String is immutable, so secret text stays in memory until GC decides to remove it.
You cannot erase a String after use.
String may also be accidentally exposed in logs/debug output more easily.
With char[], you can clear it immediately:
```java
char[] password = readPassword();
try {
    authenticate(password);
} finally {
    java.util.Arrays.fill(password, '\0'); // wipe from memory
}
```
So the main advantage is control over secret lifetime in memory.

23. What do we get in the JDK file?
JDK- For making java programs, we need some tools that are provided by JDK (Java Development Kit). JDK is the package that contains various tools, Compiler, Java Runtime Environment, etc.
JRE -  To execute the java program we need an environment. (Java Runtime Environment) JRE contains a library of Java classes +  JVM. What are JAVA Classes?  It contains some predefined methods that help Java programs to use that feature, build and execute. For example - there is a system class in java that contains the print-stream method, and with the help of this, we can print something on the console.
JVM - (Java Virtual Machine) JVM  is a part of JRE that executes the Java program at the end.  Actually, it is part of JRE, but it is software that converts bytecode into machine-executable code to execute on hardware.



24. HashMap vs Hashtable (again, short):

Thread safety: HashMap is not synchronized; Hashtable is synchronized.
Performance: HashMap is usually faster; Hashtable is slower due to locking.
Nulls: HashMap allows one null key and multiple null values; Hashtable allows no null key/value.
API age: HashMap is modern and preferred; Hashtable is legacy.
Internal idea: both use hash + bucket array, but HashMap has newer optimizations (like tree bins in heavy collisions).


25. difference between concurrentHashmap and hashtabl?
ConcurrentHashMap and Hashtable are both thread-safe, but they work very differently:
Hashtable: one global lock per map (synchronized methods).
Less concurrency, slower under high multi-thread load.
ConcurrentHashMap: fine-grained locking + CAS/non-blocking reads.
Multiple threads can read/write different buckets at same time, so much better throughput.
Iteration:
Hashtable iterator is typically fail-fast (ConcurrentModificationException).
ConcurrentHashMap iterator is weakly consistent (no CME, sees a moving snapshot).
null handling: both disallow null keys and null values.
ConcurrentHashMap also provides useful atomic methods like putIfAbsent, computeIfAbsent, replace.
In modern code, ConcurrentHashMap is usually preferred over Hashtable.


26. Reflection in Java is important because it lets a program inspect and use classes at runtime (without knowing all types at compile time).

Why it matters:

Build frameworks/libraries (Spring, Hibernate, JUnit) that auto-discover classes, annotations, methods, and fields.
Enable dependency injection, ORM mapping, and annotation-driven behavior.
Support dynamic plugin/module loading.
Help tooling, debugging, testing, and serialization.
Trade-offs:

Slower than direct calls
Can break encapsulation and reduce type safety if overused
So reflection is powerful for framework-level flexibility and runtime automation.


27. 15. What are the different ways of threads usage?
We can define and implement a thread in java using two ways:
Extending the Thread class
```java
class InterviewBitThreadExample extends Thread{  
   public void run(){  
       System.out.println("Thread runs...");  
   }  
   public static void main(String args[]){  
       InterviewBitThreadExample ib = new InterviewBitThreadExample();  
       ib.start();  
   }  
}
Implementing the Runnable interface
class InterviewBitThreadExample implements Runnable{  
   public void run(){  
       System.out.println("Thread runs...");  
   }  
   public static void main(String args[]){  
       Thread ib = new Thread(new InterviewBitThreadExample()); 
       ib.start();  
   }  
}
```
- Implementing a thread using the method of Runnable interface is more preferred and advantageous as Java does not have support for multiple inheritances of classes.
- start() method is used for creating a separate call stack for the thread execution. Once the call stack is created, JVM calls the run() method for executing the thread in that call stack.

28. What are the different types of Thread Priorities in Java? And what is the default priority of a thread assigned by JVM?
There are a total of 3 different types of priority available in Java. 

MIN_PRIORITY: It has an integer value assigned with 1.
MAX_PRIORITY: It has an integer value assigned with 10.
NORM_PRIORITY: It has an integer value assigned with 5.

In Java, Thread with MAX_PRIORITY gets the first chance to execute. But the default priority for any thread is NORM_PRIORITY assigned by JVM. 

29. What is the difference between the program and the process?
A program can be defined as a line of code written in order to accomplish a particular task. Whereas the process can be defined as the programs which are under execution. 
A program doesn't execute directly by the CPU. First, the resources are allocated to the program and when it is ready for execution then it is a process.

30. What is the difference between the ‘throw’ and ‘throws’ keyword in java?
The ‘throw’ keyword is used to manually throw the exception to the calling method.
And the ‘throws’ keyword is used in the function definition to inform the calling method that this method throws the exception. So if you are calling, then you have to handle the exception.
Example - 

class Main {
   public static int testExceptionDivide(int a, int b) throws ArithmeticException{
       if(a == 0 || b == 0)
           throw new ArithmeticException();
       return a/b;
   }
   public static void main(String args[]) {
       try{
           testExceptionDivide(10, 0);
       }
       catch(ArithmeticException e){
           //Handle the exception
       }
   }
}

31. How to not allow serialization of attributes of a class in Java?
In order to achieve this, the attribute can be declared along with the usage of transient keyword as shown below:
```java
public class InterviewBitExample { 

   private transient String someInfo; 
   private String name;
   private int id;
   // :
   // Getters setters
   // :
}
```
In the above example, all the fields except someInfo can be serialized.

32. What happens if the static modifier is not included in the main method signature in Java?
There wouldn't be any compilation error. But then the program is run, since the JVM cant map the main method signature, the code throws “NoSuchMethodError” error at the runtime.
Object cloning means creating a new object with the same state as an existing object.

In Java, classic cloning uses Cloneable + overriding clone():
```java
class Address implements Cloneable {
    String city;
    Address(String city) { this.city = city; }

    @Override
    protected Address clone() throws CloneNotSupportedException {
        return (Address) super.clone(); // shallow for this class
    }
}

class Person implements Cloneable {
    String name;
    Address address;

    Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Person clone() throws CloneNotSupportedException {
        Person p = (Person) super.clone(); // shallow copy
        p.address = address.clone();       // deep copy part
        return p;
    }
}
```
Types:

Shallow copy: object copied, nested object references shared.
Deep copy: nested objects also copied.
Note: In modern Java, many teams prefer copy constructors or factory methods over Cloneable because cloning API is tricky.

33. Is it mandatory for a catch block to be followed after a try block?
No, it is not necessary for a catch block to be present after a try block. - A try block should be followed either by a catch block or by a finally block. If the exceptions likelihood is more, then they should be declared using the throws clause of the method.


34. Will the finally block get executed when the return statement is written at the end of try block and catch block as shown below?
```java
public int someMethod(int i){
   try{
       //some statement
       return 1;
   }catch(Exception e){
       //some statement
       return 999;
   }finally{
       //finally block statements
   }
}
```
finally block will be executed irrespective of the exception or not. The only case where finally block is not executed is when it encounters ‘System.exit()’ method anywhere in try/catch block.

34. Can you call a constructor of a class inside the another constructor?
Yes, the concept can be termed as constructor chaining and can be achieved using this().

35. Contiguous memory locations are usually used for storing actual values in an array but not in ArrayList. Explain?
- In the case of ArrayList, data storing in the form of primitive data types (like int, float, etc.) is not possible. The data members/objects present in the ArrayList have references to the objects which are located at various sites in the memory. Thus, storing of actual objects or non-primitive data types (like Integer, Double, etc.) takes place in various memory locations.
- However, the same does not apply to the arrays. Object or primitive type values can be stored in arrays in contiguous memory locations, hence every element does not require any reference to the next element.

36. Why does the java array index start with 0?
It is because the 0 index array avoids the extra arithmetic operation to calculate the memory address.

37. Why is the remove method faster in the linked list than in an array?
In the linked list, we only need to adjust the references when we want to delete the element from either end or the front of the linked list. But in the array, indexes are used. So to manage proper indexing, we need to adjust the values from the array So this adjustment of value is costlier than the adjustment of references.

Example - To Delete from the front of the linked list, internally the references adjustments happened like this.

39. How does the size of ArrayList grow dynamically? And also state how it is implemented internally.

- ArrayList is implemented in such a way that it can grow dynamically. We don't need to specify the size of ArrayList. For adding the values in it, the methodology it uses is -

1. Consider initially that there are 2 elements in the ArrayList. [2, 3].
2. If we need to add the element into this. Then internally what will happen is-
ArrayList will allocate the new ArrayList of Size (current size + half of the current size). And add the old elements into the new. Old - [2, 3],    New - [2, 3, null].
3. This process continues and the time taken to perform all of these is considered as the amortized constant time. 
This is how the ArrayList grows dynamically. And when we delete any entry from the ArrayList then the following steps are performed -
It searches for the element index in the array. Searching takes some time. Typically it’s O(n) because it needs to search for the element in the entire array.

## Java Interview Questions for Experienced

40. Although inheritance is a popular OOPs concept, it is less advantageous than composition. Explain.
Inheritance lags behind composition in the following scenarios:


Inheritance is useful, but composition is often better because it gives lower coupling and more flexibility.

41. Why composition is usually preferred:

Inheritance creates tight parent-child coupling; parent changes can break children.
It exposes subclass to internals of base class (fragile base class problem).
Java supports only single class inheritance, which limits reuse.
Deep inheritance trees become hard to understand and maintain.
Composition lets you swap behavior at runtime by combining small components.
It follows “favor composition over inheritance” and fits SOLID better (especially SRP/OCP).

42. How is the ‘new’ operator different from the ‘newInstance()’ operator in java?
- new is used when class type is known at compile time.
- newInstance() (reflection) is used when class is known only at runtime (from config/DB/input).
- newInstance() can throw checked exceptions (like class not found / instantiation issues), so exception handling is required.

43. Can you explain the Java thread lifecycle?
Java thread life cycle is as follows:

New – When the instance of the thread is created and the start() method has not been invoked, the thread is considered to be alive and hence in the NEW state.
Runnable – Once the start() method is invoked, before the run() method is called by JVM, the thread is said to be in RUNNABLE (ready to run) state. This state can also be entered from the Waiting or Sleeping state of the thread.
Running – When the run() method has been invoked and the thread starts its execution, the thread is said to be in a RUNNING state.
Non-Runnable (Blocked/Waiting) – When the thread is not able to run despite the fact of its aliveness, the thread is said to be in a NON-RUNNABLE state. Ideally, after some time of its aliveness, the thread should go to a runnable state.
A thread is said to be in a Blocked state if it wants to enter synchronized code but it is unable to as another thread is operating in that synchronized block on the same object. The first thread has to wait until the other thread exits the synchronized block.
A thread is said to be in a Waiting state if it is waiting for the signal to execute from another thread, i.e it waits for work until the signal is received.
Terminated – Once the run() method execution is completed, the thread is said to enter the TERMINATED step and is considered to not be alive.
The following flowchart clearly explains the lifecycle of the thread in Java.


44. What do you understand by marker interfaces in Java?
Marker interfaces, also known as tagging interfaces are those interfaces that have no methods and constants defined in them. They are there for helping the compiler and JVM to get run time-related information regarding the objects.

```java

import java.io.*;

// Marker interface in Java
class User implements Serializable {
    String name;
    User(String name) { this.name = name; }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        User u = new User("Asha");

        // Because User implements Serializable, object can be serialized
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.ser"));
        out.writeObject(u);
        out.close();

        System.out.println("Serialized successfully");
    }
}
```

45. Double-brace initialization is a Java pattern to create and fill an object in one expression using:

An anonymous inner class (new Type() { ... })
An instance initializer block ({{ ... }})
Example:
```java
List<String> names = new ArrayList<>() {{
    add("A");
    add("B");
}};
```
Why it’s usually avoided:

Creates an extra anonymous class
Can hold hidden reference to outer object (memory issues)
Harder to read/debug
Prefer List.of(...), Set.of(...), Map.of(...), or normal initialization.


46. In Java, an object becomes GC-eligible when it has no reachable strong reference.

Common ways:

Nullify reference
Employee e = new Employee();
e = null;
Reassign reference
Employee e = new Employee();
e = new Employee(); // old object now unreferenced
Reference goes out of scope (local variable ends)
void test() {
  Employee e = new Employee();
} // e out of scope
Anonymous object with no reference
new Employee();
Remove object references from collections/maps
list.remove(obj);
map.remove(key);

47. What is the best way to inject dependency? Also, state the reason.
There is no boundation for using a particular dependency injection. But the recommended approach is - 

Setters are mostly recommended for optional dependencies injection, and constructor arguments are recommended for mandatory ones. This is because constructor injection enables the injection of values into immutable fields and enables reading them more easily.

48. A memory leak in Java means objects are no longer useful but are still reachable by references, so GC cannot free them.

So even with garbage collection, memory keeps growing and may cause OutOfMemoryError.

Common causes:

Static collections holding objects forever
Unclosed resources/listeners
Caches without eviction
Wrong equals/hashCode causing map growth
ThreadLocal values not cleared
In short: GC works, but leaked objects stay referenced, so memory is not reclaimed.

49. What happens if you call return in try and finally also return?
If both try and finally have returned, the one in the end always wins. Java executes finally at the end no matter what happens in try or catch. So the return value from try is ignored and replaced by the return from finally.


# Scenario-Based Java Interview Questions

50. Your app is slow after running for 2 hours. What do you check first?
The first thing to check is memory usage to see if it keeps growing over time, which may indicate a memory leak. Then object creation is reviewed to see if too many objects are being created and not released. Application logs are checked because excessive logging can slow down performance. Database calls are also reviewed to find slow queries or stuck connections. Finally, CPU usage is monitored to identify any background tasks consuming too many resources.

51. API is timing out sometimes. How do you debug?
The issue is first reproduced to understand under what conditions it occurs. Application logs are then checked to identify where the delay is happening. Metrics such as response time, database latency, and external API time are reviewed. Dependencies are isolated to determine whether the issue comes from the database, network, or third-party services. Once identified, optimizations like query tuning, timeouts, or retries are applied.

52. You see OutOfMemoryError - what are 3 likely causes?
One common cause is too much caching where data keeps growing without limits. Another reason is large collections like lists or maps holding data longer than needed. Not closing resources such as database connections, streams, or files can also cause memory issues. Sometimes infinite loops keep adding objects to memory. Poor garbage collection tuning can also lead to this error.

53. Multiple users update the same record - how do you avoid wrong data?
One way is using database locks so only one user can update the record at a time. Another approach is versioning, where updates fail if the data has changed since it was last read. Atomic updates help ensure data changes happen safely in one step. Transactions are also used to keep updates consistent. These methods prevent users from overwriting each other’s changes.

54. A background job runs twice how do you prevent duplicates?
The job is designed to be idempotent so running it multiple times does not create duplicate results. A marker such as a processed ID or status is stored in the database. Before doing any work, the job checks whether the task has already been completed. If it is already processed, the job safely skips it. This ensures correct results even if the job runs twice.

55. You must handle 1 lakh requests per minute what changes in code or design?
Connection pooling is used so new connections are not created for every request. Caching is added to reduce repeated database or external API calls. Heavy processing is moved out of the request flow whenever possible. Asynchronous processing is used for non-critical tasks. These changes help the system remain fast and stable under high load.

56. A thread is stuck how do you identify why?
A thread dump is taken to see what the thread is currently doing. Locks are checked to find out if the thread is waiting for another thread to release a resource. Waiting or sleeping states are reviewed to identify long pauses. Application logs help confirm where execution stopped. This usually points to deadlocks or long-running operations.

57. You need to release resources reliably what pattern do you use?
Resources are always closed in a finally block to ensure they are released. In modern Java, try-with-resources is preferred for cleaner and safer code. This guarantees that files, streams, and database connections are closed automatically. Even if an exception occurs, resources are still freed. This prevents memory and connection leaks.


# Java Memory Management Interview Questions
57. Heap vs Stack (short recap)
Stack memory is used for method calls and local variables, and it works in a very fast way. Each thread has its own stack, so data is not shared. Heap memory is used for objects and is shared across the application. Objects live longer in heap compared to stack variables. Stack is small and fast, heap is bigger but slower. Stack memory is cleared automatically when a method ends.

58. What is garbage collection (GC) in simple words?
Garbage collection is how Java frees memory automatically. When an object is no longer needed, GC removes it from memory. This helps avoid manual memory cleanup like in C or C++. Developers don’t delete objects themselves. The JVM handles this in the background. This reduces chances of memory-related bugs.

59. What makes an object “unused” or unreachable?
An object becomes unused when no active part of the program refers to it. If there is no reference from stack, static variables, or other objects, it is unreachable. Once unreachable, the object cannot be used again. GC can then remove it. This usually happens after methods finish execution. Local variables going out of scope is a common reason.

60. What is a memory leak in Java?
A memory leak happens when objects are not needed but still kept in memory. For example, a static list keeps adding objects and is never cleared. Even though GC exists, it cannot remove these objects because they are still referenced. Over time, memory usage keeps growing. This can crash the app. Long-running apps suffer the most from leaks.


61. How do you reduce memory usage?
Memory usage is reduced by clearing caches that are no longer needed. Object references are removed once the work is completed so they can be garbage collected. Files, database connections, and streams are always closed properly. Storing large objects for a long time is avoided unless necessary. Regular memory monitoring helps identify and fix issues early.

62. OutOfMemoryError vs “app is slow” - what’s the difference?
OutOfMemoryError means the JVM cannot allocate more memory and the app may crash. App slowness usually means high CPU, heavy GC, or slow DB calls. Slow apps still run, but poorly. OOM is a serious failure. They are related but not the same problem. A slow app can eventually lead to OOM if ignored.

63. When does GC run?
GC runs when the JVM decides memory needs to be freed. Developers cannot control the exact timing. It usually runs when memory is low. You can request GC, but it may not run immediately. The JVM manages this automatically. Different GC algorithms behave differently.

# Java Full Stack Developer Interview Questions

64. How do you design a REST API in Spring Boot (high level)?
The design starts by defining clear resources and meaningful URLs based on the business domain. Correct HTTP methods like GET, POST, PUT, and DELETE are chosen for each operation. Proper HTTP status codes are returned for both success and error cases. Input validation is added at the request level to prevent invalid data. Controllers are kept thin, while business logic is handled in service classes. Error responses are kept consistent across all APIs.

65. How do you handle database transactions safely in a Java backend?
Database transactions are managed using transactional boundaries, commonly with annotations like @Transactional. This ensures that all operations either complete successfully or roll back together in case of failure. Long-running tasks are avoided inside transactions to reduce lock time. External API calls are kept outside transactions to prevent blocking and timeouts. Proper isolation levels are used to maintain data consistency and avoid deadlocks.

66. What causes N+1 queries in ORM like Hibernate and how do you fix it?
N+1 queries happen when a parent list is loaded first and child data is fetched one by one. This creates many unnecessary database calls. It usually occurs with lazy loading. To fix it, I use fetch joins or entity graphs. Batch fetching can also reduce queries. This improves performance significantly.

67. How do you improve API latency without just adding servers?
API latency is improved by caching frequently accessed data. Payload sizes are reduced by returning only required fields. Pagination is added to avoid sending large responses at once. Database queries and indexes are optimized for faster access. Connections are reused through pooling. Heavy processing is moved to asynchronous or background tasks.

68. How do you version APIs without breaking clients?
API versioning is done using URL paths or request headers. Backward compatibility is maintained wherever possible. Breaking changes are introduced only in new versions. Older versions are deprecated gradually instead of being removed suddenly. Clear timelines are shared with consumers to ensure smooth migration.

69. How do you approach debugging a production issue end to end?
The process starts by checking logs, metrics, and monitoring dashboards. Distributed traces are used to identify slow or failing services. The issue is reproduced with minimal data when possible. The problematic layer, such as UI, API, or database, is isolated. Fixes are validated with tests. Monitoring confirms stability after release.

70. What testing mix do you typically aim for in full stack work?
Unit tests are used to validate core business logic. Integration tests cover API endpoints and database interactions. End-to-end tests are added for critical user flows. The goal is to balance coverage without slowing down builds. Fast feedback is prioritized. This testing mix provides confidence before deployment.

71. What’s the difference between authentication and authorization in a web app?
Authentication is about verifying who the user is. Authorization decides what the user is allowed to access. A user must be authenticated before authorization is checked. In Java applications, this is commonly handled using Spring Security. Filters handle authentication, while roles or permissions control access. Both are important for secure systems.

72. How do you handle database transactions safely in a Java backend?
Database transactions are managed using transactional boundaries, commonly with annotations like @Transactional. This ensures that all operations either complete successfully or roll back together in case of failure. Long-running tasks are avoided inside transactions to reduce lock time. External API calls are kept outside transactions to prevent blocking and timeouts. Proper isolation levels are used to maintain data consistency and avoid deadlocks.

73. What causes N+1 queries in ORM like Hibernate and how do you fix it?
N+1 queries happen when a parent list is loaded first and child data is fetched one by one. This creates many unnecessary database calls. It usually occurs with lazy loading. To fix it, I use fetch joins or entity graphs. Batch fetching can also reduce queries. This improves performance significantly.

74. How do you improve API latency without just adding servers?
API latency is improved by caching frequently accessed data. Payload sizes are reduced by returning only required fields. Pagination is added to avoid sending large responses at once. Database queries and indexes are optimized for faster access. Connections are reused through pooling. Heavy processing is moved to asynchronous or background tasks.

75. How do you version APIs without breaking clients?
API versioning is done using URL paths or request headers. Backward compatibility is maintained wherever possible. Breaking changes are introduced only in new versions. Older versions are deprecated gradually instead of being removed suddenly. Clear timelines are shared with consumers to ensure smooth migration.

76. How do you approach debugging a production issue end to end?
The process starts by checking logs, metrics, and monitoring dashboards. Distributed traces are used to identify slow or failing services. The issue is reproduced with minimal data when possible. The problematic layer, such as UI, API, or database, is isolated. Fixes are validated with tests. Monitoring confirms stability after release.

77. What testing mix do you typically aim for in full stack work?
Unit tests are used to validate core business logic. Integration tests cover API endpoints and database interactions. End-to-end tests are added for critical user flows. The goal is to balance coverage without slowing down builds. Fast feedback is prioritized. This testing mix provides confidence before deployment.

78. How do you handle CORS and security headers in a full stack setup?
Allowed origins and HTTP methods are configured carefully. Wildcards are avoided for authenticated APIs. CORS rules are applied at the gateway or backend level. Security headers are added to protect against common web attacks. All settings are reviewed carefully for production safety. This ensures secure communication.

79. How do you package and deploy a Java full stack app?
The application is packaged as a runnable JAR file. It is containerized using Docker for consistency across environments. CI/CD pipelines handle building, testing, and deployment. The application runs behind a reverse proxy such as Nginx. Configuration is externalized using environment variables. Secrets are managed securely.

80. differnce between speing and spring boot ?
spring (Spring Framework) is the core framework.
Spring Boot is built on top of Spring to reduce setup and run apps faster.

Main difference:

Spring:

You configure many things manually (beans, server setup, XML/Java config).
More control, more boilerplate.
Spring Boot:

Auto-configuration, starter dependencies, embedded server (Tomcat/Jetty), production features (Actuator).
Less boilerplate, faster development.
In short:
Spring = foundation.
Spring Boot = opinionated, faster way to build Spring apps.

