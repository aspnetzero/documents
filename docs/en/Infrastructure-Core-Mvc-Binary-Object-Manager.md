# BinaryObjectManager

User **profile pictures** are stored in database, instead of file system. But it's not stored in Users table for performance reasons (Users are frequently retrieved from database, but profile pictures are rarely needed).

A general-purpose binary saving mechanism built in ASP.NET Zero. **BinaryObject** entity can be used to save any type of binary objects (byte arrays). Since a profile picture can be converted to a byte array, user profile pictures are saved here.

**IBinaryObjectManager** interface defines methods to save, get and delete binary objects. **DbBinaryObjectManager** implements it to save binary object in database. For example, **ProfileController** uses IBinaryObjectManager to get current user's profile picture from database.

You can create a different implementation of **IBinaryObjectManager** interface to store files in another destination.