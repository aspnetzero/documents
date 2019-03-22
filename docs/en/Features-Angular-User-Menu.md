# User Menu

A user can click his name at top right corner to open user menu:

<img src="D:/Github/documents/docs/en/images/user-menu-4.png" alt="User menu" class="img-thumbnail" />

## Linked Accounts

Linked accounts are used to link multiple accounts to each other. In this way, a user can easily navigate through his accounts using this feature.

User can link new accounts or delete already linked accounts by clicking the **Manage accounts** link.

<img src="D:/Github/documents/docs/en/images/linked-accounts-3.png" alt="User menu" class="img-thumbnail" />

In order to link a new account, user must enter login credentials of related account.

<img src="D:/Github/documents/docs/en/images/link-new-account-1.png" alt="link new account" class="img-thumbnail" />

## Profile Settings

My settings is used to change user profile settings:

<img src="D:/Github/documents/docs/en/images/user-settings-3.png" alt="User settings" class="img-thumbnail" />

As shown here, **admin** user name can not be changed. It's considered a special user name since it's used in database migration seed. Other users can change their usernames.

## Login Attempts

All login attempts (success of failed) are logged in the application. A user can see last login attempts for his/her account.

<img src="D:/Github/documents/docs/en/images/login-attempts-1.png" alt="Login attempts" class="img-thumbnail" />

## Change Picture

A user can change own profile picture. Currently JPG, JPEG, GIF and PNG files are supported, you can extend it.

## Change Password

**ProfileAppService** is used to change password.

## Download Collected Data

A user can download his/her collected data using this menu item.

<img src="D:/Github/documents/docs/en/images/gdpr_download_item.png" alt="Login attempts" class="img-thumbnail" />

## Logout

**AccountController** is used to logout the user and redirect to Login page.

## Next

- [Setup Page](Features-Angular-Setup-Page)