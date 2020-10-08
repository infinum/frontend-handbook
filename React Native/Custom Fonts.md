## Custom fonts

For this tutorial we will be using custom font that can be 
downloaded here https://www.onlinewebfonts.com/download/3a88649e176a40a6d80b395ca0ae430d

### Setup

- To make font crossplatform - you need to have a proper name for it.
Android uses font file name while iOS uses PostScript 
name(with a dash between font name and type).
So to make it crossplatform - put a dash between font name and type.
For example:  
`DINNextLTProBold.ttf` - before (works only on Android)  
`DINNextLTPro-Bold.ttf` - after (crossplatform)

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

### Note

When using custom fonts, css rule `font-weight` doesn't apply.
To use different font weight we need to add separate font file
with that weight, and then change the font-family in css. For example
here https://fontshub.pro/font/din-next-lt-pro-download you can see 
a lot of variations of a single font. So if you need different 
`font-weight` - you will need to download it too and after that you can
use it like `fontFamily: DINNextLTPro-Regular;` or `fontFamily: DINNextLTPro-Light;`

### Documentation

For more details or if you have RN version < 0.60 - check this article:
* [Custom fonts in react native](https://medium.com/@mehrankhandev/ultimate-guide-to-use-custom-fonts-in-react-native-77fcdf859cf4)
