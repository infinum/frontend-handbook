There are two methods on how we use and implement icons. The first one is by adding 
custom svg icon files to the project, and the second one by using the library `react-native-vector-icons`.

The first approach with custom svg icon files is recommended when the icons are not part of an icon set (same as icon font on web). This icons end up being custom made for a specific project. To use this method refer to the [Assets](https://infinum.com/handbook/books/frontend/assets) section in this handbook.

## react-native-vector-icons

Otherwise if the project contains of icons that are available in icon sets like `Material icons, FontAwsome, AntDesign` etc.
you should use the `react-native-vector-icons`. To install the library follow the instructions on [https://github.com/oblador/react-native-vector-icons](https://github.com/oblador/react-native-vector-icons).

You should not import all available icon sets. It's enough to import just the icon set/s that are used (see example below).
Also the library provides you with a `Icon` component that receives 3 main props, `name` for the icon name, `size` for the icons
size in dpi's (default is 12) and `color` for the color of the icon (default is inherited).
For other styling props and static methodes for the `Icon` component you can refer to the previouse linked documentation.


***Note:*** To use the right icon name refer to this link [https://oblador.github.io/react-native-vector-icons/](https://oblador.github.io/react-native-vector-icons/) which is a directory of all available icons.


```javascript
import Icon from 'react-native-vector-icons/MaterialIcons';

const myIcon = <Icon name="add" size={25} color="#ff0000" />;
```



The `react-native-vector-icons` provides you also with a button component where you can easly implement an icon on the left side of a button.

```javascript
import Icon from 'react-native-vector-icons/FontAwesome';

  <Icon.Button
    name="facebook"
    backgroundColor="#3b5998"
    onPress={this.loginWithFacebook}
  >
    Login with Facebook
  </Icon.Button>
```




