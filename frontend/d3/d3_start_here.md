## D3

### SVG

We use `<svg></svg>` tag to create a new SVG in the DOM. By default, browsers allocate 300px width and 150px height, but it's possible to set different height and width using attribute. For example, `<svg width="900" height=300></svg>` would set the svg to 900px width and 300px height. The problem with this is that the svg is not responsive. We would use viewBox attribute to maintain the aspection ratio so that svg moves depending on the screen size.

```html
<svg viewBox="0 0 900 300" style="border:1px solid black"></svg>
```

ViewBox has four values: the first two sets the origin of the coordinate system, and the last two sets the bottom right corner of the viewbox.

Here are some of the geometric primitives and their attribtues used for drawing shapes in SVG

- Line (<line />)
  - x1,y1,x2,y2: position of the line
  - stroke: color of the line
  - stroke-width: thickness of the line
- Rectangle (<rect />)
  - x,y: position of the rectange from top-left corner
  - width, height
  - fill
  - stroke: add border. Note that when you add stroke to rectangle, wherever the rectangle starts, it will add half of the thickness to the left, and half of the thickness to the right of the rectangle
  - stroke-width: thickness of border
  - stroke-opacity: change opacity of border
  - rx, ry: corner radius
