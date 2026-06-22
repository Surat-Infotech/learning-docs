# Images and Media

A web page with only text would be dull. Images, audio, and video bring pages to life. In this lesson you will learn how to add pictures the right way, how to choose the best image format, and how to embed sound, video, and content from other sites.

## What You'll Learn

- How to add an image with `<img>`, `src`, and `alt`
- Why the `alt` text is so important
- How to set image width and height
- The common image formats and when to use each
- How to wrap an image with a caption using `<figure>` and `<figcaption>`
- A first look at responsive images with `srcset`
- How to add audio and video players
- How to embed outside content with `<iframe>`, and how to stay safe

## Adding an Image

Images use the `<img>` element. It is a void element, so it has no closing tag. It needs two key attributes:

- `src` ("source") — the path or URL to the image file.
- `alt` ("alternative text") — a short text description of the image.

```html
<img src="cat.jpg" alt="A ginger cat sleeping on a sofa">
```

The `src` tells the browser which image to load, and the `alt` describes it.

## Why `alt` Matters

The `alt` attribute is not optional in spirit — always write it. It serves two big purposes:

1. **Accessibility.** People who cannot see the image (for example, those using a screen reader) hear the `alt` text read aloud. Good `alt` text lets them understand the picture.
2. **Broken images.** If the image fails to load — wrong path, slow connection — the browser shows the `alt` text instead, so the user still knows what was supposed to be there.

Write `alt` text that describes the *meaning* of the image, as if you were describing it to someone over the phone.

```html
<img src="chart.png" alt="Bar chart showing sales doubling from 2024 to 2025">
```

If an image is purely decorative and adds no information, use an empty alt so screen readers skip it:

```html
<img src="divider.png" alt="">
```

## Width and Height

You can set the image size with the `width` and `height` attributes, measured in pixels.

```html
<img src="logo.png" alt="Company logo" width="200" height="100">
```

Setting both width and height is a good habit. It tells the browser how much space to reserve, so the page does not jump around while the image loads. Later, with CSS, you will control sizing more flexibly, but these attributes still help.

## Choosing an Image Format

Different image **formats** (file types) suit different jobs. Here is a quick guide:

```text
JPG (.jpg)   Photos with many colors. Small file size. No transparency.
PNG (.png)   Sharp graphics, logos, screenshots. Supports transparency.
WebP (.webp) Modern format. Smaller than JPG or PNG with similar quality.
SVG (.svg)   Icons, logos, simple shapes. Stays sharp at any size.
GIF (.gif)   Short, looping animations. Limited colors.
```

A few rules of thumb:

- For a **photograph**, use JPG or WebP.
- For a **logo or icon**, use SVG (it never gets blurry when scaled).
- When you need a **transparent** background, use PNG, WebP, or SVG.
- For a small **animation**, use GIF (or better, a video).

WebP is a great modern default because it gives small file sizes, which means faster pages.

## Image Paths

The `src` follows the same path rules you learned for links. It can be relative or absolute.

```html
<!-- Same folder -->
<img src="photo.jpg" alt="A photo">

<!-- Inside an "images" subfolder -->
<img src="images/photo.jpg" alt="A photo">

<!-- One folder up -->
<img src="../photo.jpg" alt="A photo">

<!-- From another website (absolute) -->
<img src="https://example.com/photo.jpg" alt="A photo">
```

Keeping all your images in an `images/` folder helps stay organized.

## Figures and Captions

Sometimes an image needs a **caption** — a line of text explaining it. HTML has a neat pair for this:

- `<figure>` wraps the image and its caption as one unit.
- `<figcaption>` holds the caption text.

```html
<figure>
  <img src="bridge.jpg" alt="A long suspension bridge over a river at sunset">
  <figcaption>The Golden Gate Bridge at sunset.</figcaption>
</figure>
```

The caption is visible on the page, while the `alt` is for those who cannot see the image. They serve different audiences, so it is fine for them to be a little different.

## A First Look at Responsive Images

Screens come in many sizes, from tiny phones to huge monitors. **Responsive images** let the browser pick the best-sized image for the device, saving data and keeping things sharp. The `srcset` attribute offers the browser a list of choices.

```html
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  alt="A mountain landscape">
```

Here, `srcset` lists three versions of the image along with their widths (`400w` means "400 pixels wide"). The browser automatically chooses the most suitable one. The plain `src` is a fallback for older browsers. Don't worry about mastering this now — just know it exists for when you need it.

## Audio

To add a sound player, use `<audio>` with the `controls` attribute, which shows play and volume buttons.

```html
<audio controls>
  <source src="song.mp3" type="audio/mpeg">
  Your browser does not support audio.
</audio>
```

The `<source>` element points to the file. The text after it shows only if the browser cannot play audio. You can list several `<source>` elements in different formats, and the browser picks the first one it supports.

## Video

Video works almost the same way, using the `<video>` element. You can also set `width` and a `poster` image to show before the video plays.

```html
<video controls width="640" poster="thumbnail.jpg">
  <source src="movie.mp4" type="video/mp4">
  Your browser does not support video.
</video>
```

Common options include `controls` (show play buttons), `autoplay`, `loop`, and `muted`. Be kind to your users: avoid autoplaying video with sound.

## Embedding with `<iframe>`

An **iframe** ("inline frame") embeds another web page *inside* your page, like a small window into another site. It is how you embed a YouTube video or a Google Map.

```html
<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/VIDEO_ID"
  title="A helpful video"
  allowfullscreen>
</iframe>
```

Always include a `title` attribute describing the iframe, for accessibility.

### Safety Notes

An iframe loads content from another website, so be careful:

- Only embed sites you **trust**. A malicious page could try to trick your users.
- Many sites block embedding on purpose, so some pages simply won't show.
- For untrusted content, the `sandbox` attribute restricts what the embedded page can do.

```html
<iframe src="https://example.com" title="Example" sandbox></iframe>
```

When in doubt, only embed well-known, trusted services like YouTube or Google Maps.

## Common Mistakes

- Leaving out the `alt` attribute on an image.
- Writing useless `alt` text like "image" or "picture."
- Using a wrong path in `src`, so the image is broken.
- Using a giant JPG when a small SVG or WebP would be better.
- Autoplaying video with sound, which annoys visitors.
- Embedding an untrusted website in an `<iframe>`.

## Exercises

1. Add an image to a page with a clear, descriptive `alt` text.
2. Wrap that image in a `<figure>` and add a `<figcaption>` caption.
3. Put three images in an `images/` folder and link to them with relative paths.
4. Embed a YouTube video on your page using an `<iframe>` with a `title`.
5. Add a short audio clip with `<audio controls>` and test the play button.

## Key Takeaways

- `<img>` needs a `src` (the file) and an `alt` (a description).
- Good `alt` text helps accessibility and shows when images break.
- Choose the format for the job: JPG/WebP for photos, SVG for logos, PNG for transparency.
- `<figure>` and `<figcaption>` pair an image with a caption.
- `srcset` lets the browser pick the best image size.
- `<audio>` and `<video>` with `controls` add media players.
- `<iframe>` embeds other pages — only embed sites you trust.

---

[← Back to Course Index](../README.md) | [Next: Lists →](05-lists.md)
