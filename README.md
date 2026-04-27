# p5.transparency

An addon for rendering objects with transparency in p5.js WebGL mode.

In WebGL mode, you may have noticed that transparent images still occlude other objects as if they're opaque. So if I draw a transparent cube close to the screen, if I then draw stuff behind it, they won't show up. This library provides helper functions you can surround each item with to make your items blend better.

<table>
<tr>
  <th>Before</th>
  <th>After</th>
</tr>
<tr>
  <td>

![transparency(1)](https://github.com/user-attachments/assets/106f3be9-413a-4c63-a352-13c0a84b5596)
    
  </td>
  <td>
    
![transparency](https://github.com/user-attachments/assets/5142f12e-98bc-46d7-a52c-a2b498b3cc00)

  </td>
</tr>
<tr>
  <td>

```js
for (const { x, y, z } of items) {
  push();
  translate(x, y, z);
  imageMode(CENTER);
  image(tex, 0, 0);
  pop();
}
```
    
  </td>
  <td>

```js
for (const { x, y, z } of items) {
  push();
  translate(x, y, z);
  drawTransparent(() => {
    imageMode(CENTER);
    image(tex, 0, 0);
  });
  pop();
}
```
    
  </td>
</tr>
</table>

<table>
<tr>
  <th>Before</th>
  <th>After</th>
</tr>
<tr>
  <td>

  ![transparence](https://github.com/user-attachments/assets/585cf1aa-7057-43a1-b2df-9f66691b0604)

    
  </td>
  <td>

  ![transparence(1)](https://github.com/user-attachments/assets/664bfff4-5908-4c5a-93b7-5481a26fbd39)
    
  </td>
</tr>
<tr>
  <td>

```js
for (const { x, y, z } of items) {
  push();
  translate(x, y, z);
  texture(tex);
  box(100);
  pop();
}
```
    
  </td>
  <td>

```js
for (const { x, y, z } of items) {
  push();
  translate(x, y, z);
  drawTwoSided(() => {
    texture(tex);
    box(100);
  });
  pop();
}
```
    
  </td>
</tr>
</table>


## Installation

Via a script tag:

```html
<script src="https://cdn.jsdelivr.net/npm/p5.transparency@0.0.19/p5.transparency.min.js"></script>
```

On OpenProcessing, paste this link into a new library slot:
```
https://cdn.jsdelivr.net/npm/p5.transparency@0.0.19/p5.transparency.min.js
```

## Usage

Translate to where you want your object to be draw, and then call `drawTransparent`, passing it a function to draw your semi-transparent object:

```js
translate(x, y, z);
drawTransparent(() => {
  imageMode(CENTER);
  image(tex, 0, 0);
});
```

If you are drawing a single object, such as a cube, that might occlude its own faces, use `drawTwoSided` instead:
```js
translate(x, y, z);
drawTwoSided(() => {
  texture(tex);
  box(100);
});
```

## How does it work?

The typical answer to the transparency problem is to draw your shapes in back-to-front order, ensuring that each shape won't occlude the next one. This works, but can be difficult when the definition of "back" and "front" can change dynamically as your camera moves.

This library will wait as long as it can to draw transparent object in order to collect them all, and then it automatically sorts all `drawTransparent` calls based on how close to the camera they are. Additionally, it discards any pixels with 0 transparency.

For objects that have faces that might occlude themselves, the `drawTwoSided` function does everything `drawTransparent` does, but will also draw objects in two passes: first just the back faces, and then just the front faces. This involves rendering each object twice, so it will be a bit slower, but will make three dimensional shapes with transparent textures work better.

## Tips and warnings

- **Make sure you've translated to the center of your item before calling `drawTransparent`.** Items are sorted based only on transformations you've done up until the point where you call `drawTransparent`, so if you draw something offcenter, the sorting may not be accurate.

- **Make sure any shared resources are modified *inside* `drawTransparent`.** Remember that `drawTransparent` will rearrange the order things are drawn in! If you are drawing and redrawing to the same framebuffer for each item, we will not cache its value to use later; it may get overwritten before it draws the first item.

- **Intersecting items will still have rendering issues.** If you have two transparent items that intersect, neither will be fully in front of the other, so the overlapping regions might have artifacts, such as partial occlusion, and antialiased bits leaving a slight "halo" where it occludes the other item. If you anticipate a lot of this, consider trying to split each item up into many smaller items.

  ![image](https://github.com/user-attachments/assets/014b368b-93e3-4656-8686-43715b5369a9)

- **Remember that `drawTransparent()` adds an implicit `push()`/`pop()` around your function.** Since we're rearranging the order of items under the hood, the `push`/`pop` helps make everything more predictable.
