---
title: Api Design
date: 2021-08-28 11:02:12
categories: "软件设计"
tags: "Api"
---

This article is conculsion about google api design share, it contains these content :

- The Process of Api Design
- General Principles
- Documentation Matters
- Classes
- Method Design
- Exception Design

<!--more-->

## Streaming Api

- When design streaming api, I think reliability is need to be aware of, and performance maybe not the highest priority. We need Streaming Api return the data in a timely and consistently way, maybe it's performance not be the best.
- When design streaming api, you need to considering about how to process the error, and the expansibility.
- If you process a large amount of data set, or an unkown size data set, you could choose the streaming way, cause it could process large amount of data with low memory and consistently. If you choose the NON-Streaming way, maybe it will being some memory trouble to you, and maybe it will cost a lot of time in process the data in memory, and the client will be block in waitting the response, even it will meet timeout issue.

## The Process of API Design

### start with short spec - 1 page is ideal

- at this stage, agility trumps completeness
- Bounce spec off as many people as possible
- if you keep the spec short, it's easy to modify
- Flesh it out as you gain confidence - this necessarily involves coding

### write your api early and often

- start before you've implemented the api - saves you doing implementation you'll throw away
- start before you've even specified it properly - saves you from writing specs you'll throw away
- continue writing to api as you flesh it out - prevents nasty surprise - code lives on as examples, unit tests

### writing to spi is even more important

- service provide interface(SPI) 
  - Plug-in interface enabling multiple implementations
  - Example: Java Cryptography Extension(JCE)
- write multiple plugins before release 
  - if one, it probably won't support another
  - if two, it will support more with diffculty
  - if three, it will work fine
- Will Tracz calls this "The Rule of Threes"

### maintain Realistic Expectations

- most API designs are over-constrained 
  - you won't be able to please everyone
  - Aim to displease everyone equally
- Expect to make mistakes 
  - A few years of real-word use will flush them out
  - Expect to evolve API

## General Principles

### API Should Do One Thing and Do it Well

- Functionality should be easy to explain 
  - if it's hard to name, that's generally a bad sign.(when you hard to explain how it does, or with multiple contrain , that probably means it is not reasonable)
  - Good names drive development
  - Be amenable to splitting and merging modules（make every module easy to describe）

### API Should Be As Small As Possible But No Smaller

- API should satisfy its requirements(but beyond that, more powerful is not better)
- When in doubt leave it out
  - Functionality, classes, methods, parameters, etc.
  - **You can always add, but you can never remove**
- **Conceptual weight** more important than bulk
- Look for a good power-to-weight ratio

### Implementation Should Not Impact API

- Implementation details 
  - Confuse users
  - Inhibit freedom to change implementation
- Be aware of what is an implementation detail 
  - Do not overspecify the behavior of methods
  - For example: do not specify hash functions
  - All tuning parameters are suspect
- Don't let implementation details "leak" into API 
  - on-disk and on-the-wire formats, exceptions(should warp your level exception or format, not just throw them into api.)

### Minimize Accessibility of Everything

- Make classes and members as private as posiible
- Public classes should have no public fields(with the exception of constrants)
- This maximaizes **information hiding(decouple modules from another)**
- Allows modules to be used, understood, built, tested, and debugged independently**.(so that you can improve one module  without effect the other module)**

### Names Matter — API is a Little Language

- Names Should Be Largely Selfi-Explanatory 
  - avoid cryptic abbreviations
- Be consistent-name word means same thing 
  - Throughout API,(Across APIs on the platform)
- Be regular-strive for symmetry
- Code should read like prose

```java
if (car.speed() > 2 * SPEED_LIMIT)
	generateAlert("Watch out for cops!");
```

### Documentation Matters

### Document Religiously

- Document every class, interface, method, constructor, parameter, and exception 
  - Class: what an instance represents
  - Method: contract between method and its client(preconditions, postconditions, side-effects)
  - Parameter: indicate units, form, ownership
- Document state space very carefully

### Consider Performance Consequences of API Design Decisions

- Bad decisions can limit performance 
  - Making type mutable
  - Providing constructor instead of static factory
  - Using implementation type instead of interface
- Do not warp API to gain performance 
  - Underlying performance issue will get fixed, but headaches will be with you forever
  - Good design usually coincides with good performance

### Effects of API Design Decisions on Performance are Real and Permanent

- **Component.getSize()** returns **Dimension Object(in awt?)**
- **Dimension is mutable**
- Each getSize call must allocate Dimension, and this  cause millions of needless object allocations
- Alternative added in 1.2; old client still slow

### API Must Coexist Peacefully with Platform(for me, it's java platform)

- Do what is customary 
  - Obey standard naming conventions
  - Avoid obsolete parameter and return types
  - Mimic patterns in core APIs and language(mimic the jdk pattern)
