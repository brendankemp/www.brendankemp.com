---
layout: post
title: "Refactor your breakpoints to be easier to write and understand"
date: 2018-12-17 09:25:37 -0500
comments: true
categories:
published: false
---

### Mobile First for Design
When we talk about "mobile-first", we're usually talking about the designer's realm. Designers focus on designing the mobile experience of a responsive web app first so that they focus on the most essential, pared-down experience before defining layout changes (and potentially additional features) at larger sizes.

### Mobile First for CSS
But "mobile-first" can also be a handy organizing principle for your breakpoints. Take this component for an example:

```sass
.main-content {
  font-size: 24px;
  padding: 16px 32px;
}

// [...] All other selectors without a breakpoint - potentially hundreds of lines

@media(min-width: 1280px) {
  // [...] Dozens more selectors modifying components at this breakpoint

  .main-content {
    font-size: 32px;
    padding: 32px 64px;
  }
}

@media(max-width: 768px) {
  .main-content {
    font-size: 16px;
    padding: 8px;
  }

  // [...] Dozens more selectors modifying components at this breakpoint
}
```

The first issue is that the code for `.main-content` is divided by hundreds of lines of unrelated SCSS. If we are using SCSS we can nest all our media queries under the `.main-content` selector, and see all the related code at once:

```sass

```
