## C# Concepts and implementation

This section discusses basic and advance concepts and implementation of C# language and libraries. There's not exactly a flow to this section, but merely snippets of ideas that I have come across that might find me useful to understand. The source of the snippets are as follows:

- C# 12 and .NET 8 by Mark Price (MP)
- C# in Depth by Jon Skeet (JS)
- C# Interview Guide by Konstantin Semenenko (KS)
- C# Data Structures and Algorithms by Marcin Jamro (MJ)

Note that some of the writings will have parentheses with Initial of the author's name, plus the page number for reference.

### Data Types

- Multi-Dimensional Array vs Jagged array

  - The declaration of multi-dimensional array is different in C# compared to other languages. For example, the 2-dimensional array in C# is declared with a comma

  ```csharp
  // declaration only
  int[,] numbers = new int[2,2];
  // with assignments
  int[,] numbers2 = new int[,]
  {
    {1,2},
    {3,4}
  }

  // Output 4
  Console.WriteLine(numbers2[1,1]);
  ```

  The Jagged array means that each row can have different number of columns, unlike regular arrays which requires same number of columns each row. This declaration looks similar to other languages

  ```csharp
  int[][] numbers = new int[4][];
  numbers[0] = new int[]{ 9, 5};
  numbers[1] = new int[] {0,-3,12};
  numbers[3] = [54];
  int[][] numbers2 =
  {
    [9,5],
    [0, -3, 12],
    null!,
    [5]
  }
  ```

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

- **Record struct** vs **struct**: record struct has all the benefits of struct, but record struct is not immutable unless readonly is applied. Record struct also implements == and != for you, whereas struct does not.

### String methods

Here are some important String methods in C#:

| Member                    | Description                                                                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trim, TrimStart, TrimEnd  | These methods trim whitespace characters such as space, tab, and carriage return from the start and/or end                                                                                               |
| ToUpper, ToLower          | These convert all the characters into uppercase or lowercase.                                                                                                                                            |
| Insert, Remove            | These methods insert or remove some text.                                                                                                                                                                |
| Replace                   | This replaces some text with other text.                                                                                                                                                                 |
| string.Empty              | This can be used instead of allocating memory each time you use a literal string value using an empty pair of double quotes ("").                                                                        |
| string.Concat             | This concatenates two string varibles. The + operator does the equivalent when used between string operands.                                                                                             |
| string.Join               | This concatenates one or more string variables with a character in between each one                                                                                                                      |
| string.IsNullOrEmpty      | This checks whether a string variable is null or empty                                                                                                                                                   |
| string.IsNullOrWhiteSpace | this checks whether a string variable is null or whitespace; that is, a mix of any number of horizontal and veriical spacing characters, for example, tab, space, carriage return, line feed, and so on. |

### Regular Expressions

- IsMatch is the most common method to use regular expression. Here are some of the syntax and quantifier used in regex:

| Symbol      | Meaning                | Symbol   | Meaning                    |
| ----------- | ---------------------- | -------- | -------------------------- |
| ^           | Start of input         | $        | End of input               |
| \d          | A single digit         | \D       | A single non-digit         |
| \s          | Whitespace             | \S       | non-whitespace             |
| \w          | word characters        | \W       | Non-word characters        |
| [A-Za-z0-9] | Range(s) of characters | \^       | (caret) characters         |
| [aeiou]     | Set of characters      | [^aeiou] | Not in a set of characters |
| .           | Any single character   | \.       | .(dot) character           |

Here are the quantifier

| Symbol | Meaning        | Symbol | Meaning       |
| ------ | -------------- | ------ | ------------- |
| +      | One or more    | ?      | One or none   |
| {3}    | Exactly three  | {3,5}  | Three to five |
| {3,}   | At least three | {,3}   | Up to three   |

In order to use Regex, you will need to import `using System.Text.RegularExpression`. Here are three main methods to use Regex in C#:

1. **Regex.Match**: this is used to match a string. The signature is `Match <variable> = Regex.Match(_InputStr_, _Pattern_, _RegexOptions_)`. Here's an example:

```csharp
string pattern = @"([a-zA-Z]+) (\d+)";
Match result = Regex.Match("June 24", pattern);
```

2. **Regex.Matches**: We can do global search over the whole input and have multiple corresponding data instead of just one, using `Regex.Matches`.

```csharp
string pattern = @"([a-zA-Z]+) (\d+)";
MatchCollection matches = Regex.Matches("June 24, August 9, Dec 12", pattern);
Console.WriteLine($"{matches.Count} matches");
```

3. **Regex.Replace**: Similar to Match but instead of search we replace the string instead.

```csharp
string pattern = @"([a-zA-Z]+) (\d+)";
string replacedString = Regex.Replace("June 24, August 9, Dec 12", pattern, @"$2 of $1");
Console.WriteLine(replacedString);
```

All the methods have Regex Option in the signature. Here are some of the common options:

- RegexOptions.Compiled
- RegexOptions.IgnoreCase
- RegexOptions.Multiline
- RegexOptions.RightToLeft
- RegextOptions.SingleLine

### Useful Interfaces

- **IComparable**: used to allow sorting between same classes' object. Needs to implement _CompareTo_ method which returns either -1 if the object needs to move in front of the object being compared, 1 if the object needs to move behind the object being compared, and 0 if they are the same. (MP)
- **IComparer**: comparsion method that a secondary type implements to order or sort instance of a primary type. Sometimes it might not be possible to access the source code for a type, nor it might not implement that IComparable interface. (MP)

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
