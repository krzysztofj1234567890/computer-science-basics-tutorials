# Python 3

## Python basics

__Python Indentation__: Python uses indentation to indicate a block of code:
```
if 5 > 2:
  print("Five is greater than two!")
```

P__ython Variables:__ Python has no command for declaring a variable. Variable names are case-sensitive.
``` 
x = 5
y = "Hello, World!"

# assign many values to many variables
x, y, z = "Orange", "Banana", "Cherry"

# One Value to Multiple Variables
x = y = z = "Orange"

# Unpack a Collection
fruits = ["apple", "banana", "cherry"]
x, y, z = fruits
print(x)
```

__Print Text__
```
print("Hello World!")

# same line
print("Hello World!", end=" ")

# print number
print(3)

# mixed
print("I am", 35, "years old.")
```

__Casting__: specify the data type of a variable
```
x = str(3)    # x will be '3'
y = int(3)    # y will be 3
z = float(3)  # z will be 3.0 

# Get the Type
print(type(x))
```

__global__: Normally, when you create a variable inside a function, that variable is local, and can only be used inside that function. To create a global variable inside a function, you can use the global keyword 
```
# If you use the global keyword, the variable belongs to the global scope:
def myfunc():
  global x
  x = "fantastic"
myfunc()
print("Python is " + x) 

# use the global keyword if you want to change a global variable inside a function.
x = "awesome"
def myfunc():
  global x
  x = "fantastic"
myfunc()
print("Python is " + x) 
```

__Data Types__
```
Text Type: 	        str
Numeric Types: 	    int, float, complex
Sequence Types: 	list, tuple, range
Mapping Type: 	    dict
Set Types: 	        set, frozenset
Boolean Type: 	    bool
Binary Types: 	    bytes, bytearray, memoryview
None Type: 	        NoneType
```

__In Python 3, the integer type (int) has no maximum limit; it can grow as large as the available memory allows.__

maximum float value in Python is approximately 1.7976931348623157e+308

```
# Print the data type of the variable x:
x = 5
print(type(x))

x = "Hello World"                   //	str 	
x = 20 	                            // int 	
x = 20.5 	                        // float 	
x = 1j 	                            // complex 	
x = ["apple", "banana", "cherry"] 	// list 	
x = ("apple", "banana", "cherry") 	// tuple 	
x = range(6) 	                    // range 	
x = {"name" : "John", "age" : 36} 	// dict 	
x = {"apple", "banana", "cherry"} 	// set 	
x = frozenset({"apple", "banana", "cherry"}) 	//frozenset 	
x = True 	                        // bool 	
x = b"Hello" 	                    // bytes 	
x = bytearray(5) 	                // bytearray 	
x = memoryview(bytes(5)) 	        // memoryview 	
x = None
```

__Random Number__
```
# display a random number from 1 to 9
import random
print(random.randrange(1, 10))
```

__Strings__
```
# Multiline Strings: You can use three double quotes:
a = """Lorem ipsum dolor sit amet,
consectetur adipiscing elit,
sed do eiusmod tempor incididunt
ut labore et dolore magna aliqua."""
print(a) 

# Strings are Arrays
a = "Hello, World!"
print(a[1])

# Loop through the letters in the word "banana":
for x in "banana":
  print(x)

# len
a = "Hello, World!"
print(len(a))

# Check String
txt = "The best things in life are free!"
if "free" in txt:
  print("Yes, 'free' is present.")

# Get the characters from position 2 to position 5 (not included):
b = "Hello, World!"
print(b[2:5])

# Use negative indexes to start the slice from the end of the string: 
b = "Hello, World!"
print(b[-5:-2])

# upper
a = "Hello, World!"
print(a.upper())

# Remove Whitespace
a = " Hello, World! "
print(a.strip())

# replace
a = "Hello, World!"
print(a.replace("H", "J"))

# split
a = "Hello, World!"
print(a.split(",")) # returns ['Hello', ' World!'] 

# formatting
age = 36
txt = f"My name is John, I am {age}"
print(txt)
```

__Operators__
```
# Walrus Operator: assigns values to variables as part of a larger expression
numbers = [1, 2, 3, 4, 5]
if (count := len(numbers)) > 3:
    print(f"List has {count} elements")

# Chaining Comparison Operators
x = 5
print(1 < x < 10)

# Identity Operators

# is: Returns True if both variables are the same object
x = ["apple", "banana"]
y = ["apple", "banana"]
z = x

print(x is z)   // True
print(x is y)   // False

# ==  Checks if the values of both variables are equal
print(x == y)   // True

# Membership Operators

# in: Returns True if a sequence with the specified value is present in the object
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)      // True         
```

