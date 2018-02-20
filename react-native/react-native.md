# React Native

## Usage

**Remember to wrap text in a `<Text>`**

### Styles

* [view styles](https://facebook.github.io/react-native/docs/view-style-props.html) (`<View>`, etc.)
* [text styles](https://facebook.github.io/react-native/docs/text-style-props.html)
* [image styles](https://facebook.github.io/react-native/docs/image-style-props.html)
* [layout props](https://facebook.github.io/react-native/docs/layout-props.html)
* [shadow props](https://facebook.github.io/react-native/docs/shadow-props.html)

Everything has `relative` position by default. This means that `absolute` positioning will not render an element above its siblings. Instead, any elements that should appear on top of others should be rendered after their siblings.

### Networking

For local networking (when attached to device via USB), use `adb` to let the device access `localhost`.
```bash
adb reverse tcp:{PORT} tcp:{PORT}
```

## Components

### General

#### `<View>`

https://facebook.github.io/react-native/docs/view.html

The "default" component, like an HTML `<div>`.

#### `<Text>`

https://facebook.github.io/react-native/docs/text.html

All text needs to be wrapped in a `<Text>`.

#### `<ScrollView>`

https://facebook.github.io/react-native/docs/scrollview.html

A component that makes it easy to scroll through its content.

#### `<Image>`

https://facebook.github.io/react-native/docs/image.html

Used to display images, which are provided through the `source` prop. This can be an object with a `uri` property or an import (using `require()`).

```jsx
<Image source={{ uri: 'https://somewhere.example.com/okay.png' }}>
<Image source={require('../assets/img/laser.png')}>
```

### Lists

#### `<FlatList>`

https://facebook.github.io/react-native/docs/flatlist.html

Render a list of items. Setup is a bit convoluted. An array of data is provided through the `data` prop. This will only render components as they are needed (when they should be rendered on screen or are near for scrolling).

```jsx
<FlatList data={[...]} />
```

Each item in the array is expected to have a `key` property. If they don't, a `keyExtractor` prop function can be passed to return the key for each item in the array.

```jsx
<FlatList
  data={[...]}
  keyExtractor={item => item.id}
/>
```

The `renderItem` prop is a function that returns a React element for item datum.

```jsx
<FlatList
  data={[...]}
  renderItem={({ item }) => <Text>{item.value}</Text>}
/>
```

#### `<SectionList>`

https://facebook.github.io/react-native/docs/sectionlist.html

This is like a `<FlatList>`, but breaks the data into sections, each of which has a title.

```jsx
<SectionList
  sections={[
    { data: [...], title: '...'},
    { data: [...], title: '...'}
  ]}
/>
```

#### `<RefreshControl>`

https://facebook.github.io/react-native/docs/refreshcontrol.html

Used inside of a list (i.e. `<FlatList>`) to load new data.

### Input

### <TextInput>

https://facebook.github.io/react-native/docs/textinput.html

Used to get user input. Use the `onChangeText` to update state when the user types.


#### `<Picker>`

https://facebook.github.io/react-native/docs/picker.html

Useful for picking an item from a list, but style customization is minimal.

#### `<Slider>`

https://facebook.github.io/react-native/docs/slider.html

Pick a value from a range.

#### `<Switch>`

https://facebook.github.io/react-native/docs/switch.html

Toggle choices

### Buttons

#### `<Button>`

https://facebook.github.io/react-native/docs/button.html

A pre-styled button based on the OS.

#### `<TouchableHighlight>`

https://facebook.github.io/react-native/docs/touchablehighlight.html

A button that will "highlight" (show its `underlay` color) while being pressed.

#### `<TouchableOpacity>`

https://facebook.github.io/react-native/docs/touchableopacity.html

A button that becomes opaque when pressed.

### Misc

#### `<StatusBar>`

https://facebook.github.io/react-native/docs/statusbar.html

This affects rendering around the status bar at the top of the app. The `translucent` prop should only be used if you want to draw underneath the status bar.

#### `<ActivityIndicator>`

https://facebook.github.io/react-native/docs/activityindicator.html

This is a loading icon


#### `<DrawerLayoutAndroid>`

https://facebook.github.io/react-native/docs/drawerlayoutandroid.html

This slides out content from the side. Could be useful for placing navigation links.


#### `<KeyboardAvoidingView>`

https://facebook.github.io/react-native/docs/keyboardavoidingview.html

This squishes itself when the keyboard is opened.

#### `<Modal>`

https://facebook.github.io/react-native/docs/modal.html

Render content above other content.

#### `<WebView>`

https://facebook.github.io/react-native/docs/webview.html

Let's you embed a web page in the app.

## APIs

Because not everything can be components

### AppRegistry

https://facebook.github.io/react-native/docs/appregistry.html

This is used to specify the root component for an app.

### StyleSheet

https://facebook.github.io/react-native/docs/stylesheet.html

Create the styles that will be applied to components using `StyleSheet.create({ ... })`. This also has some properties that might be useful, like `hairlineWidth`, which will set a crisp width for the device.

### Platform

https://facebook.github.io/react-native/docs/platform-specific-code.html

Determine the platform type using `Platform.OS` (`ios` or `android`). `Platform.select` can be used as a switch for setting content based on the OS. `Platform.Version` returns the OS's version.

### Animated

https://facebook.github.io/react-native/docs/animated.html

This is used for animating components, typically by interpolating between two states.

### LayoutAnimation

https://facebook.github.io/react-native/docs/layoutanimation.html

Group animated on re-render

### Easing

https://facebook.github.io/react-native/docs/easing.html

Because linear interpolation isn't always great.

### Transforms

https://facebook.github.io/react-native/docs/transforms.html

Rotate, skew, etc.

### Alert

https://facebook.github.io/react-native/docs/alert.html

At most three buttons on Android (see docs for ordering).

```js
Alert.alert(
  'title',
  'message',
  [
    { text: 'one', onPress: () => {} },
    { text: 'two', onPress: () => {} },
    { text: 'three', onPress: () => {} }
  ]
)
```

### AccessibilityInfo

https://facebook.github.io/react-native/docs/accessibilityinfo.html

Detect screen reader

### AppState

https://facebook.github.io/react-native/docs/appstate.html

Track if the app is in foreground/background/inactive.

### AsyncStorage

https://facebook.github.io/react-native/docs/asyncstorage.html

Store data locally

### BackHandler

https://facebook.github.io/react-native/docs/backhandler.html

Deal with backwards navigation

### CameraRoll

https://facebook.github.io/react-native/docs/backhandler.html

Interact with the camera (roll?)

### Clipboard

https://facebook.github.io/react-native/docs/clipboard.html

Get/set values for copy + paste

### DatePickerAndroid

https://facebook.github.io/react-native/docs/datepickerandroid.html

iOS has a component, but Android uses this instead.

### Dimensions

https://facebook.github.io/react-native/docs/dimensions.html

Get information about the window's size

```js
const { width, height } = Dimensions.get('window');
```

### Geolocation

https://facebook.github.io/react-native/docs/geolocation.html

https://developer.mozilla.org/en-US/docs/Web/API/Geolocation

### Keyboard

https://facebook.github.io/react-native/docs/keyboard.html

Low level API to deal with opening/closing the keyboard. `Keyboard.dismiss()` lets you programmatically close the keyboard.

### Linking

https://facebook.github.io/react-native/docs/linking.html

This has two functions. 1) you can open links in other apps and 2) you can listen for "deep links", which open the app from another app.

### NetInfo

https://facebook.github.io/react-native/docs/netinfo.html

Is the user online/offline, using wifi or cellular data.

### PanResponder

https://facebook.github.io/react-native/docs/panresponder.html

Groups touch events so you can detect gestures.

### PixelRatio

https://facebook.github.io/react-native/docs/pixelratio.html

Info on the pixel density of the device.

### Share

https://facebook.github.io/react-native/docs/share.html

Makes it easy to share content outside of the app.

