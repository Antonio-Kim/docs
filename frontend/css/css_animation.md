## CSS Animation

### Very basic introduction to Scalable Vector Graphics (SVGs)

We'll start with introduction to SVG. SVG stands for Scalable Vector Graphics. Unlike pixel rendering, or rasterization, SVGs are scalable and can be
fitted into any screen. The downside of this flexibility is that it has higher memory cost than rasterization. SVGs uses cartesian coordinate that
starts from top left corner of the screen as (0,0) point, and bottom right corner as the maximum allotted. SVGs uses XML so it can be used in
HTML tags, like this: `<svg>`. The main size of the SVG is called viewport, which has two main attributes: width, and height. The location of
the content inside the viewport is called viewbox, and it has one main attribute that sets dimensions: min-x, min-y, width, and height. You can manually
draw the lines at SVG elements, and here are some useful ones:

- rect
- circle
- ellipse
- line
- polyline
- polygon

### Animation in CSS

Keyframe is like a frame done in movies. We know that each film scene are divided into frames, and the speed at which each frame is passed is called
frames per second. Keyframe indicate important frame within animation, and the most common ones are beginning, middle, and the end. The frames between
the keyframes are called in-between frames and these are generally rendered by the browser. For us we need to set the keyframes and let the browser
render the in-between frames for animation to work.

There are various properties used in CSS to style each elements, but here are some most commonly used properties:

- animation-delay: how long to wait before the animation starts
- animation-direction: whether the animation is played forward or backward
- animation-duration: how long it should take for the animation to run once
- animation-fill-mode: how the element being animated should be styled when the animation is done executing
- animation-iteration-count: how many times the animation should run
- animation-name: name of the keyframes being applied
- animation-play-state: whether the animation is running or paused
- animation-timing-funciton: how the animation progresses through the style over time

We first set the keyframe using at-rule syntax:

```css
@keyframes changeColor {
  from {
    background: green;
  }
  50% {
    background: yellow;
  }
  end {
    background: red;
  }
}
```

we can then call this keyframe into our elements in css:

```css
div {
  animation: changeColor, 3s, 10;
}
```

as you can see, the div element will call the changeColor keyframe and then transition from gree to red in 3s, setting yellow at 1.5s. This process will repeat for 10 times and stop afterwards.

Animating elements are usually done at origin based on its element. To modify the origin we use `transform-origin` property in CSS. The property has three attributes:

- Length
- Percentage
- Keywords: top, right, center, left, bottom

Note that the default origin depends the element at hand. For example, html origins are 50% length, 50% percentage, and center, whereas SVG origins are 0, 0, 0.

### Styling on browsers

Typically each browsers set the default attributes for each elements, and can be separated into two major categories: firefox, and everyone else. Sometimes we want to suppress the styling from
the browsers or modify styling based on the browsers.

| prefix   | Browsers                                       |
| -------- | ---------------------------------------------- |
| -webkit- | Chrome, Safari, Opera, most iOS browsers, Edge |
| -moz     | Firefox                                        |

Because some element's styling are not equal between webkit and moz, we suppress all of them and set our own styling. For example, progress bar looks different between -webkit- and -moz-, so we
can suppress the styling like this:

```css
progress {
  -webkit-appearance: none;
  -moz-appearance: none;
}
```
