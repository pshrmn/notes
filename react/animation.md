# React Animation

## Animation Basics

There are two prominent ways to perform animation on web pages. The first is using JavaScript and the second is using CSS. React has a number of libraries that have been written to assist in animating DOM elements created by React.

### JavaScript Animation

Animation with JavaScript is done by manually setting element attributes and styles. Over the course of an animation, these values should be constantly updated. In order to achieve 60 frames per second, this means that for each second of an animation, an animated attribute or style should be updated with 60 different values.

An interpolator is used to determine which value an animated attribute or style should have at a given time. The simplest interpolator is a linear interpolator. For an animation that lasts two seconds, at 0 seconds, the animated value will have its starting value. At 1 second, the value will be halfway between the starting and ending values. At 2 seconds, the value will be its ending value.


More complicated easing functions can be used to change how the values are determined throughout the animation while maintaining the same starting and ending values. For example, an ease-in funtion will change the value slowly in the beginning of an animation, and then more rapidly as the animation finishes.

### CSS Transition

CSS can be used to animate elements. An animation between a starting state and an ending state is called a transition. Transitions can be invoke either through events, such as `hover`, or by adding and removing classes from an element.
  
CSS transitions describe which property to animate, how long the animation should last, what easing function to use, and any delay there should be before the animation begins. The transition will be triggered when the CSS rule that contains the transition is applied to an element.

The browser will handle determining the correctly values for you throughout the animation. It just needs to know what start and end values to interpolate between.

```css
div {
  background: blue;
}

div:hover {
  background: red;
  transition: background 1s;
}
```

## Animation in React

Both of the above methods can be used to animate components in React.

Using CSS is simpler because most of the work is done in CSS. When you render a component, you will just need to set a property (most likely as part of the `className` prop) that will start a CSS transition.

JavaScript animation is more involved because you will need to constantly re-render a component throughout the duration of the animation. In each re-render, the component will have to calculate the value of its animated attributes and styles at that point in time (the percentage of the way through the animation). This can be done either through `props` or `state`.

For example, a component that animates using `state` could have a function that adds a `requestAnimationFrame` callback. Each time that the callback is called, the elapsed time would be calculated to determine the new values, and then `setState` would be called to set the new value(s) of animated attributes and styles.

On a similar note, animation of components that are being unmounted is not straightforward. When a component is unmounted, its rendered element no longer exists. How, then, can it be animated? You would need to maintain a reference to that element manually and then render it for the duration of the animation before deleting the reference.

If rolling your own animation doesn't sound to appealing, your are in luck because there are a number of libraries that will handle animation for you. Two popular animation libraries are [`react-transition-group`](https://github.com/reactjs/react-transition-group) and [`react-motion`](https://github.com/chenglou/react-motion).

## `react-transition-group`

