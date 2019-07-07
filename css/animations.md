# CSS Animations

## Transitions

Basic animation of CSS properties can be done using the `transition` property. The animation occurs as a result of a CSS change, such as hovering over an element or adding/removing a class.

```css
.element {
  opacity: 0.5;
  transition: opacity 0.5s;
}

.element:hover {
  opacity: 1;
}
```

The first value passed to `transition` is the property to animate. Not all CSS properties are animatable. The `all` keyword can be used to animate all animatable properties. The second value is the duration of the animation. The third value is the timing function which specifies how the animated value is interpolated at a given point in time; this value is optional. The fourth value is a delay and is also optional.

Multiple animated properties can be provided in one transition; a comma is used to separate them.

```css
transition: opacity 0.5s;
transition: opacity 0.5s linear;
transition: opacity 0.5s linear 0.5s;
transition: opacity 0.5s 0.5s;
transition: all 1s;
transition: opacity 0.5s, color 1s;
```

## Advanced

Advanced CSS animations run automatically (as opposed to the CSS property change required by transitions). They are configured with an `animation` CSS property and a `@keyframes` rule.

An animation's name corresponds to a keyframe's name.

```css
animation-name: test

@keyframes test {}
```

An animation's duration specifies how long one cycle of the animation lasts.

```css
animation-duration: 2s;
```

How properties are interpolated at any X given time in the cycle are specified using a timing function.

```css
animation-timing-function: linear;
```

An animation can be delayed.

```css
animation-delay: 0.5s;
```

An animation can be run for a set number of cycles or continue infinitely.

```css
animation-iteration-count: 5;
animation-iteration-count: infinite;
```

An animation can start over every cycle or run in reverse.

```css
animation-direction: alternate;
```

An animation can be paused or running.

```css
animation-play-state: running;
animation-play-state: paused;
```

The `@keyframes` rule defines the state of a property at a percentage of time in the animation. 0% is also aliased to `from` while 100% is aliased to `to`.

```css
@keyframes example {
  from {
    opacity: 0;
  }

  50% {
    opacity: 0.1;
  }

  to {
    opacity: 1;
  }
}
```

## Resources

* https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions
* https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations
* https://www.kirupa.com/html5/css3_animations_vs_transitions.htm
