## XAML Syntax

Before discussing XAML in .NET MAUI, here are some important definitions:

- **Element**: start and end tag of that element together
- **Tag**: refers to either start or end of the elemnt
- **Node**: element and all its inner content, including child elements

In .NET the root element is usually either _Application_, _ContentPage_, _Shell_ or _ResourceDictionary_. Each element can have opening and closing tag, or self-closing tag if there are no children.

Every XAML file starts with XML declaration

```xml
<?xml version="1.0" encoding="utf-8">
```

Each elements can have attributes that alters the view property. In XAML, an element represents C# class and the attributes represents the members of that class.

### XAML namespace

Namespaces help to group elements and attributes to avoid name conflicts when the same name is used in a different scope. Namespace are usually defined as follow:

```xml
xmlns:prefix="identifier"
```

It consists of two components: prefix, and identifier. If prefix is omitted the whole file uses the identifier as the default namespace. There are usually two namespaces that are used in most of the
xaml file: .NET MAUI and XAML. .NET MAU namespace is usually the default namespace for most of the file:

```
xmlns="http://schemas.microsoft.com/dotnet/2021/mau"
```

The other namespace is the xaml and it has the x prefix

```
xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
```

the xmlns:x declaration specifies elements and attributes that are instrinsic to XAML, specific to 2009 XAML specification. This namespace is commonly used for UI designs in .NET MAUI. The xmlns:x can further
have constructs that modifies each view:

| Constructor     | Description                                                                                                                                                                                                          |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| x:Arguments     | Specifies constructor arguments for a non-default constructor, or for a factory method object declaration.                                                                                                           |
| x:Class         | Specifies the namespace and class name for a class defined in XAML. The class name must match the class name of the code-behind file. Note that this construct can only appear in the root element of a XAML file    |
| x:ClassModifier | Specifies the access level for the generated class in the assembly.                                                                                                                                                  |
| x:DataType      | Specifies the type of the object that the XAML element, and it's children, will bind to.                                                                                                                             |
| x:FactoryMethod | Specifies a factory method that can be used to initialize an object                                                                                                                                                  |
| x:FieldModifier | Specifies the access level for generated fields for named XAML elements.                                                                                                                                             |
| x:Key           | Specifies a unique user-defined key for each resource in a ResourceDictionary. The key's value is used to retrieve the XAML resource, and is typically used as the argument for the StaticResource markup extension. |
| x:Name          | Specifies a runtime object name for the XAML element. Setting x:Name is similar to declaring a variable in code.                                                                                                     |
| x:TypeArguments | Specifies the generic type arguments to the constructor of a generic type.                                                                                                                                           |

---

#### References

- https://learn.microsoft.com/en-us/dotnet/maui/xaml/namespaces/?view=net-maui-8.0
