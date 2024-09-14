## Classes in C#

- If a class has **virtual** method, the inherit class can override the method and create a new method using **overide** keyword. However, if the class has public method, then the inherit class needs to hide it by using **new** keyword instead. So for example,

```csharp
public class BaseClass {
	public void Display() {
		Console.WriteLine("Hello, world");
	}
}

public class DerivedClass : BaseClass {
	public new void Display() {
		Console.WriteLine("Hello from derived class");
	}
}
```