- Take advantage of API-friendly features 
  - Generics, varags, enums, default arguments
- Know and avoid API traps and pitfalls 
  - Finalizers, public static final arrays
- Don't Transliterate APIs
  - **just don't simply traslate c++ api to java, it will make java program seems like c++，and java programmer probably not willing to use them, at least not comfotable using it.**
  - take a step back, to see what the problem that api solved, what the extraction does is use, and what the platform provide, use the nature way in the target platform.

## Classes

### Minimize Mutability

- Classes should be immutable unless there's a good reason to do otherwise 
  - Advantages: simple, thread-safe, reusable
  - Disadvantages: separate object for each value
- If mutable, keep state-space small, well-defined 
  - Make clear when its legal to call which method

### Subclass Only Where It Makes Sense

- Subclassing implies substitutability(Liskov) 
  - Subclass only when is-a relationship exists(you should have one class extend another, only when you can say the instance  of first class is a instance of second class)
  - for example: you can say a set is a collection, but you cannot say the stack is a vector.(**stack extends vector,its just for implementation convenient,but its bad**)
  - Otherwise, use composition
- Public classes should not subclass other public classes for ease of implementation

### Design and Document for Inheritance or Else Prohibit it

- Inheritance violates encapsulation 
  - subclass sensitive to implementation details of superclass.
  - if a class not design for extend, then it shouldn't be extend by subclass. Otherwise, it will make subclass works bad in other release of superclass.
- if you allow subclassing(design for extend), document self-use 
  - how do methods use one another?
- Conservative policy: all concrete classes final 
  - if a class not design for extention, it should be final. then people will not able to extend it.
- Bad: Many concrete classes in J2SE libraries
- Good: AbstractSet, AbstractMap, all of these abstract implementation classes.its document allowed it easy to extention.

## Method Design

### Don't Make the Client Do Anything the Module Could Do

shouldn't call a method return values, and pass it the second method, then pass it to third method.

- Reduce need for boilerplate code

  - Generally done via cut-and-paste
  - Ugly, annoying, and error-prone

  ```java
  //Dom code to write an XML document to specified output stream.
  static final void writeDoc(Document doc, OutputStream out) throw IOException
  //the below code is bad, because it should design the api like above code shows.
  //The client needless to know how to setup a transformer,
  //it just need to pass a document and output stream
  try{
  	Transformer t = TransformerFactory.newInstance().newTransformer();
  	t.setOutputProperty(OutputKeys.DOCTYPES_SYSTEM, doc.getDocType().getSystemId());
  	t.tranform(new DomSource(doc), new StreamResult(out));
  } cathc (TransformerException e){
  // this error is can't happen, but it still a checked Exception.
  	throw new AssertinError(e); 
  }
  //**The API should easy to do the simple things, and possible to do complex things.**
  ```

### Don't Violate  Principle of Least Astonishment

- User of API should not be surprised by behavior

  - it's worth extra implementation effort
  - it's even worth reduced performence

  ```java
  public class Thread implements Runnable {
  	//Tests whether current thread has been interrupted.
  	//side-effect: Clears the interrupted status of current thread.
  	//if should be extract to another method called *clearInterruptStatus*
  	public static boolean interrupted();
  }
  ```

### Fail Fast - Report Errors as Soon as Possible After They Occur

- Compile time is best - it means you should use static typing, generics
- At runtime, first bad method invocation is best 
  - method should be failure-atomic

```java
//A properties instance maps strings to strings
public class Properties extends Hashtable {
	//the input key-value type is Object, so you can put any type of data in.
	//and then you call the save method, it will fail. But you will not know 
	//when the data is put in.Its a bad design. 
	public Object put(Object key, Object value);
	
	//Throws ClassCastException if this properties contains 
	//any keys or values that are not strings
	public void save(OutputStream out, String comments);
}
```

### Provide Programmatic Access to All Data Available in String Form

- Otherwise, clients will parse strings

  - Painful for clients
  - Worse, turns string format into de facto API

  ```java
  public class Throwable{
  	public void printStackTrace(PrintStream s);
  	public StackTraceElement[] getStackTrace();// since 1.4
  }
  public final class StackTraceElement{
  	public String getFileName();
  	public String getClassName();
  }
  ```

### Overload With Care

- Avoid ambiguous overloadings 
  - Multiple overloadings applicable to same actuals
  - Conservative: no two with same number of args
- Just because you can doesn't mean you should 
  - Often better to use a different name
- If you must provide ambiguous overloading, ensure same behavior for same arguments

```java
public TreeSet(Collection c);//ignores order
public TreeSet(SortedSet s);//respects order
//for this overload, you can pass the same collection, but it will confuse you
//sometimes it respect order, sometime not.
```

