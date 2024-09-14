## Memory Management in C#

Typically CPU memory is divided into two categories: Stack and Heap. Stack is most likely be in L1 and L2 cache. Stack's default size on Windows (both ARM and x86) is 1MB, whereas Linux has 8 MB.

### Reference vs Value Types

There are three C# keywords that you can use to define object types: **class**, **record**, and **struct**.

- Both **record** and **class** are defined a reference type. The objects are stored inside heap, and only the memory address referencing the objects are stored in the stack.
- The **record struct** or **struct** are defined a value type. The objects are stored in the stack memory.

Here are the most common struct types (MP):

- Number System: byte, sbyte, short, ushort, int, uint, long, ulong, float, double, and decimal
- Other System types: char, DateTime, DateOnly, TimeOnly, and bool
- System.Drawing types: Color, Point, PointF, Size, SizeF, Rectangle, and RectangleF

### Boxing

Boxing occurs when value type is moved to heap memory and wrapped inside System.Object instance. Unboxing occurs when value is moved back onto the stack. Boxing happens implicitly, whereas Unboxing occurs explictly. Boxing can take up 20 times longer than without Boxing.

### Garbage Collection

Once the program finish running, the stack memory is is released from top of the stack. However, Heap memory is not immediately to improve performance. In .NET destructor is called finalizer, and the syntax is the same as C++.
