## Debugging Network + Flipper

Flipper - is a very nice tool for debugging RN applications. 

### Setup

Download Flipper for your OS here https://fbflipper.com/.

For Android, start Flipper, and start your project using `yarn android`. 
Your application will appear in Flipper.  

For iOS, run `pod install` once in the `ios` directory of your project. 
After that, run `yarn ios` and start Flipper. Your application will show up in Flipper.

### Debugging

When your project will open in Flipper - you will see some plugins
in the left sidebar that will help you debugging your application.
Just click and enable it. For example in case of enabling `Network` plugin - you
will see all network requests that are happening in your application.

There are plugins to debug everything you might need:
- Layout Inspector
- Network
- Databases
- Images
- React DevTools

### Conclusion

If for any reason Flipper is not an option for you - you can also
try React Native Debugger https://github.com/jhen0409/react-native-debugger.

Read more about Flipper here:
* [Getting started with Flipper](https://fbflipper.com/docs/getting-started/react-native)
