
---

# SubEx 02 — Approach 1: Multiple Decorators

## Idea

* Create small decorator classes each adding a single extra behavior after `SaveData`.
* Wrap `DatabaseSaver` by stacking decorators.
* Easily add/remove extra behaviors by adding/removing decorators.

---

### Step 1: Define basic decorator base (optional)

```csharp
using Patterns.Ex05.ExternalLibs;

public abstract class DatabaseSaverDecorator : IDatabaseSaver
{
    protected readonly IDatabaseSaver _inner;

    protected DatabaseSaverDecorator(IDatabaseSaver inner)
    {
        _inner = inner;
    }

    public virtual void SaveData(object data)
    {
        _inner.SaveData(data);
    }
}
```

---

### Step 2: Implement MailSender decorator

```csharp
public class MailSenderDecorator : DatabaseSaverDecorator
{
    private readonly MailSender _mailSender;

    public MailSenderDecorator(IDatabaseSaver inner, MailSender mailSender) : base(inner)
    {
        _mailSender = mailSender;
    }

    public override void SaveData(object data)
    {
        base.SaveData(data);
        _mailSender.Send("user@example.com");
    }
}
```

---

### Step 3: Implement CacheUpdater decorator

```csharp
public class CacheUpdaterDecorator : DatabaseSaverDecorator
{
    private readonly CacheUpdater _cacheUpdater;

    public CacheUpdaterDecorator(IDatabaseSaver inner, CacheUpdater cacheUpdater) : base(inner)
    {
        _cacheUpdater = cacheUpdater;
    }

    public override void SaveData(object data)
    {
        base.SaveData(data);
        _cacheUpdater.UpdateCache();
    }
}
```

---

### Step 4: Modify `DatabaseSaverClient`

```csharp
using Patterns.Ex05.ExternalLibs;

namespace Patterns.Ex05.SubEx_02
{
    public class DatabaseSaverClient
    {
        public void Main(bool b)
        {
            IDatabaseSaver saver = new DatabaseSaver();
            saver = new MailSenderDecorator(saver, new MailSender());
            saver = new CacheUpdaterDecorator(saver, new CacheUpdater());

            DoSmth(saver);
        }

        private void DoSmth(IDatabaseSaver saver)
        {
            saver.SaveData(null);
        }
    }
}
```

---

### Benefits

* You can easily add more decorators without changing `DatabaseSaver`.
* Clear separation of concerns.
* Composable and extendable.

--- 


# SubEx 02 — Approach 2: Observer Pattern

## Idea

* Create a **notifier interface** for post-save actions.
* `DatabaseSaver` is wrapped in a class that calls `SaveData` and **notifies all registered observers**.
* Observers implement a common interface and subscribe/unsubscribe dynamically.
* Adding/removing observers doesn’t require changing `DatabaseSaver`.

---

### Step 1: Define observer interface

```csharp
public interface ISaveDataObserver
{
    void OnDataSaved(object data);
}
```

---

### Step 2: Create observer implementations

```csharp
using Patterns.Ex05.ExternalLibs;

public class MailSenderObserver : ISaveDataObserver
{
    private readonly MailSender _mailSender;

    public MailSenderObserver(MailSender mailSender)
    {
        _mailSender = mailSender;
    }

    public void OnDataSaved(object data)
    {
        _mailSender.Send("user@example.com");
    }
}

public class CacheUpdaterObserver : ISaveDataObserver
{
    private readonly CacheUpdater _cacheUpdater;

    public CacheUpdaterObserver(CacheUpdater cacheUpdater)
    {
        _cacheUpdater = cacheUpdater;
    }

    public void OnDataSaved(object data)
    {
        _cacheUpdater.UpdateCache();
    }
}
```

---

### Step 3: Create a notifier wrapper for `IDatabaseSaver`

```csharp
using System.Collections.Generic;
using Patterns.Ex05.ExternalLibs;

public class DatabaseSaverNotifier : IDatabaseSaver
{
    private readonly IDatabaseSaver _inner;
    private readonly List<ISaveDataObserver> _observers = new();

    public DatabaseSaverNotifier(IDatabaseSaver inner)
    {
        _inner = inner;
    }

    public void RegisterObserver(ISaveDataObserver observer)
    {
        _observers.Add(observer);
    }

    public void UnregisterObserver(ISaveDataObserver observer)
    {
        _observers.Remove(observer);
    }

    public void SaveData(object data)
    {
        _inner.SaveData(data);
        foreach (var observer in _observers)
        {
            observer.OnDataSaved(data);
        }
    }
}
```

