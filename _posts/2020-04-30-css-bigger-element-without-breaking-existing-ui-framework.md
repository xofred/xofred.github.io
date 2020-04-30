---
title:  "CSS Bigger Element Without Breaking Existing UI Framework"
tags: css frontend

---
*-webkit-transform: scale(2);*

Let's say we want to make the checkboxes in table bigger.

![](https://user-images.githubusercontent.com/2174219/80669497-274a3d80-8ad7-11ea-9663-651c773f20a2.png)

If we just use `width` and `height`, yes the size will change, but also break the UI framework, e.g., Bootstrap.

![](https://user-images.githubusercontent.com/2174219/80669644-83ad5d00-8ad7-11ea-8040-2b3df59fb58a.png)

So maybe we should use transform with scale instead.

```css
td input[type=checkbox]
{
  /* Double-sized Checkboxes */
  -ms-transform: scale(2); /* IE */
  -moz-transform: scale(2); /* FF */
  -webkit-transform: scale(2); /* Safari and Chrome */
  -o-transform: scale(2); /* Opera */
}
```

![](https://user-images.githubusercontent.com/2174219/80669785-cec77000-8ad7-11ea-8d85-f0a56b65dd94.png)



### Reference

[How can I change the size of a Bootstrap checkbox?](https://stackoverflow.com/questions/22743457/how-can-i-change-the-size-of-a-bootstrap-checkbox)
