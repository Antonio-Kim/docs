## CSS Topics

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
