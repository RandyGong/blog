---
title: Lesser-Known And Underused CSS Features In 2022
date: 2022-06-26 17:43:17
category:
- CSS
tags:
- CSS
---


There are always some attributes in CSS that are easy to be ignored by us, but they can skillfully solve some problems in our work. This article sorts out four CSS attributes that will be used in work but you may not know from [Lesser-Known And Underused CSS Features In 2022](https://www.smashingmagazine.com/2022/05/lesser-known-underused-css-features-2022/), and introduces their application scenarios and specific usage methods with examples, hoping to help you in future work.

## currentColor

Currentcolor is often called "the first CSS variable". Currentcolor is literally "current color", that is, the value of currentcolor is equal to the value of the color attribute of the current element, which looks like inheritance. It can be applied to any attribute that can receive color values, such as **border-color**, **background**, **box-shadow**, etc.

Currentcolor comes in handy when you need to apply the same color value to multiple CSS attributes that receive color values (such as **border-color**, **background**, **box-shadow**) in the same CSS selector.

Let's take a concrete example.

![image.png-146.8kB][1]

As shown above, we can directly set the border and box-shadow attributes of paragraphs in this way.

```CSS
aside {
  border-left: 5px solid currentColor;
  box-shadow: 5px 5px 10px currentColor;
}
.success {
  color: darkgreen;
}
.warning {
  color: darkgoldenrod;
}
.error {
  color: darkred;
}
```

Instead of setting each attribute with color in each class once.

```CSS
.success {
  color: darkgreen;
  border-left: 5px solid darkgreen;
  box-shadow: 5px 5px 10px darkgreen;
}

.warning {
  color: darkgoldenrod;
  border-left: 5px solid darkgoldenrod;
  box-shadow: 5px 5px 10px darkgoldenrod;
}

.error {
  color: darkred;
  border-left: 5px solid darkred;
  box-shadow: 5px 5px 10px darkred;
}
```

### Browser support
![image.png-458kB][2]

## isolation

Isolation is one of the properties of CSS3. Gu's name means "isolation". It can decide whether an element wants to create a new cascading context. It is particularly useful when used with **mix-blend-mode** and **z-index** attributes. Next, let's talk about the scenario of the combination of **isolation** and **z-index**.

When we write a general component of the tooltip prompt class, we usually set its **z-index** attribute. When using, it may happen that the **z-index** value of the tooltip is smaller than the **z-index** value of some elements in the page, causing the tooltip to be blocked. Usually, we will increase the **z-index** value of tooltip, but this may cause other elements to be blocked.

Let's take an example.

![image.png-166.1kB][3]

**h1** has a yellow decorative background, which is implemented with the pseudo element before. In order to display the decorative background under **h1**, we set **z-index**: 1 for before and **z-index**: 2 for **h1** span. Everything is normal.
One day, a tooltip was added to a part of the article. Normally, the **z-index** of tooltip is set to 1. However, when the article scrolls to make the tooltip appear next to **h1**, you will find that the tooltip will be blocked.

There are usually two solutions to this problem:

- Reset the **z-index** values of tooltip and **h1** span
- Use position: relative to set **z-index** for their respective parent elements to create a new cascading context

But is there a better solution? **isolation** is coming. It can create a new cascading context or grouping to play the role of isolation.

Next, let's use isolation:isolate on **h1** and tooltip to see the effect.

```css
h1 {
  isolation: isolate;
  /* ... */
}

.tooltip {
  isolation: isolate;
  /* ... */
}
```

![image.png-165.8kB][4]

### Browser support
![image.png-378.1kB][5]

## font-variant-numeric

When we are doing the countdown or the animation effect of increasing from 0 to a certain value, due to the different width of different numbers, the text will swing from side to side during the animation execution.

The numbers in the following pictures use the Fira sans font. It can be seen that the different widths of the numbers cause the second line to accommodate more than one character.

![image.png-245.8kB][6]


We can solve this problem through the attribute **font-variant-numeric**. The font variant numeric property is used to control the display of alternative glyphs for numbers, fractions, and sequence numbers. There are many values for this attribute. See [here](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric) for details. Among them, tabular numbers can make all numbers the same width. However, it should be noted that these features are affected by fonts, and some fonts may not be supported.

![image.png-46.1kB][7]

### Browser support
![image.png-477.7kB][8]

## scroll-padding-top

When encountering the fixed layout structure of the header at the top of the page, if the jump action of the anchor link is added to the page, without processing, clicking the anchor link to jump to the position will be covered by the header.

![image.png-394.4kB][9]

At this time, the **scroll-padding-top** attribute comes in handy. The **scroll-padding-top** attribute is used to define the offset of the best viewing area of the scrolling window from the top. We can solve this problem by setting the value of scroll padding top to a fixed height for HTML.

```css
html {
  scroll-padding-top: 6rem;
}
```

![image.png-176.8kB][10]

### Browser support
![image.png-442.3kB][11]


  [1]: http://static.zybuluo.com/RandyGong/mo9hfa5yiviaf0n9350gxupf/image.png
  [2]: http://static.zybuluo.com/RandyGong/44kqawipri3xrcv3yl1gkqcd/image.png
  [3]: http://static.zybuluo.com/RandyGong/5iz53qiiy98d8hh3pgehexhb/image.png
  [4]: http://static.zybuluo.com/RandyGong/yttk9kqrp25xort6osahgkqa/image.png
  [5]: http://static.zybuluo.com/RandyGong/5qv90n31qfmr7ndzeruexz8u/image.png
  [6]: http://static.zybuluo.com/RandyGong/na14xep3d0v0jfkz0ee2a1fl/image.png
  [7]: http://static.zybuluo.com/RandyGong/0xgtg7nj832v9rk12jm5wxbl/image.png
  [8]: http://static.zybuluo.com/RandyGong/rgpoylxhe3qml92twgygrk3k/image.png
  [9]: http://static.zybuluo.com/RandyGong/d053o0uiyfjd1ufjl9gql8cd/image.png
  [10]: http://static.zybuluo.com/RandyGong/uug18yjfzhlgracwtehbi8pb/image.png
  [11]: http://static.zybuluo.com/RandyGong/xyairh6afjofgzxqzl0z01ag/image.png