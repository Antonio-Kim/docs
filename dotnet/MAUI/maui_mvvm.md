## .NET MAUI Views

MVVM separates the user interface logic, data, and app's structure into three distinct components: Model, View, and ViewModel. Model is similar to what you find in typical ASP.NET apps.
**Views** are individual UI element that user interact while using the app. Typical UI/Views are _Label_, _Button_, _Entry_, and _Image_. See the reference at the end of the document for more info on other Views available.
Data binding is a technique that connects the view with the ViewModel in a way that the UI reflects the ViewModel's data and changes synchronously and automatically.

### Data binding

Data binding is used to synchronize the properties of target and source objects. There are three objects involved in data binding and they are the binding target, binding source, and binding object. View and ViewModels are connected to synchronize data, and each contains specific properties for this. In View, there are Binding Target, which wraps around BindableObject, which also wraps Target Property. Similarly, ViewModel contains Binding Source, which wraps around Object, which then wraps around Source Property. The target property and source property are connected via Binding Object.

The object of binding class (Microsoft.Maui.Controls.Binding) represents the connection between the target property and source property. It manages the synchrnoization of property values between View and ViewModels and also handles communication between the two. The binding declaration happens inside the XAML View:

```xml
<Lable Text="{Binding FirstName}">
```

The **Binding target** is the attribute of an UI element, such as _Text_ on Label, _Source_ on Image, _IsEnabled_ on Button. The Target property must be bindable property to participate in data binding. The source is the object containing the property that is bound to the target property in the view.

- **Target**: UI element invovled in data binding and is a child of BindableObject.
- **Target Property**: Property/attribute of target object, and must be _BindableProperty_. If target is _Label_, then target property is something like **Label**.
- **Source**: Source object referenced by data binding. Usually a ViewModel class
- **Source object value path**: Path to value in the source object. The Path values are typically ViewModel Property, such as Name or Property.

Here's an example of Source Object:

```csharp
[QueryProperty(nameof(ItemId), nameof(ItemId))]
public class ItemDetailViewModel : BaseViewModel {
	private string itemId;
	private string name;
	private string description;
	public string Id { get; set; }
	public string Name {
		get => name;
		set => SetProperty(ref name, value);
	}
	public string Description...
	public string ItemId...
}
```

Here's a quick summary of data binding components:

| Data binding elements    | Examples            |
| ------------------------ | ------------------- |
| Target                   | Label               |
| Target Property          | Text                |
| Source Object            | ItemDetailViewModel |
| Source object value path | Name or Description |

Binding object typically follows the syntax below:

```xml
<object property="{Binding bindProp1=value[, bindPropN=valueN]*}" ... />
```

#### Binding mode

There are four ways that target and source share data between:

1. **OneWay**: used for presenting data from source
2. **TwoWay**: changes in either view or viewmodel affects other and automatically updates
3. **OneWayToSource**: Reverser of OneWay where the view updates the source.
4. **OneTime**: Target is initialized from source but only when it's initialized.

---

- https://learn.microsoft.com/en-us/dotnet/maui/user-interface/controls/?view=net-maui-8.0#views
