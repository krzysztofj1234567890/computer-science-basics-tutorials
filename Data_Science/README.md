# DATA SCIENCE with python

In a world of data space where organizations deal with petabytes and exabytes of data, the era of Big Data emerged.
Data science is a field that involves using statistical and computational techniques to extract insights and knowledge from data.

It encompasses a wide range of tasks, including data cleaning and preparation, data visualization, statistical modeling, machine learning, and more. Data scientists use these techniques to discover patterns and trends in data, make predictions, and support decision-making.

Python has been considered the preferred choice among data scientists.

## Python Pandas

### install pandas

Open Visual Studio Code.
In terminal run the following command:

```
pip install pandas
```

### DataFrame

DataFrame is two-dimensional size-mutable, potentially heterogeneous tabular data structure with labeled axes (rows and columns):

| id  | name  | phone      | age  |   
|---- | ----- | ---------- | ---- |
| 1   | Chris | 1234567890 | 40   |   <- row 1
| 2   | Tom   | 1234567891 | 10   |   <- row 2
| 3   | Kate  | 1234567892 | 11   |   <- row 3
| 4   | Mom   | 1234567893 | NULL |

columns: id, name, phone, age

In the real world, a Pandas DataFrame will be created by loading the datasets from existing storage, storage can be SQL Database, CSV file, and Excel file.

Dataframe can be created in different ways here are some ways by which we create a dataframe:

```
# import pandas as pd
import pandas as pd
 
# list of strings
lst = ['Geeks', 'For', 'Geeks', 'is', 'portal', 'for', 'Geeks']
 
# Calling DataFrame constructor on list
df = pd.DataFrame(lst)
print(df)
```

Dataframe with column titles:

```
# Import pandas package
import pandas as pd
 
# Define a dictionary containing employee data
data = {'Name':['Jai', 'Princi', 'Gaurav', 'Anuj'],
        'Age':[27, 24, 22, 32],
        'Address':['Delhi', 'Kanpur', 'Allahabad', 'Kannauj'],
        'Qualification':['Msc', 'MA', 'MCA', 'Phd']}
 
# Convert the dictionary into DataFrame 
df = pd.DataFrame(data)
print(df)
```

Select columns:

```
# select two columns
print(df[['Name', 'Qualification']])
```

Read data from file and select a data item

```
data = pd.read_csv("/home/kj/Krzys/git/computer-science-basics-tutorials/Data_Science/nba.csv", index_col ="Name")
first = data.loc["Avery Bradley"]
second = data.loc["R.J. Hunter"]
print(first, "\n\n", second)
```

## Python Numpy

Numpy is a general-purpose array-processing package. It provides a high-performance multidimensional array object, and tools for working with these arrays.

Array in Numpy is a table of elements (usually numbers), all of the same type, indexed by a tuple of positive integers. In Numpy, number of dimensions of the array is called rank of the array.A tuple of integers giving the size of the array along each dimension is known as shape of the array. An array class in Numpy is called as ndarray.

### Creating a Numpy Array

```
import numpy as np
 
# Creating a rank 1 Array
print( "---- Creating a rank 1 Array" )
arr = np.array([1, 2, 3])
print("Array with Rank 1: \n",arr)
 
# Creating a rank 2 Array
print( "\n---- Creating a rank 2 Array" )
arr = np.array([[1, 2, 3],
                [4, 5, 6]])
print("Array with Rank 2: \n", arr)
 
# Creating an array from tuple
print( "\n---- Creating an array from tuple" )
arr = np.array((1, 3, 2))
print("\nArray created using passed tuple:\n", arr)
```

### Basic Array Operations
```
# Defining Array 1
print( "\n---- Defining Arrays 1 and 2" )
a = np.array([[1, 2],
              [3, 4]])
 
# Defining Array 2
b = np.array([[4, 3],
              [2, 1]])
               
# Adding 1 to every element
print( "\n---- Adding 1 to every element" )
print ("\nAdding 1 to every element:", a + 1)
 
# Subtracting 2 from each element
print( "\n---- Subtracting 2 from each element" )
print ("\nSubtracting 2 from each element:", b - 2)
 
# sum of array elements Performing Unary operations
print( "\n---- sum of array elements Performing Unary operations" )
print ("\nSum of all array elements: ", a.sum())
 
# Adding two arrays
print( "\n---- adding two arrays" )
print ("\nArray sum:\n", a + b)

# Square root of Array
Sqrt = np.sqrt(a)
print("\nSquare root of Array1 elements: ")
print(Sqrt)
 
# Transpose of Array using In-built function 'T'
Trans_arr = a.T
print("\nTranspose of Array: ")
print(Trans_arr)
```

## References

https://www.geeksforgeeks.org/data-science-tutorial/
