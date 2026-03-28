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



