## CSS Topics

## Definitions

- rem: root em, where em is relative unit based on the font size of the element's parent. For example, if div is the parent of the element p, and div has size 12px, then 0.5 em
  will be 12px \* 0.5 = 6px. The rem in this case, will be based on root element instead of relative.
- Inline element: takes up only the exact amount of space it needs for its content, similar to `<span>` or `<a>` elements
- Block elements: placed in new line and takes the full width of their available space unless given specific width.

In terms of positioning items, two main concepts in css are:

- Flexbox
- Grid

### Grid

As default, Grid in html are set with one column and one row. You can change the sizes by setting `display: grid;` inside the element and then set number of columns and rows
by `grid-template-columns` and `grid-template-rows`. Typically the number of rows and columns are set by `repeat()` function in css, where the first parameter sets how many
columns/rows there will be, and second parameter sets the size. There are four main sizing properties:

- pixel
- rem
- fr
- auto

One useful function for sizing is `minmax()`. This sets what the sizing will be based on what's available on the screen, unless it's hard-coded like pixel and rem. The auto keyword
sets the size of the element based on the size of the element. It will expand or shrink based on what's available and what the size of the contents are. fr keywoard is called fraction
and it's based on what's left after sizing the elements that are currently occupied. It then will automatically divide the leftover spaces equally.

You can preemptively assign area and name them instead of sizing up like grid-row and grid-column attributes and then assign values like grid-row: 2 / 4, using grid-template area:

```css
grid-template-areas:
  'header header header'
  'content content content'
  'footer footer . ';
```

this assigns specific area of the grid into naming variable. The dot shown on bottom right corner indicates that it has no assignable area and is left empty. Once area are now assigned,
we have to assign the elements to the grid area.

```css
header {
  grid-area: header;
}
article {
  grid-area: content;
}
footer {
  grid-area: footer;
}
```

### Flexbox

In Flexbox, three main ideas are:

- How it is position across (justify-content): flex-start(default), flex-end, center, space-between, space-around
- How it is position vertically (align-items): flex-start(default), flex-end, center, baseline, stretch
- How items inside the box are position (flex-direction): row, row-reverse, column, column-reverse

In order to display content with attributes above, you will need to set the css with `display: flex;`

Sometimes, one of the item has to be in different location compared to other items inside. You can use `order` attribute to move the selected attribute around the container.

### Multi-column

This is one of lesser known layout module than grid or flexbox. You can set the number of columns in an element that will continuously flow the content to the next column. To set multi-column, you use
`column-count` property and then set the value to the number you would want.

### Selectors

There are four main selectors in CSS:

- Descednant combinator (space): select **ALL** the HTML elements within a parent
- Child combinator (>): select **immediate** child elements of the selector
- Adjacent sibiling combinator (+): select elements that in the same level as the current element
- General sibling combinator (~): select all the elements that are sibilings after the element targetd by the selector

### Screen size

We need to account for different screen sizes, not just on pc but cellphones and tablets have all different sizes. To set specific layout we use `@media` query to set css attributes
based on the screen size. For example, we can set different grid layout if the screen is large:

```css
@media (min-width: 955px) {
  main {
    grid-template-columns: repeat(2, 1fr);
    grid-template-area:
      'header header'
      'content author'
      'footer .';
  }
}
```

this sets two columns and three rows instead of examples from grid where it sets three columns and three rows.

### Fonts

You can import fonts using `@import url()` on CSS file. The font imports are usually through Google fonts. Fonts have weight attributes that sets the boldness of the font

| Value | Common weight name        |
| ----- | ------------------------- |
| 100   | Thin (Hairlien)           |
| 200   | Extra Light (Ultra Light) |
| 300   | Light                     |
| 400   | Normal (Regular)          |
| 500   | Medium                    |
| 600   | Semi Bold (Demi Bold)     |
| 700   | Bold                      |
| 800   | Extra Bold (Ultra Bold)   |
| 900   | Black (Heavy)             |
| 950   | Extra Black (Ultra Black) |

We can set our font properties like this:

```css
h1 {
  font-weight: 700;
  font-size: 4rem;
  font-family: 'Oswald', sans-serif;
  line-height: 1;
  text-transform: uppercase;
  text-align: center;
}
```

However the shorthand properties are more common:

```css
h1 {
  font: 700 4rem/1 'Oswald', sans-serif;
  text-transform: uppercase;
  text-align: center;
}
```

### Lists

Whether it is unordered or ordered list, we can modify the initial symbols displayed on the list. Typically, the default symbols used are disc or numbers on unordered list and ordered list, respectively. We can set to other symbols, even emojis. To do so, we need to use `@counter-style` to apply to list-style attribute to html elements. There are three main properties in `@counter-style`:

- symbols
- system descriptor
- suffix descriptor

Symbols define what will be used to create the bullet style. The system descriptor defines the algorithm used to convert the list items to the visual representation we set on the symbols. Most common value used on system descriptor is **cyclic** which tells the browser to loop through the list, modify the symbols to the list, and return to the first. The suffix descriptor tells the browser what comes between the bullet and the contents of the list items. By default, it is a period. Here's an example of how all of them fit together:

```css
@counter-style emoji {
  symbols: '\2615';
  system: cyclic;
  suffix: ' ';
}

article ul {
  list-style: emoji;
}
```

Another way to modify the symbol on the list is to use list-style-image. However the `@counter-style` at-rule gives more flexibility than list-style-image. If you need just simple replacement of the symbol in the list, list-style-image is valid attribute to use instead.

### Miscellanous

- You can set position of images by using `float`. Make sure to add some margins to account for some space
