# Notifications

Notification icon is located next to the language selection button. 

<img src="images/notifications-icon-1.png" alt="notifications" class="img-thumbnail" />

User can see 3 recent notifications by clicking this icon.

<img src="images/recent-notifications-1.png" alt="notifications" class="img-thumbnail" />

User can marks all notifications as read by clicking the **Set all as read** link or can mark a single notification by clicking the **Set as read** link next to each notification. Notifications are sent real-time using SignalR. In addition, a **desktop push notification** is shown when a notification is received.

## Notification Settings

**Settings** link opens notification settings dialog.

<img src="images/notification-settings-2.png" alt="notifications" class="img-thumbnail" />

In this dialog there is a global setting for user to enable/disable receiving notifications. If this setting is enabled, then user can enable/disable each notification individually.

You can also define your custom notifications in **AppNotificationProvider** class. For example, new user registration notification is defined in the **AppNotificationProvider** as below.

````csharp
context.Manager.Add(
     new NotificationDefinition(
        AppNotificationNames.NewUserRegistered,
        displayName: L("NewUserRegisteredNotificationDefinition"),
        permissionDependency: new SimplePermissionDependency(AppPermissions.Pages_Administration_Users)
    )
);
````

See [notification definitions](https://aspnetboilerplate.com/Pages/Documents/Notification-System#notification-definitions) section for detailed information.

**AppNotifier** class is used to publish notifications. **NotificationAppService** class is used to manage application logic for notifications. 

When a notification is sent, Angular app receives it via SignalR and **UserNotificationHelper.ts** (under `app\shared\layout\notifications\` folder) is used to format this notification information before showing it to user. If you want to redirect user to a new page or to an external website, you can modify **getUrl** method. 

See [notifications documentation](https://aspnetboilerplate.com/Pages/Documents/Notification-System) for detailed information.

## Notification List

All notifications of the user are listed in this page. We can delete and mark a notification as read in this page.

<img src="images/notifications-list-core-4.png" alt="Notification list" class="img-thumbnail" />

## Next

- [Chat](Features-Angular-Chat)