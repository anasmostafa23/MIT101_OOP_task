# Task 3 Solution (C#)

## Goal

Refactor `VkUserService` and `TwitterUserService` to eliminate duplication of the high-level `GetUserInfo` logic. The algorithm to:

1. Parse the user ID and username from URL
2. Fetch user's friends
3. Convert them into `UserInfo`
4. Return a `UserInfo` object

...must be centralized and separated from the low-level API-specific details.

---

## Solution Overview

We apply the **Template Method** pattern:

* The shared algorithm is placed in an **abstract base class**.
* Platform-specific services (e.g., Twitter, VK) override abstract steps.

This is different from Strategy Pattern used in Task 1.

---

## Step-by-Step Implementation

### Step 1: Create Abstract Base Class

```csharp
namespace Patterns.Ex02
{
    public abstract class SocialNetworkUserService
    {
        public UserInfo GetUserInfo(string pageUrl)
        {
            string userId = ParseUserId(pageUrl);
            string userName = GetUserName(userId);

            var friendsRaw = GetFriends(userId);
            var friends = ConvertToUserInfos(friendsRaw);

            return new UserInfo
            {
                Name = userName,
                UserId = userId,
                Friends = friends
            };
        }

        protected abstract string ParseUserId(string pageUrl);
        protected abstract string GetUserName(string userId);
        protected abstract object[] GetFriends(string userId);
        protected abstract UserInfo[] ConvertToUserInfos(object[] rawUsers);
    }
}
```

### Step 2: Update `VkUserService` to Inherit

```csharp
namespace Patterns.Ex02
{
    public class VkUserService : SocialNetworkUserService
    {
        protected override string ParseUserId(string pageUrl)
        {
            return Parse(pageUrl);
        }

        protected override string GetUserName(string userId)
        {
            return GetName(userId);
        }

        protected override object[] GetFriends(string userId)
        {
            return GetFriendsById(userId);
        }

        protected override UserInfo[] ConvertToUserInfos(object[] rawUsers)
        {
            return ConvertToUserInfo(rawUsers.Cast<VkUser>().ToArray());
        }

        private string GetName(string userId) => "NAME";
        private string Parse(string pageUrl) => "USER_ID";
        private VkUser[] GetFriendsById(string userId) => new VkUser[0];
        private UserInfo[] ConvertToUserInfo(VkUser[] friends) => new UserInfo[0];
    }
}
```

### Step 3: Update `TwitterUserService` to Inherit

```csharp
using System.Linq;
using System.Text.RegularExpressions;
using Patterns.Ex01.resExternalLibs.Twitter;

namespace Patterns.Ex02
{
    public class TwitterUserService : SocialNetworkUserService
    {
        private readonly TwitterClient _client = new TwitterClient();

        protected override string ParseUserId(string pageUrl)
        {
            var regex = new Regex("twitter.com/(.*)");
            var userName = regex.Match(pageUrl).Groups[0].Value;
            return GetUserId(userName).ToString();
        }

        protected override string GetUserName(string userId)
        {
            return _client.GetUserNameById(long.Parse(userId));
        }

        protected override object[] GetFriends(string userId)
        {
            return _client.GetSubscribers(long.Parse(userId));
        }

        protected override UserInfo[] ConvertToUserInfos(object[] rawUsers)
        {
            return rawUsers.Cast<TwitterUser>()
                .Select(c => new UserInfo
                {
                    UserId = c.UserId.ToString(),
                    Name = _client.GetUserNameById(c.UserId)
                })
                .ToArray();
        }

        private long GetUserId(string userName)
        {
            return 0; // Provided method stub
        }
    }
}
```

---

## Why Template Method Pattern?

* Shared algorithm is locked in the base class.
* Subclasses handle the platform-specific implementation.
* No duplication of the high-level `GetUserInfo` logic.
* New networks (e.g., Facebook, Telegram) can easily be added.

---

## Folder Structure

```
PatternsExercises/
├── Ex02/
│   ├── UserInfo.cs
│   ├── VkUser.cs
│   ├── VkUserService.cs               # Modified (inherits base)
│   ├── TwitterUserService.cs         # Modified (inherits base)
│   ├── SocialNetworkUserService.cs   # New (abstract base class)
```

---

## Optional Python Rewrite

```python
from abc import ABC, abstractmethod

class UserInfo:
    def __init__(self, user_id, name, friends):
        self.UserId = user_id
        self.Name = name
        self.Friends = friends

class SocialNetworkUserService(ABC):
    def get_user_info(self, page_url):
        user_id = self.parse_user_id(page_url)
        name = self.get_user_name(user_id)
        raw_friends = self.get_friends(user_id)
        friends = self.convert_to_user_infos(raw_friends)
        return UserInfo(user_id, name, friends)

    @abstractmethod
    def parse_user_id(self, page_url): pass

    @abstractmethod
    def get_user_name(self, user_id): pass

    @abstractmethod
    def get_friends(self, user_id): pass

    @abstractmethod
    def convert_to_user_infos(self, raw_users): pass
```

Each platform will subclass and implement the abstract methods.
