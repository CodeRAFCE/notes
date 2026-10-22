# The Critical Rendering Path (CRP)

The **Critical Rendering Path** is the sequence of steps the browser takes to convert HTML, CSS, and JavaScript into actual pixels on the screen.

```text
HTML Bytes ──> Tokens ──> DOM Tree ──┐
                                     ├──> Render Tree ──> Layout ──> Paint
CSS Bytes  ──> Tokens ──> CSSOM Tree ┘        ▲
                                              │ (can block/alter)
JavaScript ───────────────────────────────────┘
```

---

## 1. DOM Construction (Document Object Model)

The browser receives HTML as raw byte packets across the network and converts them into the DOM tree through a 4-step pipeline:

1. **Bytes to Characters:** Raw bytes (e.g., `3C 62 6F 64 79 3E`) are translated into readable characters based on character encoding (typically UTF-8).
2. **Characters to Tokens:** The browser turns characters into distinct HTML tokens (e.g., `StartTag: html`, `StartTag: body`, `EndTag: body`).
3. **Tokens to Nodes:** Tokens become JavaScript objects ("Nodes") containing their attributes and properties.
4. **Nodes to Tree:** Nodes are linked based on hierarchy into the **DOM tree**.

> **Note:** HTML parsing is **incremental**. The browser starts building the DOM as soon as the first byte chunks arrive—it does not wait for the entire file to download.

---

## 2. CSSOM Construction (CSS Object Model)

When the parser encounters a `<link rel="stylesheet">` or `<style>` block:

* It requests and parses the CSS rules into an inheritance tree: the **CSSOM**.
* **CSS is render-blocking:** The browser stops rendering until the entire CSSOM is built. Painting before CSS is parsed would cause a Flash of Unstyled Content (FOUC).
* Unlike HTML, CSS is **not incremental**. The browser cannot use partial CSS because a rule lower down in the file might override a rule at the top (cascading).

---

## 3. Where JavaScript Fits In

Standard `<script>` tags significantly impact the rendering timeline:

* **JavaScript is parser-blocking:** When the HTML parser hits a `<script>` tag, it completely stops DOM construction, downloads the script, and executes it immediately.
* **JavaScript waits for the CSSOM:** If a script runs while CSS is still downloading, execution pauses until the CSSOM is ready (because JS might query style metrics like `element.offsetWidth`).

### The Solutions:
* `async`: Downloads the script in the background without blocking the parser. Executes as soon as it downloads (interrupting HTML parsing only while running).
* `defer`: Downloads in the background and defers execution until HTML parsing finishes completely.

---

## 4. The Render Tree

Once the DOM and CSSOM are ready, the browser combines them into the **Render Tree**.

* The Render Tree only includes **visible content**.
* **Excluded elements:**
  * `<head>`, `<meta>`, `<script>`, `<title>`.
  * Any element with `display: none` (and its children).
* **Included elements:**
  * Elements with `visibility: hidden` or `opacity: 0` are included because they still take up spatial geometry in the layout.

---

## 5. Layout (Reflow)

The browser traverses the Render Tree to calculate the exact **geometry and coordinates** of every node relative to the device's viewport.

* It computes box models: exact pixel widths, heights, margins, padding, and coordinates $(x, y)$.
* This stage is called **Layout** on initial render, and **Reflow** if element dimensions change later via user interaction or DOM manipulation.

---

## 6. Paint & Composite

1. **Paint:** The browser turns the layout boxes into real pixels on the screen (colors, text, borders, box-shadows).
2. **Composite:** If elements exist on different stacking layers (using `z-index`, 3D transforms, or `will-change`), the browser merges these layers in the correct order to present the final screen image.

---

## Common Misconceptions vs. Reality

| Misconception | Reality |
| :--- | :--- |
| *"Raw HTML is converted directly into the DOM."* | It goes through **Bytes $\rightarrow$ Characters $\rightarrow$ Tokens $\rightarrow$ Nodes $\rightarrow$ DOM**. |
| *"JavaScript only loads after CSS is done."* | JavaScript is preloaded immediately, but synchronous scripts block DOM parsing and must wait for the CSSOM to finish building before executing. |
| *"All DOM elements go into the Render Tree."* | `<head>`, `<script>`, and elements with `display: none` are stripped out completely. |

---

## 30-Second Interview Summary

> *"The Critical Rendering Path is the pipeline the browser uses to turn code into pixels. First, it parses HTML to build the **DOM**. When it sees stylesheets, it parses them to build the **CSSOM**, which is render-blocking. Standard JavaScript blocks the DOM parser unless marked `async` or `defer`, and pauses execution until the CSSOM is built. Once both trees are ready, they merge into the **Render Tree** (which ignores hidden nodes like `display: none`). Finally, the browser runs **Layout** to calculate element coordinates and sizes, and **Paints** the pixels to the screen."*