__Lists__
```
mylist = ["apple", "banana", "cherry"]

# List items can be of any data type:
list1 = ["abc", 34, True, 40, "male"]

# lists are defined as objects with the data type 'list'
mylist = ["apple", "banana", "cherry"]
print(type(mylist))       // <class 'list'>

# Print the second item of the list:
thislist = ["apple", "banana", "cherry"]
print(thislist[1])

# Change the second item:
thislist = ["apple", "banana", "cherry"]
thislist[1] = "blackcurrant"
print(thislist)

# Change the second value by replacing it with two new values
thislist = ["apple", "banana", "cherry"]
thislist[1:2] = ["blackcurrant", "watermelon"]
print(thislist)                       // ['apple', 'blackcurrant', 'watermelon', 'cherry']

# Append Items
thislist = ["apple", "banana", "cherry"]
thislist.append("orange")
print(thislist)

# Insert Items
thislist = ["apple", "banana", "cherry"]
thislist.insert(1, "orange")
print(thislist)                     // ['apple', 'orange', 'banana', 'cherry']

# Remove the first occurrence of "banana":
thislist = ["apple", "banana", "cherry", "banana", "kiwi"]
thislist.remove("banana")
print(thislist)

# Loop Through a List
thislist = ["apple", "banana", "cherry"]
for x in thislist:
  print(x)

# List Comprehension
thislist = ["apple", "banana", "cherry"]
[print(x) for x in thislist] 

# Comprehention
fruits = ["apple", "banana", "cherry", "kiwi", "mango"]
newlist = [x for x in fruits if "a" in x]
print(newlist)                    // ['apple', 'banana', 'mango']

# Sort the list alphabetically:
thislist = ["orange", "mango", "kiwi", "pineapple", "banana"]
thislist.sort()
print(thislist)

# Sort the list based on how close the number is to 50:
def myfunc(n):
  return abs(n - 50)
thislist = [100, 50, 65, 82, 23]
thislist.sort(key = myfunc)
print(thislist)

# Join two list:
list1 = ["a", "b", "c"]
list2 = [1, 2, 3]
list3 = list1 + list2
print(list3)                      // ['a', 'b', 'c', 1, 2, 3]
```

__Tuples__: are used to store multiple items in a single variable.

Tuple items are __ordered, unchangeable__, and allow duplicate values.

```
mytuple = ("apple", "banana", "cherry")

# Print the second item in the tuple:
thistuple = ("apple", "banana", "cherry")
print(thistuple[1])

# Convert the tuple into a list to be able to change it:
x = ("apple", "banana", "cherry")
y = list(x)
y[1] = "kiwi"
x = tuple(y)
print(x)                        // ("apple", "kiwi", "cherry")
```

__Sets__: used to store multiple items in a single variable.

A set is a collection which is __unordered, unchangeable, and unindexed__.
Sets __cannot have duplicates__.
Once a set is created, you cannot change its items, __but you can add new items__.

```
set1 = {"abc", 34, True, 40, "male"} 

# Loop through the set, and print the values:
thisset = {"apple", "banana", "cherry"}
for x in thisset:
  print(x) 

# Add an item to a set, using the add() method:
thisset = {"apple", "banana", "cherry"}
thisset.add("orange")
print(thisset)                    // {'orange', 'apple', 'cherry', 'banana'} 

# Add elements of a list to at set:
thisset = {"apple", "banana", "cherry"}
mylist = ["kiwi", "orange"]
thisset.update(mylist)
print(thisset)                    // {'banana', 'cherry', 'apple', 'orange', 'kiwi'}

# Remove "banana" by using the remove() method:
thisset = {"apple", "banana", "cherry"}
thisset.discard("banana")
print(thisset)                    // {'apple', 'cherry'}

# loops
thisset = {"apple", "banana", "cherry"}
for x in thisset:
  print(x) 

# frozenset is an immutable version of a set.

```

__Dictionaries__: store data values in key:value pairs

A dictionary is a collection which is __ordered*, changeable and do not allow duplicates__.

```
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
print(thisdict["brand"])

# Duplicate values will overwrite existing values:
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964,
  "year": 2020
}
print(thisdict) 

# The values in dictionary items can be of any data type:
thisdict = {
  "brand": "Ford",
  "electric": False,
  "year": 1964,
  "colors": ["red", "white", "blue"]
} 

# Get the value of the "model" key:
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
x = thisdict["model"]


```