[On GitHub](https://github.com/reactjs/react-transition-group)

```bash
npm install react-transition-group
```

React Transition Group provides components to make animation of entering and leaving components simple.

When a component unmounts, it will still be rendered in the DOM until its leaving animation has finished.

### <TransitionGroup>

The `<TransitionGroup>` keeps track of the elements are passed to it through its `children` prop. The direct children of a `<TransitionGroup>` should be `<Transition>`/`<CSSTransition>` components. These transition components need to be given `key` props to uniquely identify them.

```jsx
const AnimatedComponent = () => (
  <TransitionGroup>
    <CSSTransition key="one">
      <Item>One</Item>
    </CSSTransition>
    <CSSTransition key="two">
      <Item>Two</Item>
    </CSSTransition>
  </TransitionGroup>
)
// knownKeys = ['one', 'two']
```

In its state, the `<TransitionGroup>` maintains a list of all of its children. Whenever the `<TransitionGroup>` updates, it compares the keys of its new `children` prop to the ones stored in state. Doing this, it creates two arrays: one which contains entering elements (their key was not in the existing element list) and one which contains leaving elements (their key was not in the new `children` elements).

```jsx
const AnimatedComponent = () => (
  <TransitionGroup>
    <CSSTransition key="one">
      <Item>One</Item>
    </CSSTransition>
    <CSSTransition key="three">
      <Item>Three</Item>
    </CSSTransition>
  </TransitionGroup>
)
// knownKeys = ['one', 'two']
// updateKeys = ['one', 'three']
// enteringKeys = ['three']
// leavingKeys = ['two']
```

```jsx
const AnimatedComponent = () => (
  <TransitionGroup
    classNames="example"
  >
    <CSSTransition key="one">
      <Item>One</Item>
    </CSSTransition>
    <CSSTransition key="three">
      <Item>Three</Item>
    </CSSTransition>
  </TransitionGroup>
)
// assuming Item renders a div, the following will be rendered:
// <div>One</div>
// <div class="example-leave">Two</div>
// <div class="example-enter">Three</div>
```

On the next animation frame, each entering and leaving element will also be given an "active" class based on their action. Delaying until the next animation frame will ensure that the starting CSS values are applied to the element before beginning the transition.

```html
// <div class="example-leave example-leave-active">Two</div>
// <div class="example-enter example-enter-active">Three</div>
```

### Transition components

```jsx
<Transition
  // control when children mount/unmount
  mountOnEnter={false}
  unmountOnExit={true}
  // when true, this will transition when included
  // in the initially mounted children
  appear={false}
  // enter/exit transitions can be turned off
  enter={true}
  exit={true}
  // the duration of transitions is controlled through
  // the timeout prop. These can either be individually set for
  // appear/enter/exit or with one constant duration
  // REQUIRED
  timeout={500}
  timeout={{
    enter: 500,
    exit: 750
  }}

>
  <Thing />{/* An element or function (render prop) */}
</Transition>

<CSSTransition
  // The <CSSTransition> sets classes based on
  // the classNames prop
  classNames="fade"
  classNames={{
    active: "fading",
    enter: "fade-in",
    exit: "fade-out"
  }}
>
</CSSTransition>
```

## `react-motion`

[On GitHub](https://github.com/chenglou/react-motion)

```bash
npm install react-motion
```

React Motion allows you to animate the `style` prop of an element. Its `<Motion>` component allows you to animate components within the page. The `<TransitionMotion>` component is similar to `<Motion>`, but is used to animate components as they mount and unmount.

## Springs

React Motion uses "springs" to interpolate the values of animated properties throughout the duration of an animation. A `spring` is a type of easing function that allows you to control the stiffness and damping of the interpolation. The idea is that by tuning these values, you can get a more pleasing movement in an  animation.

A `spring` call takes two arguments. The first is the ending value and the second is a configuration object in which you can specify the `damping` and `stiffness` of the spring. React Motion also provides a few preset configuration objects that you may prefer to use.

Springs <strong>only</strong> work with numbers. For example, if you wanted to animate colors, you would have to convert them to a numerical representation like RGB or HSL.

You can use [this demo](http://chenglou.github.io/react-motion/demos/demo5-spring-parameters-chooser/) provided by React Motion to help you in choosing which `stiffness` and `damping` values work best for your animation.

```js
spring(50, { stiffness: 180, damping: 30 })
```

## <Motion>

The concept behind React Motion is that you animate between style states. The `children` passed to a `<Motion>` component should be a function. That function's job is to render a component. The argument passed to the function is the style that should be used to render the component at that point in time.

```jsx
const Thing = (props) => (
  <Motion style={{ opacity: spring(0.5) }}>
    { (style) => <div style={style}>Can you see me?</div> }
  </Motion>
)
```

You can control the initial style values by passing a `defaultStyle` prop to the `<Motion>` component.

```jsx
const Thing = () => (
  <Motion defaultStyle={{ opacity: 0 }} style={{ opacity: spring(1) }}>
    { (style) => <div style={style}>Can you see me?</div> }
  </Motion>
);
```

You can check out a demo of the `<Motion>` component in this [codepen](http://codepen.io/pshrmn/pen/pRroLV).

## <TransitionMotion>

The `<TransitionMotion>` component animates components as they mount and unmount. The premise is similar to React Transition Group, but the approach is completely different.

`<TransitionMotion>` requires a `styles` prop. This should be an array of objects. Each object must have a `key` property, which is used to identify the object. The object must also include a `style` property which is an object containing the style properties and their values that should be animated. Optionally, a `data` property can also be included, which contains data to pass along to the rendered object.

Alternatively, the `styles` prop can be a function. That function will receive the config array from the previous rendering. It should return an updated config array.

Similar to the `<Motion>` component, `<TransitionMotion>` expects a function as its `children` prop. The argument passed to that function will be an array with `key`, `style`, and `data` properties. The function should iterate over the array and return a React element for each item in the array. A `defaultStyles` array can also be passed, similar to `<Motion>`'s `defaultStyle` prop. Like the `styles` prop, each item in the `defaultStyles` array must have a `key` property.

```jsx
const App = () => (
  <TransitionMotion
    defaultStyles={[
      { key: 'one', style: {borderWidth: 0}},
      { key: 'two', style: {borderWidth: 0}},
      { key: 'three', style: {borderWidth: 0}}
    ]}
    styles={[
      { key: 'one', style: { borderWidth: spring(10) }, data: 'one'},
      { key: 'two', style: { borderWidth: spring(5) }, data: 'two'},
      { key: 'three', style: { borderWidth: spring(15) }, data: 'three'}
    ]}
    >
    {(styles) => (
      <div>
        { styles.map(({ key, style, data}) => (
          <div key={key} style={{
            borderColor: 'black',
            borderStyle: 'solid',
            ...style
          }}>{ data }</div>
        ))}
      </div>
    )}
  </TransitionMotion>
);
```

You can check out a demo of the `<TransitionMotion>` component in this [codepen](http://codepen.io/pshrmn/pen/egEmwZ).

How does `<TransitionMotion>` animate components that are unmounting? By default, when a component unmounts it disappears. However, you can provide a `willLeave` functoin prop to animate a leaving component.

When the component detects that an item with a known key no longer exists in the `styles` array, it will pass the last known configuration object for that style to the `willLeave` function. That function should return a style object of properties that the component should animate to before being removed from the DOM.

```jsx
const willLeave = () => ({
  borderWidth: spring(0)
})

const App = (props) => (
  <TransitionMotion
    styles={props.items ? props.items.map(item => ({
      key: item.key
      styles: item.styles,
      data: item.data
    })) : []}
    willLeave={willLeave}
    >
    {(styles) => (
      <div>
        { styles.map(({ key, style, data}) => (
          <div key={key} style={{
            borderColor: 'black',
            borderStyle: 'solid',
            ...style
          }}>{ data }</div>
        ))}
      </div>
    )}
  </TransitionMotion>
);
```

You can check out a demo of the `<TransitionMotion>` component with entering and leaving styles in this <a href="http://codepen.io/pshrmn/pen/OWjVaP">codepen</a>.
