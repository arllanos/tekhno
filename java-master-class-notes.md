# Java Programming notes

## Java primitives types
### Whole numbers
| type | width |
|--|--|
| byte | 8 |
| short | 16 |
| int | 32 |
| long | 64 |

### Decimal numbers (floating point/real numbers)
| type | width | notes |
|--|--|--|
| float | 32 | single precision |
| double | 64 | double precision |
| char | 16 | allow single byte and unicode chars as well |
| boolean | 8 | true or false |

### Reference Types vs Value Types
All the primitive types are value types, that is, they hold a value. Unlike primitive types, an array is a reference type and String is also a reference type. 

`int[] myIntArray = new int[5];` in this case `myIntArray` is a reference to the object and not the object itself. In other words `myIntArray` holds a reference to an array in memory


### String
The String is a datatype in Java, is not a primitive type. 
It's actually a Class, but it enjoys a bit of favoritism in Java to make it easier to use than a regular class.
A String can contain a sequence of characters. A large number of characters. Technically it's limited by memory or the MAX_VALUE of an int which was 2.14 Billion. That's a lot of characters.

```java
String myString = "This is a string";
myString = myString + ", and this is more";
myString = myString + " \u00A9 2019";
```
### Integer
Java uses the concept of Wrapper class for all eight primitives types - In the case of an int, we can use Integer, and by doing that it gives us ways to perform operation on an int

Integer Minimum Value = -2,147,483,648
Integer Maximum Value = 2,147,483,647

### Static vs Instance Methods
#### Static Methods
- **static methods** are declared using the **static** modifier
- **static methods** can't access instance methods and instance variables directly
- the are usually used for operations that don't require any data from an instance of the class (this)
- whenever you see a method that **does not use instance variables** that method should be declared as a static method
- static methods are called as **ClassName.methodName();** or as **methodName();** only if in the same class

```java
class Calculator {
	public static void printSum(int a, int b) {
		System.out.printl("sum=" + (a+b));
	}
}
public class Main {
	public static void main(String[] args) {
		Calculator.printSum(5, 10);
		printHello(); // shorter form of Main.printHello();
	}
	public static void printHello() {
		System.out.println("hello");
	}
}
```

#### Instance Methods
- **instance methods** belongs to an instance of a class
- to use an **instance method** we have to instantiate the class, usually by calling the **new** keyword
- **instance methods** can access instance methods and instance variables directly
- **instance methods** can also access static methods and static variables directly


### Static vs Instance Variables
#### Static Variables
- declared by using the keyword **static**
- every instance of that class **shared the same static variables**
- if changes are made to that variable, all other instances will see the effect of the change
- not used very often but can be very useful
- for example when reading user input using **Scanner** we will declare scanner as a static variable
- that way **static methods** can access it directly

#### Instance Variables
- they **don't** use the **static** keyword.
- instance variables are also known as **fields** or member variables.
- they belong to an instance of a class.
- every instance has it's own copy of an instance variable.
- every instance can have a different value (state).
- instance variables represent the state of an instance.

#### Access Modifiers
Access we are allowing to have others to the new class we are creating
- public: unrestricted access.
- private: no other class can access that class.
- protected: which allows classes in this package to access your class.

