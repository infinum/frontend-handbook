There are several ways to persist data inside RN ecosystem. Data persistence methods can generaly be divided between local and server options.
Here we will focus on local storage options of data persistence, therefore naming `data persistence` will refer to local methods only.

Most popular methods for data persistence in React Native are:

 - AsyncStorage
 - Realm
 - SQLite
 - Firebase
 - RN local MongoDB

### AsyncStorage

[AsyncStorage](https://react-native-community.github.io/async-storage/) is the most common method for data persistence in React Native. It's presents asynchronous, unencrypted, persistent, key-value storage system. In other words, AsyncStorage stores data as simple strings and should **never** be used for storing sensitive informations. If you need to save (persist) sensitive informations refer to [Realm](https://www.mongodb.com/realm/mobile/database) or [Firebase](https://rnfirebase.io/) methods. Refer to the same storage systems if you need to save larger files (data) since AsyncStorage has `6MB` limit after which saved data will most likely get deleted by platform storage optimization methods.

**Important**: Even though iOS "*sandbox policy*" permits other applications to access data written by other apps, it can still be accessed in jailbroken phones, since they have root access to the file system.