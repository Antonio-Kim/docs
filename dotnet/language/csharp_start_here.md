## C# Concepts and implementation

This section discusses basic and advance concepts and implementation of C# language and libraries. There's not exactly a flow to this section, but merely snippets of ideas that I have come across that might find me useful to understand. The source of the snippets are as follows:

- C# in Depth by Jon Skeet (JS)
- C Interview Guide by Konstantin Semenenko (KS)
- C# Data Structures and Algorithms by Marcin Jamro (MJ)

Note that some of the writings will have parentheses with Initial of the author's name, plus the page number for reference.

### Data Types

#### Value Types

- Value Tuples - allows you to group multiple data elements of specific types

```csharp
(int, int, double) result = Calculate(4, 8, 13);
// (int min, int max, double avg) result = Calculate(4,8,13); - Another way to destructo Value tuples
Console.WriteLine($"Min = {result.Item1} / max = {result.Item2} / Avg = {result.Item3}");

public static (int, int, double) Calculate(params int[] numbers) {
  if (numbers.Length == 0) { return (0,0,0);}
  int min = int.MaxValue;
  int max = int.MinValue;
  int sum = 0;
  foreach (int number in numbers) {
    if  (number > max) { max = number; }
    if  (number < min) { min = number; }
    sum += number;
  }
  return (min, max, (double)sum / numbers.Length);
}
```

### Collections

- **IEnumerate** is used to read over collections and allows iteration of the objects. Useful for just reading values inside the collection
- **ICollection** on the other hand extends IEnumerate and allows adding, removing and checking existence of elements in the collection (KS, p60)
- Here are some useful collection types that are frequently used.
  - List<T>
  - Dictionary<K, V>
  - HashSet<T>
  - Queue<T>
  - Stack<T>

### LINQ

- **WHERE** vs **SELECT**: Use Where when you want to filter out the collections you are targeting, and use select when you want to modify the elements of the collection (KS, p61)
- **ALL**: Checks if every element in the collection satisfies particular condition. Use ALL when you need all the elements to meet specific criterion. If the collection is empty, it returns true since no elements violate the condition.
- **ANY**: Checks at least one of the element satisfies the condition. If the collection is empty, it returns false because there's no element that meets the criteria.
- **FirstOrDefault**: Returns the first element that matches the condition, or first element if no condition is specified. If there's no element that meets the condition, it returns default value
- **SingleOrDefault**: returns the only element that matches the condition, but throws exception if there are more than one element that matches the condition. If no element matches the condition, it returns default value. (KS, p.61)

### Exceptions

- Here are some common Exceptions:
  - ArgumentNullException
  - ArgumentOutOfRangeException
  - DivideByZeroException
  - InvalidOperationException
  - FileNotFoundException
  - StackOverflowException
  - NullReferenceException

### Testing

- **Assert** vs **Throw**: Assert is more about validating code logic during development, whereas throw is about handling exxceptional runtime scenarios

### Asynchronous Programming

- **Multithreading** vs **Asynchronous**: multithreading involves multiple threads running concurrently in a single process. Asynchronous programming focuses on allowing programs to continue executing without waiting for to finish. It usually invovles event loop. (KS, p70)
- Pitfalls of async/await
  - Deadlock: occurs when one asynchronous code is waiting for the other asynchronous code, but the other asynchronous code is waiting for the one asynchronous code
  - Thread starvation: situation can arise when over-relying on the thread pool and all the threads are consumed, causing delays in processing.
  - Performance overheads
  - Debugging complexity
- Task, Task<T>, and ValueTask<T>
  - **Task**: asynchronous operation that doesn't return a value. Esentially a promise that some work will be completed in the future
  - **Task<T>**: asynchronous operation that returns a value of type T
  - **ValueTask<T>**: result of asynchronous code might be available synchronously, potentially avoiding heap allocation.
