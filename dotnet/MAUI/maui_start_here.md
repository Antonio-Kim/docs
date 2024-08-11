## .NET MAUI Project structure and application startup

.NET MAUI uses .NET Generic Host for app startup and configuration. Generic Host includes the following:

- Dependency Injection (DI)
- Logging
- Configuration
- Lifecycle management

### Important files and folders

- **Resources**: folder with shared fonts, images, and assets used by all platforms.
- **MauiProgram.cs**: configures the app and specifies that the App class should be used to run the application.
- **The App.xaml.cs**: the constructor for the App class creates a new instance of the AppShell class, which is then displayed in the application window. This is where UI resources are maintained, but also Application Life Cycles.
- **AppShell.xaml**: file that contains the main layout for the application and starting page of MainPage. Both AppShell files handle the registration of pages for navigational routing
- **MainPage.xaml**: file that contains the layout for the page.
- **MainPage.xaml.cs**: file contains the application logic for the page.

### Purpose of each important files

- **App.xaml**: defines all the resources that the app will use, using XAML file format
- **App.xaml.cs**: this is where application is defined at run time. App.xaml.cs also deals with app's lifecycle, which can also override each platform's life-cycle event handlers. The MainPage property in App.xaml.cs determines which page is displayed when App starts running
- **AppShell.xaml**: sets the main layout of the app. Provides features like app styling, URI based navigation and layout options
- **MainPage.xaml**: defines a page with UI definition and structure. This is a single page, which means other pages can be defined with another xaml.
- **MainPage.xaml.cs**: the code/behaviour defined in the MainPage.xaml. Deals with logic and various event handlers triggered by the controls of the MainPage.xaml.
- **MauiProgram.xaml.cs**: Platform-specific definition of the app are done here.

In summary, the `App` files are definition of App enviroment at runtime `AppShell` file defines main structure/navigator of the App, and `MainPage` is just a page of an App.

#### csproj file

- PropertyGroup: defines platform frameworks that project targets, as well as metadata about the app.
- ItemGroup: specify image and color for splash screen when the app is loading.