### Arrays
Data structure that allows you to store sequence of values that are all of the same type. e.g., array of integers, array of characters, array of strings, etc, that is, this will work for all primitive types and even objects like strings.
Arrays have a set size that you determine when create it.
The default values of numeric array elements are set to **zero**.
Append is easy and achieved in O(1) but insert (in the middle is expensive and is achieved in O(k)
Arrays are **zero indexed**: an array with **n elements** is indexed **from 0 to n-1**
If we try to access index that is out of range Java will give an **ArrayIndexOutOfBoundsException**, which indicates that the index is out of range, i.e., out of bounds.
```java
int[] array; // declare  
array = new int[5]; //initialize, where 10 is the number of slots
```
or
```java
int[] array = new int[5];
```
or 
```java
int[] array = {1,2,3,4,5};
```
* To **print** an array turn it to string `Arrays.toString(reverse(myArray))`
* To **copy** an array use `Arrays.copyOf(original, newLength)`
* To **resize** an array ... too tedious!
	```java
	int[] original = baseData; // baseData is the array to resize and has 10 elements
	baseData = new int[12];
	for (int i = 0; i < original.length; i++)
		baseData[i] = original[i];
	```

## Java Collections Framework (Part 1)
### List
Interface that extends the collection interface. Implemented by ArrayList, LinkedList, Stack, Vector. 
A list is an **ordered collection**. The user of this interface has precise control over where in the list each element is inserted. The user can access elements by their integer index (position in the list), and search for elements in the list. 
Unlike sets, lists typically allow duplicate elements.

### ArrayList
ArrayList is a resizeable array. It inherits from the list. It holds objects. 
```java
private ArrayList<String> groceryList = new ArrayList<string>();
```
Add Item
```java
groceryList.add(item)
```
Get an item
```java
groceList.get(i)
```
Modify an item
```java
groceyList.set(position, newItem);
```
Remove an item
```java
groceryList.remove(position);
```
Get the size
```java
groceryList.size();
```
Compare two lists
```java
list1.equals(list2);
```
 Query the list to find an item
```java
boolean exists = groceryList.contains(searchItem);
int position = groceryList.indexOf(searchItem);
if (position >=0) {
	item = groceyList.get(position);
} else {
	item = null;
}
```
### LinkedList
Alternative to ArrayList. LinkedList stores the actual link to the next item in the list as well as the actual data. In other words, each element in the list, holds a link to the item that follows it, as well as the actual value. 
This kind of data structure provides better performance for adding new elements or removing elements in the middle of the array. For example in an array when an elements is added or removed in a position between existing elements, internally Java will need to move the elements for either, make room for adding the new element (when adding) or moving up all elements after the removed element when removing.
```java
LinkedList<String> placesToVisit = new LinkedList<String>();
placesToVist.add("Sydney");
placesToVist.add("Melbourne");
placesToVist.add("Brisbane");
placesToVist.add("Perth");
placesToVist.add("Canberra");
placesToVist.add("Adelaide");
placesToVist.add("Darwin");

Iterator<String> i = LinkedList.iterator();
while(i.hasNext() {
	System.out.println("Now visiting " + i.next());
}
// add at specific index position
// note the same can be done in arrayList but would be less time efficient
placesToVisit.add(1, "Alice Springs");

// remove at position 4
placesToVisti.remove(4);
```
#### Generics
The idea is to allow **type** (Integer, String, ... and user defined types) to be a parameter to methods, classes and interfaces. For example, classes like HashSet, ArrayList, HashMap, etc use generics.
ArrayList for example, is a generic type and it can be used without a type parameters as in:
```java
ArrayList items = new ArrayList();
```
or specifying a type as in:
```java
ArrayList<int> items = new ArrayList<>();
```
## Java Collections Framework (Part 2)
The collection framework includes not only List interface, the array and the linked lists classes. It also includes Sets, Maps, Trees, Queues.
At the top level of the collections framework is the **Collections** class. It exposes static methods that you can operate on collections, such as, sort() and other methods for all the fundamental operations that are required for the various collection types.
The framework provides methods that enable collections to be moved into arrays and vice versa and additionally methods to allow arrays to be viewed as collections.
The core elements of the collections framework are interfaces (abstract types that represent collections)

#### HashMap
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
```
Add an item
```java
map.put(myKey, myValue);
```
Get an item
```java
map.get(myKey); // return myValue
```
Contains
```java
if (map.ContainsKey(myKey) {
	System.out.println("Key exists in the map");
}
```
Remove an item
```java
int retVal = map.remove(myKey); // retVal will contains myVal if myKey is in map, else null
```

## OOP
A class is a blueprint (model) for an object that we're creating
A class has fields (for the state of the objects). These better to be private. 
Example Method to update objects' fields:
```java
	private String name;
	
	public void setName(String name) {
		String validName = name.toLowerCase();
		if (validName.equals("carrera") || validName.equals("commodore")) {
			this.name = name;
		} else {
			this.name= "Unknown";
		}
	}

	public String getName() {
		return this.name;
	}
```
Example classes modeling **composition** relationship, that is, a Monitor has Resolution
Resolution.java (is part of relationship)
```java
public class Resolution {
	private int width;
	private int height;
	
	public Resolution(int width, int height) {
		this.width = width;
		this.height = height;
	}

	public int getWidth() {
		return width;
	}

	public getHeight() {
		return height;
	}
}
```
Monitor.java
```java
public class Monitor {
	private String model;
	private String manufacturer;
	private int size;
	private Resolution nativeResolution;

	public Monitor(String model, string manufacturer, int size, Resolution nativeResolution)
	{
		this.model = model;
		// ...
	}
	
	// getters ...
}
```
