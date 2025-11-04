# 🎨 Modern Theming with React + Vite + CSS Modules

## Project Structure

```plaintext
src/
  themes/
    blue/
      light.css
      dark.css
    red/
      light.css
      dark.css
  components/
    Button/
      Button.jsx
      Button.module.css
      _Button.theme-blue.css
      _Button.theme-red.css
  ThemeProvider.jsx
  App.jsx
```

---

## Why `.module.css`

| File Type              | Scope                 | Purpose                            |
| ---------------------- | --------------------- | ---------------------------------- |
| `*.module.css`         | **Local / Component** | Scoped CSS with unique class names |
| `*.css` (no `.module`) | **Global**            | Theme variables or global resets   |

Vite automatically detects `.module.css` → compiles to unique hashed class names for **CSS Modules**.

---

## Co-locate component styles

Each React component lives in its own folder:

```
Button/
  Button.jsx
  Button.module.css
```

✅ Benefits:

- Styles stay close to logic
- Safer refactors
- No global class conflicts

---

## Base component CSS module

**`Button.module.css`**

```css
@import "./_Button.theme-blue.css";
@import "./_Button.theme-red.css";

.button {
  border-radius: 6px;
  padding: 0.5rem 1rem;
  border: none;
  cursor: pointer;
  font-weight: 500;
  background-color: var(--surface-button-bg);
  color: var(--text-button);
  transition: background-color 0.2s ease;
}

.button:hover {
  background-color: var(--surface-button-hover-bg);
}
```

**`Button.jsx`**

```jsx
import styles from "./Button.module.css";

export default function Button({ children }) {
  return <button className={styles.button}>{children}</button>;
}
```

---

## Theme variables

Each theme defines its own **CSS custom properties** for color, surface, and text.

**`blue/light.css`**

```css
:root {
  --surface-bg: #ffffff;
  --surface-border: #d4d9e2;
  --surface-button-bg: #e6efff;
  --surface-button-hover-bg: #d3e3ff;
  --text-primary: #0d1b2a;
  --text-button: #0a3b8c;
}
```

**`blue/dark.css`**

```css
:root {
  --surface-bg: #1b1e24;
  --surface-border: #2b2f36;
  --surface-button-bg: #243a6b;
  --surface-button-hover-bg: #3454a1;
  --text-primary: #f5f6fa;
  --text-button: #d8e3ff;
}
```

**`red/light.css`**

```css
:root {
  --surface-bg: #ffffff;
  --surface-border: #e2d1d1;
  --surface-button-bg: #ffe6e6;
  --surface-button-hover-bg: #ffd3d3;
  --text-primary: #330000;
  --text-button: #a30000;
}
```

**`red/dark.css`**

```css
:root {
  --surface-bg: #1e1212;
  --surface-border: #2b1b1b;
  --surface-button-bg: #6b2424;
  --surface-button-hover-bg: #a13434;
  --text-primary: #fbecec;
  --text-button: #ffb8b8;
}
```

---

## Theme-specific overrides

Optionally provide per-theme overrides that enhance visual identity.

**`_Button.theme-blue.css`**

```css
:global(.theme-blue) .button {
  background-color: var(--surface-button-bg);
  color: var(--text-button);
  border: 1px solid var(--surface-border);
}

:global(.theme-blue.theme-dark) .button {
  background-color: var(--surface-button-bg);
  color: var(--text-button);
  border: 1px solid var(--surface-border);
}
```

**`_Button.theme-red.css`**

```css
:global(.theme-red) .button {
  background: linear-gradient(to right, #e74c3c, #f28b82, #e74c3c);
  border: 1px solid #c0392b;
  color: #fff;
}
```

---

## Theme provider (dynamic switching)

**`ThemeProvider.jsx`**

```jsx
import { useEffect } from "react";

export function ThemeProvider({ theme, mode, children }) {
  useEffect(() => {
    const root = document.documentElement;

    // Clear old classes
    root.classList.remove(
      "theme-blue",
      "theme-red",
      "theme-dark",
      "theme-light"
    );

    // Add current theme + mode
    root.classList.add(`theme-${theme}`);
    root.classList.add(`theme-${mode}`);
  }, [theme, mode]);

  return children;
}
```

**Usage**

```jsx
import { ThemeProvider } from "./ThemeProvider";
import Button from "./components/Button/Button";
import { useState } from "react";

export default function App() {
  const [theme, setTheme] = useState("blue");
  const [mode, setMode] = useState("light");

  return (
    <ThemeProvider theme={theme} mode={mode}>
      <div className="app-container">
        <Button>Click Me</Button>
        <button
          onClick={() => setTheme((t) => (t === "blue" ? "red" : "blue"))}
        >
          Switch Theme
        </button>
        <button
          onClick={() => setMode((m) => (m === "light" ? "dark" : "light"))}
        >
          Switch Mode
        </button>
      </div>
    </ThemeProvider>
  );
}
```

---

## Summary Table

| Feature                 | Approach                                            |
| ----------------------- | --------------------------------------------------- |
| Component styling       | `.module.css` (CSS Modules)                         |
| Shared variables        | CSS custom properties (`--surface-bg`, etc.)        |
| Theming                 | Separate theme folders per color scheme             |
| Style overrides         | `_Component.theme-[name].css` optional imports      |
| Runtime theme switching | Toggle global classes (`.theme-blue.theme-dark`)    |
| File naming             | `_underscore` → partials not directly imported      |
| Folder layout           | Keep component CSS beside its `.jsx` implementation |

---

## Recommended Setup

- **Use CSS Modules** for local scoping.
- **Define neutral design tokens** (surface, text, accent) with CSS variables.
- **Use global theme classes** for color scheme switching.
- **Keep overrides modular**, one file per theme.
- **Avoid Sass** — CSS Modules and variables are enough for scalable theming.
