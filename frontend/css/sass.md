### SASS for CSS

- Use .SCSS for all SASS components since it is superset of CSS. The other method is using indent but the focus will be on SCSS since it looks and feels exactly like CSS.
- Variables were popular in SASS because earlier days in CSS did not have variables. To create a variable, add $ followed by variable name, then :, and then the value.

```scss
$primary: #063373;
```

- To reduce redundancy, SASS allows base rule and have it extended or included, using additional at-rule syntax called `@extend` and `@include`. For example,

```scss
.image-base {
  width: 300px;
  height: 300px;
  object-fit: cover;
  margin: 0 2rem;
}

img:first-of-type {
  @extend .image-base;
}
img:nth-of-type(2) {
  @extend .image-base;
}
img:last-of-type {
  @extend .image-base;
}
```

- Like functions, `@mixins` and `@include` allows generateion of declarations and rules. Mixings takes parameters (not mandatory) and return styles. To use the style inside css, we use include with specific parameters to apply the style inside. Mixins start with declaration of `@mixins`, then variable name, then parameters. Let's take an example:

```scss
@mixin handle-img($border-radius, $position, $side) {
  border-radius: $border-radius;
  object-position: $position;
  float: $side;
  margin-#{side}: 0;
}
```

note that margin-#{side} takes in interpolation, just like in JavaScript's backtick.

- The key difference between `@extend` and `@mixins` is that Sass doesn't copy or generate code when extend; it only adds selector to the base. Whereas for mixins, Sass generates code. Thus, if you want to dynamically modify properties, use mixin. Otherwise, use extend instead of copying values.

- You can also reduce redundancy by using nesting, where you do not need to create multiple css for each of pseudo-classes.

```scss
a {
  font-weight: 800;
  &:link,
  &:visited {
    color: $primary;
    text-decoration-style: dotted;
  }
  &:hover {
    text-decoration-style: dashed;
  }
  &:focus {
    text-decoration-style: solid;
    outline: none;
  }
}
```

- Similar to looping over dictionary, we can create a map to set key-value pairs, and use `@each` to create css for each key-value pair. Here's a great example of using every concepts stated above:

```scss
.call-out {
  border: solid 1px;
  border-radius: 4px;
  padding: 0.5rem 1rem;
}

$callouts: (
  success: $success,
  warning: $warning,
  error: $error,
);
@each $type, $color in $callouts {
  .#{$type} {
    @extend .call-out;
    border-color: $color;
    &::before {
      content: '#{$type}: ';
      text-transform: capitalize;
    }
  }
}
```

- colours can be created using colour functions in scss, `scale-color`. The parameters however, has to be either rgb or hsl, and cannot be both.
- We can also create if-else statement in Sass by using `@if` and `@else`
