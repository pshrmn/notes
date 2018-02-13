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
2. The `value.setValue()` method. This is a convenient way to set a starting value before beginning an animation.
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
    });
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
