## .NET MAUI Project structure and application startup

### Important files and folders

- **Resources**: folder with shared fonts, images, and assets used by all platforms.
- **MauiProgram.cs**: configures the app and specifies that the App class should be used to run the application.
- **The App.xaml.cs**: the constructor for the App class creates a new instance of the AppShell class, which is then displayed in the application window. This is where UI resources are maintained, but also Application Life Cycles.
- **AppShell.xaml**: file that contains the main layout for the application and starting page of MainPage. Both AppShell files handle the registration of pages for navigational routing
- **MainPage.xaml**: file that contains the layout for the page.
- **MainPage.xaml.cs**: file contains the application logic for the page.
