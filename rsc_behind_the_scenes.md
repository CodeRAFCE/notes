# How React Server Components Render Behind the Scenes

React Server Components (RSC) still use the Virtual DOM, but the generation and reconciliation of that tree are split between the server runtime and the browser.

---

## 1. On the Server (Execution & Tree Generation)

1. **Execution**: The server runtime (such as Node.js or an Edge runtime) executes Server Component functions. It resolves `async/await` operations, queries databases, and handles private credentials.
2. **Virtual DOM Construction**: React creates standard Virtual DOM objects (React elements) describing the UI.
3. **Component Separation**:
   - **Server Components**: Evaluated recursively down to primitive elements (`div`, `span`, `p`, etc.).
   - **Client Components (`'use client'`)**: React pauses execution at this boundary. It does not run the component's internal logic. Instead, it places a **module reference placeholder** that stores:
     - The path/URL to the client JavaScript bundle.
     - The serialized props passed to that component.

---

## 2. Over the Network (The RSC Payload)

Because live JavaScript functions and memory references cannot travel across HTTP, React serializes this tree into a streaming format known as the **RSC Payload**.

The payload is a compact stream of serialized instructions that looks conceptually like this:

```text
- Render <div>
  - Render <h1> with text "User Profile"
  - Placeholder for Client Component "LikeButton.js" with props { "id": 10 }
```

- Zero server logic, database drivers, or private secrets enter this stream.
- Only the UI structure, data primitives, and client module references are sent.

---

## 3. In the Browser (Deserialization & Reconciliation)

1. **Parsing**: The browser runtime reads the incoming RSC Payload stream line-by-line and deserializes it back into live **Virtual DOM objects** in memory.
2. **Client Module Fetching**: When React encounters a client reference placeholder, it downloads and executes the corresponding client bundle (e.g., `LikeButton.js`), if not already loaded.
3. **Tree Integration**: The client component is instantiated and inserted into the open slot in the Virtual DOM.
4. **Reconciliation & Commit**: React diffs this complete Virtual DOM against the real browser DOM and commits the updates to the screen.

---

## Key Takeaway

| Question | Answer |
| :--- | :--- |
| **Is the Virtual DOM still used?** | **Yes.** |
| **Where does it live?** | The server creates the initial tree description, serializes it to the **RSC Payload**, and the browser reconstructs it back into an active Virtual DOM. |
| **Why does this matter?** | Because updates stream as Virtual DOM nodes (rather than raw HTML strings), subsequent server updates can be diffed into the existing page **without wiping out client-side state, form inputs, or cursor focus**. |