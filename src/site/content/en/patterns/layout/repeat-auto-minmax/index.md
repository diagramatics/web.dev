---
layout: pattern
title: RAM (Repeat, Auto, Minmax)
description: A responsive layout with automatically-placed and flexible children.
date: 2021-10-20
draft: true
---

<figure class='w-figure'>
  <video controls autoplay loop muted playsinline class='w-screenshot'>
    <source src='https://storage.googleapis.com/web-dev-assets/one-line-layouts/07-ram-1.mp4'>
  </video>
</figure>

For this seventh example, combine some of the concepts you've already learned about to create a responsive layout with automatically-placed and flexible children. Pretty neat. The key terms to remember here are `repeat`, `auto-(fit|fill)`, and `minmax()`, which you remember by the acronym RAM.

All together, it looks like:

```css/2
.parent {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
}
```

You are using repeat again, but this time, using the `auto-fit` keyword instead of an explicit numeric value. This enables auto-placement of these child elements. These children also have a base minimum value of `150px` with a maximum value `1fr`, meaning on smaller screens, they will take up the full `1fr` width, and as they reach `150px` wide each, they will start to flow onto the same line.

With `auto-fit`, the boxes will stretch as their horizontal size exceeds 150px to fill the entire remaining space. However, if you change this to `auto-fill`, they will not stretch when their base size in the minmax function is exceeded:

<figure class='w-figure'>
  <video controls autoplay loop muted playsinline class='w-screenshot'>
    <source src='https://storage.googleapis.com/web-dev-assets/one-line-layouts/07-ram-2.mp4'>
  </video>
</figure>

```css/2
.parent {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}
```
