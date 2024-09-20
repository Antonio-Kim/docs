## Data Structures in C#

### ArrayList (not to be confused with ArrayList in Java)

ArrayList is dynamic array of objects that can be extended if needed be. Because it can add object types, it can add mixed types inside the ArrayList. It is recommended that List collection is used over ArrayList since List is strongly typed.

Here are some of the methods used in ArrayList:

- Adding
  - Add(value)
  - AddRange(IEnumerable)
  - Insert(value, index)
- Reading
  - Contains(value);
  - IndexOf(int)
- Deleting
  - Remove(value)
  - RemoveAt(index)
  - RemoveRange(index, numberOfRemoval);
  - Clear()
- Properties
  - Count;
  - Capacity;

### Generic List

Here are the properties and methods used in List<T>:

- Properties
  - Count;
  - Capacity;
- Adding
  - Add(T)
  - AddRange(IEnumerable<T>)
  - Insert(int32, T)
  - InsertRange(Int32, IEnumerable<T>)
- Reading
  - Contains(T)
  - IndexOf(T)
  - LastIndexOf(T)
- Deleting
  - Remove(T)
  - RemoveAt(Int32)
  - RemoveRange(Int32, Int32)
  - Clear()
- Methods
  - Reverse()
  - ToArray()
  - Min
  - Max
  - Sum
  - Average
  - All : checks whether all elements in the list satisfies a condition
  - Any: checks whether at least one element in the list satisfies a condition
  - ElementAtOrDefault: return element at the given index or returns default value if index is out of bounds
- NB: You will need to add .ToList() at the end since the methods return IEnumerable<T>
  - Distinct: returns collection with unique elements
  - OrderBy / OrderByDescending: return elements in the list in ascending or descending order
  - Skip(): skip the first given amount
  - Take(): take the first given amount

### Linked List

C# provides its own implementation of Doubly Linked List, LinkedList<T>. Here are some of the useful properties and methods:

- Adding
  - AddFirst
  - AddLast
  - AddBefore
  - AddAfter
- Properties
  - Contains
- Deleting
  - Remove
  - Clear
