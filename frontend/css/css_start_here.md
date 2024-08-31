## CSS Topics

In terms of positioning items, two main concepts in css are:

- Flexbox
- Grid

### Selectors

There are four main selectors in CSS:

- Descednant combinator (space): select **ALL** the HTML elements within a parent
- Child combinator (>): select **immediate** child elements of the selector
- Adjacent sibiling combinator (+): select elements that in the same level as the current element
- General sibling combinator (~): select all the elements that are sibilings after the element targetd by the selector

### Flexbox

In Flexbox, three main ideas are:

- How it is position across (justify-content): flex-start(default), flex-end, center, space-between, space-around
- How it is position vertically (align-items): flex-start(default), flex-end, center, baseline, stretch
- How items inside the box are position (flex-direction): row, row-reverse, column, column-reverse

In order to display content with attributes above, you will need to set the css with `display: flex;`

Sometimes, one of the item has to be in different location compared to other items inside. You can use `order` attribute to move the selected attribute around the container.
