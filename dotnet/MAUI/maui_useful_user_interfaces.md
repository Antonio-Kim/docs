## Useful (more common) user interfaces for XAML

Any of the functionality can be assigned a variable within the namespace of xaml. Unlike CS files, xaml uses xmlns:x, somewhat similar to using directive. You'll notice that each of the xaml files have the following:

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyMauiApp.MainPage">
```

Important! The xmlns attribute in XML stands for XML Namespace, which defines a scope for elements and attributes within the document. It functions akin to key-value dictionaries, where each namespace serves a specific purpose and functionality. In XAML, the xmlns attribute is crucial for defining XML namespaces. The first xmlns declaration, xmlns="http://schemas.microsoft.com/dotnet/2021/maui", sets the default namespace to use .NET MAUI controls throughout the document. The second xmlns:x declaration, xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml", introduces a namespace prefix x that references schema documentation for XAML. This namespace enables the usage of XAML-specific functionality defined by Microsoft. The attribute x:Class="MyMauiApp.MainPage" within the XAML file specifies that the MainPage class in the MyMauiApp namespace is associated with the ContentPage, linking the XAML UI with its corresponding code-behind logic.

### Layout

- <VerticalStackLayout>
  - Spacing
  - Padding

### Items

- Label
  - Text
  - FontSize
- Entry: Input box
  - Text: Default text when displayed
- Button
  - Text
  - IsEnabled: disable functionality initially
  - Clicked: event handler attribute that will use function defined in .cs
