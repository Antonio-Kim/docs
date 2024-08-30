## CSS Topics

In terms of positioning items, two main concepts in css are:

- Flexbox
- Grid

### Flexbox

In Flexbox, three main ideas are:

- How it is position across (justify-content): flex-start(default), flex-end, center, space-between, space-around
- How it is position vertically (align-items): flex-start(default), flex-end, center, baseline, stretch
- How items inside the box are position (flex-direction): row, row-reverse, column, column-reverse

In order to display content with attributes above, you will need to set the css with `display: flex;`

Sometimes, one of the item has to be in different location compared to other items inside. You can use `order` attribute to move the selected attribute around the container.
