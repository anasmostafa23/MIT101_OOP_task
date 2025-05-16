# Task 3 - SubEx 01 Solution (C#)

## Goal

Without modifying `DatabaseSaver`, `MailSender`, or `CacheUpdater`, ensure that when `SaveData` is called:

1. A confirmation email is sent via `MailSender`
2. The cache is updated via `CacheUpdater`

Only `DatabaseSaverClient` and new code can be changed.

---

## Solution Overview

We will use the **Decorator Pattern** to wrap the existing `IDatabaseSaver` implementation with additional behavior (sending mail and updating cache).

---

## Step-by-Step Implementation

### Step 1: Create a Decorator

```csharp
using Patterns.Ex05.ExternalLibs;

public class DatabaseSaverWithPostActions : IDatabaseSaver
{
    private readonly IDatabaseSaver _inner;
    private readonly MailSender _mailSender;
    private readonly CacheUpdater _cacheUpdater;

    public DatabaseSaverWithPostActions(IDatabaseSaver inner, MailSender mailSender, CacheUpdater cacheUpdater)
    {
        _inner = inner;
        _mailSender = mailSender;
        _cacheUpdater = cacheUpdater;
    }

    public void SaveData(object data)
    {
        _inner.SaveData(data);
        _mailSender.Send("user@example.com");
        _cacheUpdater.UpdateCache();
    }
}
```

### Step 2: Modify `DatabaseSaverClient`

```csharp
using Patterns.Ex05.ExternalLibs;

namespace Patterns.Ex05.SubEx_01
{
    public class DatabaseSaverClient
    {
        public void Main(bool b)
        {
            var rawSaver = new DatabaseSaver();
            var decoratedSaver = new DatabaseSaverWithPostActions(
                rawSaver,
                new MailSender(),
                new CacheUpdater()
            );

            DoSmth(decoratedSaver);
        }

        private void DoSmth(IDatabaseSaver saver)
        {
            saver.SaveData(null);
        }
    }
}
```

---

## Benefits

* **Open/Closed Principle**: `DatabaseSaver` remains unchanged.
* New behavior added externally using composition.

---

## Folder Structure

```
PatternsExercises/
├── Ex05/
│   ├── ExternalLibs/
│   │   ├── DatabaseSaver.cs         # Unchanged
│   │   ├── MailSender.cs           # Unchanged
│   │   ├── CacheUpdater.cs         # Unchanged
│   │   ├── IDatabaseSaver.cs       # Interface used
│   ├── SubEx_01/
│   │   ├── DatabaseSaverClient.cs  # Modified
│   │   ├── DatabaseSaverWithPostActions.cs # New
```

---

## Optional Python Version

```python
class IDatabaseSaver:
    def save_data(self, data):
        raise NotImplementedError("This method should be overridden.")

class DatabaseSaver:
    def save_data(self, data):
        print("Saving data to the database")

class MailSender:
    def send(self, email):
        print(f"Sending email to {email}")

class CacheUpdater:
    def update_cache(self):
        print("Cache updated")

class DatabaseSaverWithPostActions(IDatabaseSaver):
    def __init__(self, saver, mail_sender, cache_updater):
        self.saver = saver
        self.mail_sender = mail_sender
        self.cache_updater = cache_updater

    def save_data(self, data):
        self.saver.save_data(data)
        self.mail_sender.send("user@example.com")
        self.cache_updater.update_cache()

# Usage Example:
if __name__ == "__main__":
    base_saver = DatabaseSaver()
    enhanced_saver = DatabaseSaverWithPostActions(base_saver, MailSender(), CacheUpdater())
    enhanced_saver.save_data({"key": "value"})
```
