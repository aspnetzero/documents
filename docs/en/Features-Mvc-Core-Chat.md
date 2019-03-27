# Chat

Chat icon is located next to user's profile image on top right corner of the page. The number in the red circle shows total unread chat message count.

<img src="images/chat-icon-1.png" alt="User menu" class="img-thumbnail" />

When user clicks this icon, chat panel appears on the right of page. This panel contains friends of user and list of blocked users.

<img src="images/chat-friends-1.png" alt="User menu" class="img-thumbnail" />

User can add new friends by writing the username into username textbox above friend list. If "Chat with other tenants" feature is enabled for tenant, users of other tenants can be added as a friend by writing
\[tenancy name\]\\\[user name\] (ex: Default\\admin). If "Chat with host users" feature is enabled, host users can be added as friend by writing .\\\[user name\] in the same textbox. 

While online friends/users have a green circle on their profile image, offline friends/users have a gray circle.

User can pin or unpin the chat panel by clicking the pin icon on top right corner of the chat panel. Application tries to remember last state of chat panel and restores it when user login to application.

When a friend/user is selected, conversation panel is opened.

<img src="images/chat-conversation-1.png" alt="User menu" class="img-thumbnail" />

Chat system also allows sending images, files and link of current page to friends

<img src="images/chat-attachments-core.png" alt="Chat attachments" class="img-thumbnail" />

User can block or unblock friend/user in this area. There is a wrench icon right of the selected user's username. This icon opens an action menu and this menu contains block user or unblock user actions according to user's block status.

Chat messages are distributed over **ChatHub** signalR hub class which uses **ChatMessageManager** domain class.

**ChatUserStateWatcher** class is responsible for watching online/offline state changes of chat users. When a user becomes online or offline, this class catches the state change and notifies friends of related user.

**FriendshipAppService** and **FriendshipManager** classes are responsible for managing friendship requests. Chat messages from blocked users are not delivered to target users.

Since chat is a real time operation, application caches friends of online users and unread message count from each friend. **UserFriendsCache** class manages these caching operations.

**UserFriendCacheSyncronizer** class is responsible for keeping user friends cache up to date. In order to do that, it watches some events of Friendship and ChatMessage entities.

## Chat Features

<img src="images/chat-features-1.png" alt="User menu" class="img-thumbnail" />

There are three chat features in the system. These are "Chat", "Chat with host", "Chat with other tenants". These features can be enabled/disabled per edition/tenant. By using these features host can enable/disable chat with other tenant's users or host users.

## Next

- [User Menu](Features-Mvc-Core-User-Menu)