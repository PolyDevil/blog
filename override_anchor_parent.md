# How to override anchor parent for element with fixed positioning
There is a simple user avatar component with an upload button:
![image](https://user-images.githubusercontent.com/25101758/100390147-7857eb00-3040-11eb-95f6-1780e64c4c50.png)

There are some requirements we need to meet:
 - both image and icon should be clickable;
 - icon should be accessible with keyboard;

## Clickable area:

![clickable_area](https://user-images.githubusercontent.com/25101758/100393902-53697500-304c-11eb-8a34-eb73bfad8eac.png)

## Focusable area:

![focusable_area](https://user-images.githubusercontent.com/25101758/100394172-32edea80-304d-11eb-8794-24a261d2d7de.png)

## Our markup [demo](https://v4kqc.csb.app/):
```jsx
import React from "react";
import { CameraOutlined } from "@ant-design/icons";
import "./styles.css";

const image = "https://dummyimage.com/600x600/adb0b3/fff.png&text=++%5E_%5E++";

export default function App() {
  return (
    <div className="Avatar">
      <img src={image} alt="user avatar" />
      <form>
        <label name="upload" aria-label="upload new avatar">
          <CameraOutlined />
          <input type="file" />
        </label>
      </form>
    </div>
  );
}
```
Since we want to expand input's clickable area - we need to wrap the icon and the input in `label`.

Let's add some styles:
```css
*,
*:before,
*:after {
  box-sizing: inherit;
}

html {
  box-sizing: border-box;
}

html,
body {
  height: 100%;
  padding: 0;
  margin: 0;
}

#root {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  overflow: hidden;
}

.Avatar {
  width: 50vmin;
  height: 50vmin;
  color: #fff;
  background: currentColor;
  outline: 4px solid #000;
  outline-offset: 10vmin;
  position: relative;
}

img {
  width: 100%;
  border-radius: 50%;
}

form {
  width: 15vmin;
  height: 15vmin;
  background: #0085ff;
  display: inline-block;
  border-radius: 50%;
  position: absolute;
  right: 0;
  bottom: 0;
  border: 1vmin solid currentColor;
}

label {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.anticon {
  font-size: 6vmin;
}

input[type="file"] {
  visibility: hidden;
  position: absolute;
}
```

![markup](https://user-images.githubusercontent.com/25101758/100394326-caebd400-304d-11eb-891f-86f36a5667f0.png)

[Demo](https://tmn5k.csb.app/)

If you hover over blue button (lets call it button) - you will see `pointer` cursor, which means that it is clickable.
But white area around blue button (border) is also clickable. Replacing `border` with `box-shadow` would fix that:
```css
form {
/* ... */
- border: 1vmin solid currentColor;
+ box-shadow: 0 0 0 1vmin currentColor; 
+ overflow: hidden;
}
```

[Result](https://44rdi.csb.app/)

Now only blue area is clickable. But we wanted to make whole image to be clickable, remember?
There is very detailed article about increasing clickable area by [Ahmad Shadeed](https://twitter.com/shadeed9) - [Enhancing The Clickable Area Size](https://ishadeed.com/article/clickable-area/).

We will use `:after` trick.
![image](https://user-images.githubusercontent.com/25101758/100391386-4ba5d280-3044-11eb-8070-e5374297b171.png)

So the idea is:
 - set position of img's parent as `relative`;
 - create `label:before { content: ''; position: absolute; top: 0; right: 0; bottom: 0; left: 0; cursor: pointer; }`

```css
+ label:before {
+  content: '';
+  position: absolute;
+  top: 0;
+  right: 0;
+  bottom: 0;
+  left: 0;
+  cursor: pointer;
+  background: red;
+ }
```

![label_before_absolute (1)](https://user-images.githubusercontent.com/25101758/100394488-5a918280-304e-11eb-9e29-a64b3c311cfe.png)

[Demo](https://heky8.csb.app/)

Well, it didnt work the way we expected. Ther reason is that label's parent - form, is also element with non-static positioning, and as we know that every element with non-static positioning is positioned relative to its closest positioned ancestor. [MDN docs](https://developer.mozilla.org/en-US/docs/Web/CSS/position)

The solution is also in the documentation:
> fixed

> The element is removed from the normal document flow, and no space is created for the element in the page layout. It is positioned relative to the initial containing block established by the viewport, except when one of its ancestors has a transform, perspective, or filter property set to something other than none (see the CSS Transforms Spec), in which case that ancestor behaves as the containing block.

There is also an old article about this behavior of elements with fixed positioning -
http://meyerweb.com/eric/thoughts/2011/09/12/un-fixing-fixed-elements-with-css-transforms/

```css
.Avatar {
/* ... */
- position: relative;
+ transform: translate(0);
}

label:before {
/* ... */
- position: absolute;
+ position: fixed;
  background: rgba(255, 0, 0, .2); /* just to show the area */
}
```

![label_before_fixed](https://user-images.githubusercontent.com/25101758/100394873-bf99a800-304f-11eb-9c05-5e6f95ed0f7d.png)

I highlighted new clickable area with red color.

[Demo](https://80w86.csb.app/)

Let's remove background, add border-radius and cursor:

```css
label:before {
/* ... */
+ cursor: pointer;
+ border-radius: 50%;
+ cursor: pointer;
}
```

[Demo](https://jqzme.csb.app/)

Now we just need to add a style if element is focused:
```
form {
/* ... */
+ transition: color 0.2s ease-in, box-shadow 0.2s ease-in;
}

form:focus-within {
  color: #345b76;
}
```

And remove `input` overlap:
```css
input[type="file"] {
/* ... */
+ pointer-events: none;
}
```

## [Final Result](https://z0x9n.csb.app/)
![final](https://user-images.githubusercontent.com/25101758/100395071-87df3000-3050-11eb-857b-40f68a1eb259.png)

 - Markup is really clean and semantic;
 - Keyboard accessible;
 - Nice clickable areas;
