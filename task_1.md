# Task 1 Solution (C#)

## Goal

Enable the use of `FtpClient` for reading logs inside `LogImporter` **without modifying** `FtpClient`, `ILogReader`, or `LogImporter`. Only `LogImporterClient` and new files/interfaces can be edited or created.

---

## Solution Overview

We use the **Adapter Pattern** to make `FtpClient` compatible with the `ILogReader` interface. This approach allows `LogImporter` to remain untouched while gaining support for reading logs from FTP.

---

## New Class: `FtpLogReaderAdapter`

This class implements `ILogReader` and internally uses `FtpClient` to read the file.

```csharp
using Patterns.Ex00;

public class FtpLogReaderAdapter : ILogReader
{
    private readonly FtpClient _ftpClient;

    public FtpLogReaderAdapter(FtpClient ftpClient)
    {
        _ftpClient = ftpClient;
    }

    public string ReadLogFile(string identificator)
    {
        return _ftpClient.DownloadFile(identificator); // Assume this method exists
    }
}
```

---

## Modified Class: `LogImporterClient`

Only this file is changed to use the adapter.

```csharp
using Patterns.Ex00;

public class LogImporterClient
{
    public void DoMethod()
    {
        var ftpClient = new FtpClient();
        var adapter = new FtpLogReaderAdapter(ftpClient);
        var importer = new LogImporter(adapter);
        importer.ImportLogs("ftp://log.txt");
    }
}
```

---

## Why Adapter Pattern?

* `ILogReader` is the expected interface in `LogImporter`.
* `FtpClient` does not implement it and cannot be changed.
* We wrap `FtpClient` in an adapter that does implement `ILogReader`.

---

## Advantages

* Follows SOLID principles (Open/Closed).
* Easy to extend with other sources (e.g. HTTP reader).
* No violation of restrictions (existing classes untouched).

---

## Folder Structure

```
PatternsExercises/
├── Ex00/
│   ├── FtpLogReaderAdapter.cs       # New
│   ├── LogImporterClient.cs         # Modified
│   ├── FileLogReader.cs             # Unchanged
│   ├── LogImporter.cs               # Unchanged
│   ├── ILogReader.cs                # Unchanged
```

---

## Optional Extension

To support more protocols (HTTP, SFTP), simply create more adapters:

````csharp
public class HttpLogReaderAdapter : ILogReader
{ ... }

---

## Python Translation of the Solution

The Adapter Pattern can also be implemented in Python using composition.

### Interfaces and Base Classes
```python
from abc import ABC, abstractmethod

class ILogReader(ABC):
    @abstractmethod
    def read_log_file(self, identificator: str) -> str:
        pass
````

### Existing `FtpClient` (unchangeable)

```python
class FtpClient:
    def download_file(self, identificator: str) -> str:
        return "FILE_FROM_FTP"
```

### Adapter: `FtpLogReaderAdapter`

```python
class FtpLogReaderAdapter(ILogReader):
    def __init__(self, ftp_client: FtpClient):
        self._ftp_client = ftp_client

    def read_log_file(self, identificator: str) -> str:
        return self._ftp_client.download_file(identificator)
```

### `LogImporter` and Client Usage

```python
class LogImporter:
    def __init__(self, reader: ILogReader):
        self._reader = reader

    def import_logs(self, source: str):
        file = self._reader.read_log_file(source)
        print(f"Imported: {file}")

# Client code
ftp_client = FtpClient()
adapter = FtpLogReaderAdapter(ftp_client)
importer = LogImporter(adapter)
importer.import_logs("ftp://log.txt")
```

This Python version mirrors the C# approach and is suitable for projects requiring flexible and decoupled log import implementations.

```
```
