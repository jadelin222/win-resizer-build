# Making a Unity WebGL Build Fullscreen

Every time Unity exports a new WebGL build, the generated `index.html` uses a fixed canvas size (e.g. 960x600). Here's what to change to make it fill the browser window.

## Changes to `index.html`

### 1. Add a viewport meta tag (in `<head>`)

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
```

### 2. Add fullscreen CSS (in `<head>`)

```html
<style>
  * { margin: 0; padding: 0; }
  html, body { overflow: hidden; width: 100%; height: 100%; }
</style>
```

### 3. Fix the canvas element

**Before (Unity default):**
```html
<canvas id="unity-canvas" width=960 height=600 tabindex="-1" style="width: 960px; height: 600px; background: #231F20"></canvas>
```

**After:**
```html
<canvas id="unity-canvas" tabindex="-1" style="width: 100vw; height: 100vh; position: fixed; top: 0; left: 0; background: #231F20;"></canvas>
```

- Remove the `width=960 height=600` attributes
- Replace `width: 960px; height: 600px` with `width: 100vw; height: 100vh`
- Add `position: fixed; top: 0; left: 0;`

### 4. Remove the mobile-only resize block

Delete this entire block (Unity adds it by default):

```js
if (/iPhone|iPad|iPod|Android/i.test(navigator.userAgent)) {
  // ... mobile resize code ...
}
```

It's not needed because the CSS now handles all devices.

### 5. Add `devicePixelRatio: 1` to the config

In the `createUnityInstance` options, add:

```js
devicePixelRatio: 1,
```

This prevents rendering at 2x resolution on high-DPI screens, which can hurt performance.

### 6. Do NOT set `matchWebGLToCanvasSize: false`

Leave it at the default (`true`). Setting it to `false` will cause Unity to render at a tiny internal resolution even though the canvas looks fullscreen.

## Permanent Fix (Recommended)

To avoid editing `index.html` after every build, create a **custom WebGL template** in your Unity project:

1. Create folder: `Assets/WebGLTemplates/Fullscreen/`
2. Save a corrected `index.html` there, replacing build-specific filenames with Unity template variables:
   - `{{{ LOADER_FILENAME }}}` — the loader .js file
   - `{{{ DATA_FILENAME }}}` — the .data file
   - `{{{ FRAMEWORK_FILENAME }}}` — the framework .js file
   - `{{{ CODE_FILENAME }}}` — the .wasm file
3. In Unity: **Edit > Project Settings > Player > WebGL > Resolution and Presentation**, select "Fullscreen"
4. Every future build will use your fullscreen template automatically
