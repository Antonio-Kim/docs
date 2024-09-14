## File System in C#

Here are some of the useful library and its method when using File System:

- Path
  - Path.PathSeparator
  - Path.DirectorySeparatorChar
  - Path.GetTempPath()
- Directory
  - Directory.GetCurrentDirectory()
- Environment
  - Environment.GetCurrentDirectory
  - Environment.SystemDirectory
- SpecialFolder(Use "GetFolderPath" method to retrieve)
  - SpecialFolder.System
  - SpecialFolder.ApplicationData
  - SpecialFolder.MyDocuments
  - SpecialFolder.Personal
- DriveInfo.GetDrives() - This retrieves all the infomation of the current available drives
  - DriveInfo.IsReady
  - Drive.Name
  - Drive.DriveType
  - Drive.DriveFormat
  - Drive.TotalSize
  - Drive.AvailableFreeSpace

When working with custom path, we must be careful with whether the folder exists or not. Do not make any assumptions about the custom path unless you check them out first. Here's an example of creating a folder

```csharp
string newFolder = Combine(GetFolderPath(SpecialFolder.Desktop), "NewFolder");
WriteLine($"Working with: {newFolder}");
WriteLine($"Does it exist? {Path.Exists(newFolder)}");

WriteLine("Creating it...");
CreateDirectory(newFolder);

WriteLine($"Does it exists? {Directory.Exists(newFolder)}");
Write("Confirm the directory exists, and then press any key.");
ReadKey(intercept: true);

WriteLine("Deleting it...");
Delete(newFolder, recursive: true);
WriteLine($"Does it exists? {Path.Exists(newFolder)}");
```

Now let's deal with files. Here's a similar example as above but with creating files.

```csharp
string dir = Combine(GetFolderPath(SpecialFolder.Desktop), "OutputFiles");
CreateDirectory(dir);

string textFile = Combine(dir, "Dummy.txt");
string backupFile = Combine(dir, "Dummy.bak");
WriteLine($"Working with: {textFile}");

WriteLine($"Does it exist? {File.Exists(textFile)}");

StreamWriter textWriter = File.CreateText(textFile);
textWriter.WriteLine("Hello, C#");
textWriter.Close();
WriteLine($"Does it exists? {File.Exists(textFile)}");

File.Copy(sourceFileName: textFile, destFileName: backupFile, overwrite: true);
WriteLine($"Does {backupFile}? {File.Exists(backupFile)}");
WriteLine("Confirm the files exists, and then press any key.");
ReadKey(intercept: true);

File.Delete(textFile);
WriteLine($"Does it exist? {File.Exists(textFile)}");

WriteLine($"Reading contents of {backupFile}:");
StreamReader textReader = File.OpenText(backupFile);
WriteLine(textReader.ReadToEnd());
textReader.Close();
```

### Controlling how to work with files

- The `File.Open` method has overloads to specify additional options using enum values
  - FileMode: controls what you want to do with the file, like CreateNew, OpenOrCreate, or Truncate
  - FileAccess: controls what level of acces you need, like ReadWrite
  - FileShare: controls locks on the file to allow other processes the specified level of access, like Read
  - FileAttribute: checks a FileSystemInfo-derived type's Attributes property for values like Archive and Encrypted

```csharp
FileStream file = File.Open(pathToFile, FileMode.Open, FileAccess.Read, FileShare.Read);
```
