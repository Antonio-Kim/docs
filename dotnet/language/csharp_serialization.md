## Serialization in C#

**Object Graph** is a structure of multiple objects that are related to each other, either through a direct reference or indirectly through a chain of references

**Serialization** is process converting a live object graph into a sequence of bytes using a specified format. **Deserialization** is the reverse process.

Note that as mentioned above, serialization and deserialzation is the process of converting live object graph into a format, such as bytes, json, and xml. Streams are storage location that can handle these format, although it doesn't mean it only handles serialized data.

### Example: JSON serialization and deserialization

NB: we add the following using statements inside the project file:

```xml
<ItemGroup>
    <Using Include="System.Console" static="true" />
    <Using Include="System.Environment" static="true" />
    <Using Include="System.IO.Path" static="true" />
</ItemGroup>
```

and add the following using statement in the program.cs:

```csharp
using System.Xml.Serialization;
using FastJson = System.Text.Json.JsonSerializer;
```

Let's start with a class that has some fields inside. Here's a class of a Person

```csharp
public class Person
{
	#region fields
	[XmlAttribute("fname")]
	public string? FirstName { get; set; }
	[XmlAttribute("lname")]
	public string? LastName { get; set; }
	[XmlAttribute("dob")]
	public DateTime DateOfBirth { get; set; }
	public HashSet<Person>? Children { get; set; }
	protected decimal Salary { get; set; }
	#endregion

	public Person(decimal initialSalary)
	{
		Salary = initialSalary;
	}
	public Person()
	{

	}
}
```

We then have people data structures like this:

```csharp
List<Person> people = new()
{
	new (initialSalary: 30_000M)
	{
		FirstName = "Alice",
		LastName = "Smith",
		DateOfBirth = new(year: 1974, month: 3, day: 14)
	},
	new (initialSalary: 40_000M)
	{
		FirstName = "Bob",
		LastName = "Jones",
		DateOfBirth = new(year: 1969, month: 11, day: 23)
	},
	new (initialSalary: 20_000M)
	{
		FirstName = "Charlie",
		LastName = "Cox",
		DateOfBirth = new(year: 1984, month: 5, day: 4),
		Children = new()
		{
			new(initialSalary: 0M)
			{
				FirstName = "Sally",
				LastName = "Cox",
				DateOfBirth = new(year: 2012, month:7, day: 12)
			}
		}
	}
};
```

Serializing JSON file is simple as this:

```csharp
string jsonPath = Combine(CurrentDirectory, "people.json");
using (StreamWriter jsonStream = File.CreateText(jsonPath))
{
	Newtonsoft.Json.JsonSerializer jss = new();
	jss.Serialize(jsonStream, people);
}
```

Deserialization, however, requires little bit more effort:

```csharp
await using (FileStream jsonLoad = File.Open(jsonPath, FileMode.Open))
{
	List<Person>? loadedPeople = await FastJson.DeserializeAsync(utf8Json: jsonLoad, returnType: typeof(List<Person>)) as List<Person>;

	if (loadedPeople is not null)
	{
		foreach (Person p in loadedPeople)
		{
			WriteLine("{o} has {1} children", p.LastName, p.Children?.Count ?? 0);
		}
	}
}
```

Note that you have option to format the JSON output. You'll need to add option into Serialize method of JSONSerializer function. Some of the options are shown below:

```csharp
JsonSerializerOptions options = new()
{
	IncludeFields = true,
	PropertyNameCaseInsensitive = true,
	WriteIndented = true,
	PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
};
```

Then add the `option` into `jss.Serialize(jsonStream, people, option)` for different format.