### Use Appropriate Parameter and Return Types

- Favor interface types over classes for input 
  - (To client, it)Provide flexibility, performance(because it allow client to use high performance implementation)
- Use most specific possible input parameter type 
  - Move error from runtime to compile time
- Don't use string if a better type exists 
  - Strings are cumbersome, error-prone, and slow
- Don't use floating point for monetary values 
  - Binary floating point causes inexact results!
- User **double(64 bits)** rather than **float(32 bits)**
  - Precision loss is real, performance loss negligible

### Use Consistent Parameter Ordering Across Methods

- Especially important if parameter types identical

```java
#include <String.h>
char *strcpy (char *dest, char *src);
void bcopy (void *src, void *dst, int n);
//if you call the two methods above with same parameter order, you will be confused.its a big mistake
java.util.Collections - frist parameter always collection to be modified or queried
java.util.concurrent - time always specified as *long delay, TimeUnit unit*
```

### Avoid Long Parameter Lists

- Three or fewer parameters is ideal 
  - More and users will have to refer to docs
- Long lists of identically typed params harmful 
  - Programmers transpose parameters by mistake
  - Programs still compile, run, but misbehave!
- Two techniques for shortening parameter lists 
  - Break up method(Like use build patten to build parameter)
  - Create helper class to hold parameters

### Avoid Return Values that Demand Exceptional Processing

anything that demand special check in the client, that client maybe forget to check, and it will be broke the code.

- Return zero-lenth array or empty collection, not null.

## Exception Design

### Throw Exceptions to Indicate Exception Conditions

- Don't force client to use exceptions for control flow

```java
private byte[] a = new byte[BUF_SIZE];
void processBuffer (ByteBuffer buf){
	try{
		while(true){
			buf.get(a);
			processBytes(tmp, BUF_SIZE);
		}
	} catch (BufferUnderflowException e){
			int remaining = buf.remaining();
			buf.get();
			processBytes(bufArray, remaining);
	}
}
```

- Conversely, don't faill silently

### Favor Unchecked Exceptions

- Checked - client must take recovery action
- Unchecked - programming error
- Overuse of checked exceptions causes boilerplate

### Include Failuer-Capture Information in Exceptions

- Allow diagonsis and repair or recovery
- For unchecked exceptions, message suffices
- For checked exceptions, provide accessors

## Refactoring API Designs

1. Sublist Operations in Vector

```java
public class Vector{
	public int indexOf(Object elem, int index);
	public int lastIndexOf(Object elem, int index);
}
```

- Not very powerful - supports only search
- Hard to use without documentation

Sublist Operations Refactored

```java
public interface List{
	List subList(int fromIndex, int toIndex);
}
```

- Extremely powerful - support all operation
- Use of interface reduces conceptual weight 
  - high power-to-weight ratio
- Easy to use without documentation

2. Thread-Local Variables

```java
//Broken - inappropriate use of String as capability
//Keys constitute a shared global namespace
public class ThreadLocal{
	private ThreadLocal(){}//Non-instantiable
	
	//set cuttent thread's value for named variable
	public static void set(String key, Object value);
		
	//Returns current thread's value of named variable
	public static Object get(String key);
}
```

Thread-Local Variables Refactored(1)

```java
public class ThreadLocal{
	private ThreadLocal(){}//Non-instantiable

	public static class Key { key(){} }	
	//Generate a unique, unforgeable key
	public static Key getKey(){return new Key();}

	public static void set(Key key, Object value);
	public static Object get(Key key);
}
```

- Works, but requires boilerplate code use(see: *Don't Make the Client Do Anything the Module Could Do*)

```java
static ThreadLocal.Key serialNumberKey = ThreadLocal.getKey();
ThreadLocal.set(serialNumberKey, nextSerialNumber());
System.out.println(ThreadLocal.get(serialNumberKey ));
```

look at this class, all method and inner class is static. Beside, *get* and *set* method accept a parameter with Key type. so that we can move *get* and *set* method into Key class.

And there are no other method in this class, so maybe we can move the Key class outside, then rename it to ThreadLocal

Thread-Local Variables Refactored(2)

```java
public class ThreadLocal{
	public ThreadLocal(){}
	public static void set(Object value);
	public static Object get();
}
```

- Removes clutter from API and client code

```java
static ThreadLocal serialNumberKey = new ThreadLocal();
serialNumberKey.set(nextSerialNumber());
System.out.println(serialNumberKey.get());
```

## Conclusion

- API design is a noble and rewarding craft 
  - Improve the lot of programmers, end-users, companies
- This talk covered some heuristics of the craft 
  - Don't adhere to them slavishly, but...
  - Don't violate them without good reason
- API design is tough 
  - Not a solitary activity
  - Perfection is unachievable, but try anyway