## Streams in C#

**Stream**: Sequence of bytes that you can read from and written to. Although it's possible to access the data like array in random fashion, it's more efficient to access them in sequential order. Stream data are used in cases where data are typically accessed once. Streams are used when your runtime program does not have control over - unmanaged resources - things like Files, Network, and Database. Note that Stream specifically work with Bytes. The process of converting objects into bytes/json/xml, and bytes/json/xml into streams are called Serialization and Deserialization.

In C# there's abstract class named **Stream** but many different concrete classes inherit the abstract class like, FileStream, MemoryStream, BufferedStream, GZipStream, and many others. Here are some of **Stream** members that are frequently used:

| Member            | Description                                                                                                                                                                                                          |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CanRead, CanWrite | These properties determine if you can read fomr and write to the stream                                                                                                                                              |
| Length, Position  | These properties determine the total number of bytes and the current position within the stream. These properties may throw a NotSupportedException for some types of streams, for example, if CanSeek returns false |
| Close, Dispose    | This method closes the stream and releases its resources. You can call either method since the implementation of Dispose calls Close                                                                                 |
| Flush             | If the stream has a buffer, then this method writes the bytes in the buffer to the stream, and the buffer is cleared.                                                                                                |
| CanSeek           | This propertyu determines if the Seek method can be used                                                                                                                                                             |
| Seek              | This method moves the current position to the one specified in its parameter                                                                                                                                         |
| Read, ReadAsync   | These methods read a specified number of bytes from the stream into a byte array and advance the position                                                                                                            |
| ReadByte          | This method reads the next byte from the stream and advances the position.                                                                                                                                           |
| Write, WriteAsync | These methods write the contents of a byte array into the stream.                                                                                                                                                    |
| WriteByte         | This method writes a byte to the stream                                                                                                                                                                              |

### Stream Categories

There are three types of stream categories that are often used in C#: **storage**, **function** and **stream helpers**.

#### Storage Stream

This stream represents location where the bytes will be stored.

| Namespace          | Class         | Description                                   |
| ------------------ | ------------- | --------------------------------------------- |
| System.IO          | FileStream    | Bytes stored in the filesystem                |
| System.IO          | MemoryStream  | Bytes stored in memory in the current process |
| System.Net.Sockets | NetworkStream | Bytes stored at a network location            |

#### Function Stream

Function streams cannot exists alone, but can be plugged into other stream to add functionalities

| Namespace                    | Classes                   | Description                              |
| ---------------------------- | ------------------------- | ---------------------------------------- |
| System.Security.Cryptography | CryptoStream              | This encrypts and decrypts the stream.   |
| System.IO.Compression        | GZipStream, DeflateStream | These compress and decompress the stream |
| Systen.Net.Security          | AuthenticatedStream       | This sends credentials across the stream |

#### Stream Helpers

Streams are done on lower level like bytes, but you can add additional helpers to help deal with with other types of data.

| Namespace  | Class        | Description                                                                                                                                                                                                        |
| ---------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| System.IO  | StreamReader | This reads from the underlying stream as plain text                                                                                                                                                                |
| System.IO  | StreamWriter | this writes to the underlying stream as plain text                                                                                                                                                                 |
| System.IO  | BinaryReader | This reads from streams as .NET types. For example, the ReadDecimal method reads the next 16 bytes from the underlying stream as a decimal value, and the ReadInt32 method reads the next 4 bytes as an int value. |
| System.IO  | BinaryWriter | This writes to streams as .NET types. Form example, the Write method with a decimal parameter writes 16 bytes to the underlying stream, and the Write method with an int parameter writes 4 bytes.                 |
| System.Xml | XmlReader    | This reads from the underlying stream using the XML format                                                                                                                                                         |
| System.Xml | XmlWriter    | This writes to the underlying stream using the XML format                                                                                                                                                          |

#### How does these stream implemented?

Typically streams are created into a pipeline where you joined multiple types of stream, and typically ends with a storage stream. For example, it is very common to combien a helper like StreamWriter and multiple function streams like CryptoStream and GZipStream with a storage stream like FileStream into a pipeline.

### Example: Writing to Text Stream

Here's a simple example of writing a text file.

```csharp
string textFile = Combine(CurrentDirectory, "coffees.txt");
StreamWriter text = File.CreateText(textFile);

string[] coffees = new[]
{
	"Starbucks", "Tim Hortons", "Second Cup", "McDonald's", "Costco"
};

foreach (string coffee in coffees)
{
	text.WriteLine(coffee);
}

text.Close();
```

Common way to implement any type of streams is to use `using` statement inside the method or the code that's being implement. This will call Dispose() function to ensure that stream has been removed from memory.

```csharp
using (FileStream file2 = FileOpenWrite(Path.Combine(path, "file2.txt")))
{
	using (StreamWrite writer2 = new StreamWrite(file2))
	{
		try
		{
			writer2.WriteLine("Welcome to .NET"!);
		}
		catch (Exception ex)
		{
			WriteLine($"{ex.GetType()} says {ex.Message}");
		}
	}
}
```

### Example: Combining Compression and XML

XML file are typically larger than normal text file and one way to reduce the size is to use compression. There are many types of compression available in .NET, but the example below will only deal with gzip and brotli compression. Note that there's two ways to invoke stream, and both of them are shown below as well.

```csharp
private static void Compress(string algorithm = "gzip")
{
	string filePath = Combine(CurrentDirectory, $"stream.{algorithm}");
	FileStream file = File.Create(filePath);

	Stream compressor;
	if (algorithm == "gzip")
	{
		compressor = new GZipStream(file, CompressionMode.Compress);
	}
	else
	{
		compressor = new BrotliStream(file, CompressionMode.Compress);
	}

	using (compressor)
	{
		using (XmlWriter xml = XmlWriter.Create(compressor))
		{
			xml.WriteStartDocument();
			xml.WriteStartElement("callsigns");
			foreach (string item in Viper.CallSigns)
			{
				xml.WriteElementString("callsign", item);
			}
		}
	}
	OutputFileInfo(filePath);

	WriteLine("Reading the compressed XML file:");
	file = File.Open(filePath, FileMode.Open);
	Stream decompressor;
	if (algorithm == "gzip")
	{
		decompressor = new GZipStream(file, CompressionMode.Decompress);
	}
	else
	{
		decompressor = new BrotliStream(file, CompressionMode.Decompress);
	}

	using (decompressor)
	using (XmlReader reader = XmlReader.Create(decompressor))

	while (reader.Read())
	{
		if ((reader.NodeType == XmlNodeType.Element) && (reader.Name == "callsign"))
		{
			reader.Read();
			WriteLine($"{reader.Value}");
		}
	}
}
```
