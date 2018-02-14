# Animated

* https://facebook.github.io/react-native/docs/animations.html
* https://facebook.github.io/react-native/docs/animated.html#configuring-animations
* https://facebook.github.io/react-native/docs/gesture-responder-system.html
* https://facebook.github.io/react-native/docs/panresponder.html

## Values

Animated values are created using `Animated.value` and `Animated.valueXY` (for vectors).

```js
const value = new Animated.value(0);
const vector = new Animated.valueXY({ x: 0, y: 0 });
```

### Updating the value

There are three ways that the value is updated:

1. The `value.animate()` method. This method is called internally by the `spring`, `timing`, and `decay` methods to transition between values. It gets passed an `Animation`, which it starts. The `Animation` receives an `onUpdate` callback, which will update the `value`'s value. `onUpdate` is only called for JS animations; native animations will handle updates internally.
2. The `value.setValue()` method. This can be used to set a value at any time, but is especially convenient for setting a starting value before beginning an animation.
3. Through a native event emitted listening for `onAnimatedValueUpdate` events.

### `value.interpolate()`

The "real" value of an `Animated.Value` might not be the same as what you want to render. `value.interpolate()` will map a value from an input range to an output range.

```js
const v = Animated.value(7);
v.interpolate({
  inputRange: [0, 100],
  outputRange: ['0deg', '360deg']
});
```

### Detecting Value Changes

Animate values have an `addListener` method that can be used to call a function when the value changes.

```js
value.addListener(e => this.setState({ x: e.value }));
```

## Animations

Animations start with an initial value (a number) and animate to something. This is usually to another value (i.e. to an `Animated.value`), but can also be velocity based.

Animations can be started and stopped. Once an animation has been stopped, it cannot be restarted.

**Note: Don't forget to call the `animation.start()`!**

### Types

#### Timing

`Animated.timing` animates between two values for a given duration (default is `500ms`).

```js
Animated.timing(value, {
  toValue: 200,
  duration: 1000
}).start(); // don't forget to start
```

#### Spring

`Animated.spring` will overshoot its `toValue` and "spring" back to it (using a [harmonic oscillator](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator))

```js
Animated.spring(value, {
  toValue: 76
}).start();
```

#### Decay

`Animated.decay` will animate a value based on a velocity, starting with an initial velocity the animated value keeps changing until the velocity "decays" to zero.

```js
Animated.decay(value, {
  velocity: { x: 3, y: 2 } // required
});
```

## Combining Animations

* `Animated.sequence` is used to run animations in sequence.
* `Animated.parallel` is used to run multiple animations at the same time.
* `Animated.delay` will start an animation after a delay.
* `Animated.stagger` run in parallel, but staggers the starts.

These methods will automatically call `start()` and `stop()` on their nested animated.

## Components

Animations are frequently controlled through a component's `style` prop, although other props can be animated too.

```jsx
class Bouncer extends React.Component {
  constructor(props) {
    super(props);

    this.value = Animated.value(1);
  }
  render() {
    const transform = {
      scaleX: this.value,
      scaleY: this.value
    };

    return (
      <Animated.View
        style={[
          styles.square,
          { transform }
        ]}
      />
    )
  }
}
```

Only components that are setup for animation can be animated. The `Animated` object provides `Animated.View`, `Animated.Text`, `Animated.Image`, and `Animated.ScrollView` components. `Animated.createAnimatedComponent` can be used to create a animatable component (this is how the four provided components are created).

The above example won't actually animate yet; the animation has to be started. This can be done once a component has mounted, in response to an event, etc.

```jsx
class Bouncer extends React.Component {
  constructor(props) {
    super(props);

    this.value = Animated.value(1);
  }

  componentDidMount() {
    // scale the value to 3x its original size over 2 seconds
    Animated.timing(this.value, {
      toValue: 3,
      duration: 2000
    }).start();
  }

  render() {
    const transform = {
      scaleX: this.value,
      scaleY: this.value
    };

    return (
      <Animated.View
        style={[
          styles.square,
          { transform }
        ]}
      />
    )
  }
}
```