---

### Step 4: Modify `DatabaseSaverClient`

```csharp
using Patterns.Ex05.ExternalLibs;

namespace Patterns.Ex05.SubEx_02
{
    public class DatabaseSaverClient
    {
        public void Main(bool b)
        {
            var rawSaver = new DatabaseSaver();
            var notifier = new DatabaseSaverNotifier(rawSaver);

            notifier.RegisterObserver(new MailSenderObserver(new MailSender()));
            notifier.RegisterObserver(new CacheUpdaterObserver(new CacheUpdater()));

            DoSmth(notifier);
        }

        private void DoSmth(IDatabaseSaver saver)
        {
            saver.SaveData(null);
        }
    }
}
```

---

### Benefits

* Adding/removing observers is easy and doesn’t affect `DatabaseSaver`.
* Follows **Open/Closed Principle**.
* Supports many observers dynamically.

---

## Approach 1: Multiple Decorators (Python)

```python
from abc import ABC, abstractmethod

class IDatabaseSaver(ABC):
    @abstractmethod
    def save_data(self, data):
        pass

class DatabaseSaver(IDatabaseSaver):
    def save_data(self, data):
        print("Saving data to database")

class MailSender:
    def send(self, email):
        print(f"Sending email to {email}")

class CacheUpdater:
    def update_cache(self):
        print("Cache updated")

class DatabaseSaverDecorator(IDatabaseSaver):
    def __init__(self, saver):
        self._saver = saver

    def save_data(self, data):
        self._saver.save_data(data)

class MailSenderDecorator(DatabaseSaverDecorator):
    def __init__(self, saver, mail_sender):
        super().__init__(saver)
        self._mail_sender = mail_sender

    def save_data(self, data):
        super().save_data(data)
        self._mail_sender.send("user@example.com")

class CacheUpdaterDecorator(DatabaseSaverDecorator):
    def __init__(self, saver, cache_updater):
        super().__init__(saver)
        self._cache_updater = cache_updater

    def save_data(self, data):
        super().save_data(data)
        self._cache_updater.update_cache()

# Usage example:
if __name__ == "__main__":
    base_saver = DatabaseSaver()
    saver_with_mail = MailSenderDecorator(base_saver, MailSender())
    saver_with_cache_and_mail = CacheUpdaterDecorator(saver_with_mail, CacheUpdater())
    saver_with_cache_and_mail.save_data({"key": "value"})
```

---

## Approach 2: Observer Pattern (Python)

```python
from abc import ABC, abstractmethod

class IDatabaseSaver(ABC):
    @abstractmethod
    def save_data(self, data):
        pass

class DatabaseSaver(IDatabaseSaver):
    def save_data(self, data):
        print("Saving data to database")

class ISaveDataObserver(ABC):
    @abstractmethod
    def on_data_saved(self, data):
        pass

class MailSender:
    def send(self, email):
        print(f"Sending email to {email}")

class CacheUpdater:
    def update_cache(self):
        print("Cache updated")

class MailSenderObserver(ISaveDataObserver):
    def __init__(self, mail_sender):
        self._mail_sender = mail_sender

    def on_data_saved(self, data):
        self._mail_sender.send("user@example.com")

class CacheUpdaterObserver(ISaveDataObserver):
    def __init__(self, cache_updater):
        self._cache_updater = cache_updater

    def on_data_saved(self, data):
        self._cache_updater.update_cache()

class DatabaseSaverNotifier(IDatabaseSaver):
    def __init__(self, saver):
        self._saver = saver
        self._observers = []

    def register_observer(self, observer):
        self._observers.append(observer)

    def unregister_observer(self, observer):
        self._observers.remove(observer)

    def save_data(self, data):
        self._saver.save_data(data)
        for observer in self._observers:
            observer.on_data_saved(data)

# Usage example:
if __name__ == "__main__":
    base_saver = DatabaseSaver()
    notifier = DatabaseSaverNotifier(base_saver)
    notifier.register_observer(MailSenderObserver(MailSender()))
    notifier.register_observer(CacheUpdaterObserver(CacheUpdater()))
    notifier.save_data({"key": "value"})
```

---


