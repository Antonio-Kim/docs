## Basic Layouts in .NET MAUI

- **StackLayout**: organizes elements in a one-dimensional stack, either horizontally or vertically. It is often used as a parent layout, which contains other child layout. Two subtype layout are used:
  - **Horizontal**
  - **Vertical**
- **FlexLayout**: similar to StackLayout but can also wrap children if there are too many to fit in a single row or column
- **AbsoluteLayout**: particularly useful in scenarios where you need fin-grained control over exact position. However, it should be noted that this layout should be used only in certain situation where it is beneficial over other layout.