A component created by `Animated.createAnimatedComponent` does a few things to animate. 

**Note:** This is covering JS rendering, not native.

Before the component mounts and receives new props, the component will create an `AnimatedProps`, which iterates over all of the props and attaches itself to any that are an animated node (i.e. `Animated.value`). When a component updates, any old `AnimatedProps` attached to an animated node are removed.

Native components have a [`setNativeProps`](https://facebook.github.io/react-native/docs/direct-manipulation.html) method that is used for setting native properties without re-rendering. When a component created with `Animated.createAnimatedComponent` renders, it will use a React `ref` to store a reference to the underlying component (i.e. `<View>`).

When a value node is updated (i.e. by an animation), it receives a `flush` value. If this is `true`, then the node will call the `update` method of all of its attached children. The update method will call a `AnimatedProps`'s callback method. The behavior of this callback method varies, but for JS rendering, it will call the `ref` component's `setNativeProps` method with the new values.

## Native vs JS Animations

https://facebook.github.io/react-native/blog/2017/02/14/using-native-driver-for-animated.html#caveats

## PanResponder

https://facebook.github.io/react-native/docs/panresponder.html

The `PanResponder` is a responder that lets you react to streams of events (gestures). (The `<Touchable___>` components implement responders, but without gestures.)

A pan responder is created using `PanResponder.create()`. It receives an object with configuration functions.

### Configuration functions


```js
PanResponder.create({
  /*
   * There can be multiple pan responders in a page, so a component that implements a
   * pan responder needs to specify when it should be active. This may either be done at
   * the start of a touch or when a touch moves onto a component (letting it "take over"
   * the gesture).
   * 
   * These methods can either be called during an event's capture phase:
   */
   onStartShouldSetResponderCapture: () => true,
   onMoveShouldSetResponderCapture: () => true,
  /*
   *   
   * or during its bubble phase
   */
  onStartShouldSetResponder: () => true,
  onMoveShouldSetResponder: () => true,
  /*
   * Typically, you would use the bubble functions, but the capture functions are useful
   * if a parent wants to ensure it becomes the pan responder before any children
   * can claim the role.
   */

  /*
   * While a component may try to become the active pan responder, some other component may
   * get the role instead.
   */

  // called when a component successfully becomes the pan responder
  onPanResponderGrant: () => {},
  // called when the component doesn't not become the pan responder
  onPanResponderReject: () => {},

  /*
   * Once a component becomes a pan responder, there are a few functions that it may call.
   */

  // called at the start of a gesture event
  onPanResponderStart: () => {},
  // called every time the gesture moves
  onPanResponderMove: () => {},
  // called when the gesture ends (finger leaves devices)
  onPanResponderRelease: () => {},
  // called when another component becomes the pan responder.
  onPanResponderTerminate: () => {},
  // A component can also allow/prevent itself to be removed as the pan responder
  // where returnin gtrue allows and false prevents
  onPanResponderTerminationRequest: () => true
});
```

All of the method properties passed to `PanResponder.create()` will receive two arguments. The first is a React Native event, which provides information about the latest event. The second is a gesture state, which provides information about the accumulated gesture (how far the gesture has moved, the velocity of the gesture, the current position and where the gesture started).

`onPanResponderMove` can be paired with `Animated.event` to move a component while a gesture is taking place. `Animated.event` will call `setValue` for `Animated.Value`s using event/gesture state values. However, I find this to be a bit convoluted and doing this manually is simple.

```js
onPanResponderMove: Animated.event([
  // set values based on the event
  {
    nativeEvent: {
      // keys are Animated.Values
      locationX: this.xValue
    }
  },
  // set values based on the gesture state
  {
    vx: this.xVelocity
  }
])

onPanResponderMove: (event, gesture) {
  this.xValue.setValue(event.locationX);
  this.xVelocity.setValue(gesture.vx);
}
```
