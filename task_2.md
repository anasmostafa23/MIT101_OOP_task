# Task 2 Solution (C#)

## Goal

Implement `GetSubscribers` in `SubscriberViewer` to work with different social networks **without modifying existing classes**, including:

* `InstagramClient`, `TwitterClient`
* `InstagramUser`, `TwitterUser`
* `SocialNetworkUser`

Only `SubscriberViewer` and new classes/interfaces may be added.

---

## Solution Overview

Use the **Strategy Pattern** to encapsulate subscriber-fetching logic per social network in separate strategy classes. The `SubscriberViewer` dynamically delegates to the correct strategy based on the `SocialNetwork` enum.

---

## Step-by-Step Breakdown

### Step 1: Define a common strategy interface

```csharp
using Patterns.Ex01;

public interface ISocialNetworkSubscriberStrategy
{
    SocialNetworkUser[] GetSubscribers(string userName);
}
```

### Step 2: Implement Twitter strategy

```csharp
using Patterns.Ex01;
using Patterns.Ex01.ExternalLibs.Twitter;
using System.Linq;

public class TwitterSubscriberStrategy : ISocialNetworkSubscriberStrategy
{
    public SocialNetworkUser[] GetSubscribers(string userName)
    {
        var client = new TwitterClient();
        var userId = client.GetUserIdByName(userName);
        var twitterUsers = client.GetSubscribers(userId);

        return twitterUsers
            .Select(u => new SocialNetworkUser
            {
                UserName = client.GetUserNameById(u.UserId)
            })
            .ToArray();
    }
}
```

### Step 3: Implement Instagram strategy

```csharp
using Patterns.Ex01;
using Patterns.Ex01.ExternalLibs.Instagram;
using System.Linq;

public class InstagramSubscriberStrategy : ISocialNetworkSubscriberStrategy
{
    public SocialNetworkUser[] GetSubscribers(string userName)
    {
        var client = new InstagramClient();
        var instaUsers = client.GetSubscribers(userName);

        return instaUsers
            .Select(u => new SocialNetworkUser
            {
                UserName = u.UserName
            })
            .ToArray();
    }
}
```

### Step 4: Update `SubscriberViewer` to use strategies

```csharp
using System;
using System.Collections.Generic;

namespace Patterns.Ex01
{
    public class SubscriberViewer
    {
        private readonly Dictionary<SocialNetwork, ISocialNetworkSubscriberStrategy> _strategies;

        public SubscriberViewer()
        {
            _strategies = new Dictionary<SocialNetwork, ISocialNetworkSubscriberStrategy>
            {
                { SocialNetwork.Twitter, new TwitterSubscriberStrategy() },
                { SocialNetwork.Instagram, new InstagramSubscriberStrategy() }
            };
        }

        public SocialNetworkUser[] GetSubscribers(string userName, SocialNetwork networkType)
        {
            return _strategies[networkType].GetSubscribers(userName);
        }
    }
}
```

---

## Why Strategy Pattern?

* Supports adding new platforms (e.g. VK, LinkedIn) without touching existing logic.
* Isolates API-specific code.
* Promotes Open/Closed Principle.

---

## Folder Structure

```
PatternsExercises/
├── Ex01/
│   ├── ISocialNetworkSubscriberStrategy.cs      # New
│   ├── TwitterSubscriberStrategy.cs             # New
│   ├── InstagramSubscriberStrategy.cs           # New
│   ├── SubscriberViewer.cs                      # Modified
```

---

## Optional Extension

Use a factory or dependency injection container to register strategies automatically.

```csharp
public class SubscriberStrategyFactory
{
    public static ISocialNetworkSubscriberStrategy Create(SocialNetwork type) { ... }
}
```

---

## Python Translation of the Solution

Below is a Python equivalent using the Strategy Pattern:

### Step 1: Define interface

```python
from abc import ABC, abstractmethod
from typing import List

class SocialNetworkUser:
    def __init__(self, user_name: str):
        self.UserName = user_name

class SocialNetworkSubscriberStrategy(ABC):
    @abstractmethod
    def get_subscribers(self, user_name: str) -> List[SocialNetworkUser]:
        pass
```

### Step 2: Twitter strategy (mocked)

```python
class TwitterClient:
    def get_user_id_by_name(self, name): return 1
    def get_user_name_by_id(self, id): return "twitter_user"
    def get_subscribers(self, id): return [ {"UserId": 1} ]

class TwitterSubscriberStrategy(SocialNetworkSubscriberStrategy):
    def get_subscribers(self, user_name: str):
        client = TwitterClient()
        user_id = client.get_user_id_by_name(user_name)
        twitter_users = client.get_subscribers(user_id)
        return [SocialNetworkUser(client.get_user_name_by_id(u["UserId"])) for u in twitter_users]
```

### Step 3: Instagram strategy (mocked)

```python
class InstagramClient:
    def get_subscribers(self, user_name): return [ {"UserName": "insta_user"} ]

class InstagramSubscriberStrategy(SocialNetworkSubscriberStrategy):
    def get_subscribers(self, user_name: str):
        client = InstagramClient()
        insta_users = client.get_subscribers(user_name)
        return [SocialNetworkUser(u["UserName"]) for u in insta_users]
```

### Step 4: Viewer class

```python
from enum import Enum

class SocialNetwork(Enum):
    Instagram = 1
    Twitter = 2

class SubscriberViewer:
    def __init__(self):
        self._strategies = {
            SocialNetwork.Twitter: TwitterSubscriberStrategy(),
            SocialNetwork.Instagram: InstagramSubscriberStrategy()
        }

    def get_subscribers(self, user_name: str, network: SocialNetwork) -> List[SocialNetworkUser]:
        return self._strategies[network].get_subscribers(user_name)
```

This Python version replicates the C# logic using idiomatic constructs like abstract base classes and enums.
