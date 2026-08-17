---
{"publish":true,"created":"2026-03-20T12:17:47.513Z","modified":"2026-08-17T03:04:17.709Z"}
---

Properties :
Size, Scaling, Lossy Compression, Browser Support, Transparency Support

#### Formats

###### AVIF (The Performance Leader)

- **Best For:** High-quality photographs, complex hero images, and heavy visuals where load speed is the top priority.
- **Size & Compression:** The most advanced compression available today. It is roughly **50% smaller than JPEG** and **20% smaller than WebP** for the same visual quality.
- **Scaling:** Raster-based (pixels). It will blur if you scale it up beyond its original dimensions.
- **Transparency:** Supports full alpha transparency (smooth edges, translucent shadows).
- **Color Depth:** Supports **10-bit and 12-bit HDR**. This prevents "banding" in gradients (like a sky or sunset) that you often see in JPEGs.
- **Developer Note:** It has a higher "decoding cost," meaning it takes a tiny bit more CPU power to open. Don’t use 50 AVIFs on a single page for low-end mobile devices; stick to WebP for smaller UI thumbnails.

###### WebP (The All-Rounder)

- **Best For:** Daily web development tasks, product images, and replacing PNGs. It is currently the "industry standard" for a balance of speed and compatibility.
- **Size & Compression:** Offers both **Lossy** (for photos) and **Lossless** (for graphics) modes. Usually **25–35% smaller than JPEG**.
- **Scaling:** Raster-based. Requires multiple versions (`srcset`) to look sharp on high-density (Retina) screens.
- **Transparency:** Excellent support for alpha transparency. It is the best replacement for heavy transparent PNGs.
- **Browser Support:** Nearly universal (~97%). It is safe to use as your primary format in 2026.
- **Developer Note:** WebP decodes very fast, making it great for "scroll-heavy" pages like social feeds or e-commerce galleries.

###### SVG (The Vector Specialist)

- **Best For:** Icons, logos, simple illustrations, and decorative UI elements.
- **Size:** Extremely tiny because it is **XML code**, not pixels. The file size depends on the complexity of the paths, not the physical size of the image.
- **Scaling:** **Infinite.** You can scale an SVG from 10px to 10,000px, and it will stay perfectly sharp.
- **Functionality:** Since it's code, you can animate it with CSS/JS and change its colors (e.g., `path { fill: red; }`) directly in your stylesheet.
- **Developer Note:** SVGs are **searchable and accessible**. Screen readers can read the `<title>` tag inside an SVG. Always "minify" your SVG code to remove junk data from design tools like Figma or Illustrator.

###### PNG (The Precision Choice)

- **Best For:** Screenshots, images containing technical text, and situations where "Lossless" quality is non-negotiable.
- **Size:** Generally **very large**. It does not discard any data, which makes it heavy for the web.
- **Scaling:** Raster-based. Heavily affected by pixelation if upscaled.
- **Transparency:** Supports perfect alpha transparency.
- **Developer Note:** In 2026, you should almost always **convert PNGs to WebP (Lossless)**. You get the exact same quality at a fraction of the file size. Only use raw PNGs if a specific legacy tool requires it.

###### JPEG/JPG (The Legacy Fallback)

- **Best For:** Acting as the "safety net" for very old browsers or systems that don't support modern formats.
- **Size:** Moderate. While it was the king for decades, it is now inefficient compared to AVIF/WebP.
- **Scaling:** Raster-based.
- **Transparency:** **None.** It does not support transparent backgrounds.
- **Compression:** Lossy only. If you compress it too much, you see "blocky" artifacts around edges.
- **Developer Note:** Most JPEGs contain **EXIF Metadata** (camera type, GPS, date). Always use a tool to "strip" this metadata before deployment to save 10–15% in file size instantly.

#### Decoding

- **`decoding="async"`**: Tells the browser to decode the image in the background so it doesn't block other tasks like text rendering or animations.
- **`decoding="sync"`**: Forces the browser to finish decoding the image before moving on to anything else—usually avoided unless the image is critical.
- **`img.decode()` (JavaScript)**: A method that returns a **Promise**, allowing you to wait until an image is fully "unpacked" before you actually inject it into the page, ensuring it appears instantly without a flicker
