## Custom fonts

For this tutorial we will be using custom font that can be 
downloaded here https://www.onlinewebfonts.com/download/3a88649e176a40a6d80b395ca0ae430d

### Setup
- To make font crossplatform - you need to have a proper name for it.
Between font name and type you should have a dash. For example:  
`DINNextLTProBold.ttf` - wrong  
`DINNextLTPro-Bold.ttf` - correct

- Add font file in “assets/fonts” directory

- Define assets directory. Create File `react-native.config.js` and add following code:

```
module.exports = {
  project: {
    ios: {},
    android: {}, // grouped into "project"
  },
  assets: ["./assets/fonts/"], // stays the same
};
```

- Link assets using react native link. Run in terminal:

`react-native link`

- Now you can use your font in your styles:

`fontFamily: DINNextLTPro-Bold;`

### Documentation

For more details or if you have RN version < 0.60 - check this article:
* [Custom fonts in react native](https://medium.com/@mehrankhandev/ultimate-guide-to-use-custom-fonts-in-react-native-77fcdf859cf4)
