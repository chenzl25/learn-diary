##Flex

###[A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

###**container part**

####display
```
.container {
  display: flex; /* or inline-flex */
}
```
####flex-direction
```
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

####flex-wrap
```
.container{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

####flex-flow
```
//shortcut
flex-flow: <‘flex-direction’> || <‘flex-wrap’>
```

####justify-content
```
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```
####align-items
```
.container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```
####align-content
```
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```
###**item part**
####order
```
.item {
  order: <integer>;
}
```
####flex-grow
```
.item {
  flex-grow: <number>; /* default 0 */
}
```

####flex-shrink
```
.item {
  flex-shrink: <number>; /* default 1 */
}
```
####flex-basis
```
.item {
  flex-basis: <length> | auto; /* default auto */
}
```
####flex: none
```
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

####align-self
```
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```
---
Ps:*我们可以配合media query来使用，达到更好的效果*
```
@media all and (max-width: 800px) {
  //.....
}
```