---
title: "Native HTML tricks - 4 elements that replace useState"
date: "2026-05-04"
description: "Disclosure, modals, color pickers, and popovers - native HTML that skips the React boilerplate"
---
## Before you reach for `useState`, check what HTML already does
The React reflex when you see a toggle is automatic: `const [open, setOpen] = useState(false)`, a button with `onClick`, conditional render, maybe a `useEffect` with `addEventListener('keydown', ...)` so Escape closes it. Five imports, twenty lines, a custom hook to wrap it up.

Then one day you find out the browser handles most of these things natively. Has done for years. No React, no state, no JavaScript at all. This post is about those elements. Each one comes with a code block and a working example below it - click around and think about how many lines of React you'd write to get the same thing.

### First Tip
#### Use `<details>` and `<summary>` for any disclosure UI - zero JS.
Why?
These are native HTML tags. The browser handles the toggle for you. Click `<summary>`, the browser flips the `open` attribute on `<details>`, the children are hidden when `open=false` and visible when `open=true`. It works with the keyboard (Enter/Space), screen readers know it's a disclosure widget, and it works even if your JavaScript blows up. Your React version will be 12 lines and have worse accessibility. This one is 4 lines and ships more features.

```html
<details>
  <summary>Click to expand</summary>
  <p>Here's the hidden content. It shows up when you click the summary.</p>
</details>
```

And here it is live (click it):

<details>
  <summary>Click to expand</summary>
  <p>Here's the hidden content. It shows up when you click the summary.</p>
</details>

### Second Tip
#### Use `<input type="color">` instead of pulling in a color picker library.
Why?
Everyone has seen those color picker libraries - extra package, a few kilobytes, custom modal, custom canvas, custom state. Meanwhile the browser has a native system picker - the same one Photoshop and Figma use - and you launch it with a single attribute. It returns a hex string in `value`, plays nicely with forms, and on mobile it opens the OS-level picker.

```html
<input type="color" value="#d4af37">
```

Live (click the swatch):

<input type="color" value="#d4af37">

### Third Tip
#### Use `<dialog>` instead of building your own modal from scratch.
Why?
A typical React modal needs a portal to `document.body` to escape a parent's `overflow:hidden`, a custom backdrop, manual Escape handling, a focus trap, and focus restoration on close. That's hours of work or a dependency like radix-dialog. The native `<dialog>` element ships all of that: the `::backdrop` pseudo-class for styling the overlay, Escape closes the modal, focus is trapped inside, and on close it returns to where it was. You open it with `dialog.showModal()` (one line of JS), close it with `dialog.close()` or a form with `method="dialog"`.

```html
<dialog id="myModal">
  <p>This is a modal!</p>
  <form method="dialog">
    <button>Close</button>
  </form>
</dialog>

<button onclick="document.getElementById('myModal').showModal()">
  Open modal
</button>
```

Here's what a dialog looks like from the inside (rendered statically with the `open` attribute - click the button to close it):

<dialog open style="border: 1px solid #d4af37; background: #262626; color: #d4d4d4; padding: 1.5rem; border-radius: 4px;">
  <p>This is a native &lt;dialog&gt;. Backdrop, Escape, focus trap - all for free.</p>
  <form method="dialog">
    <button style="background: #d4af37; color: #1a1a1a; border: none; padding: 0.5rem 1rem; cursor: pointer; border-radius: 2px;">Close</button>
  </form>
</dialog>

Opening a dialog still needs one line of JS (`.showModal()`). If you want zero JS at all, that's the next tip.

### Fourth Tip
#### Use the Popover API for tooltips, menus, and dropdowns - no JS required.
Why?
HTML 2024 added the popover API. Two attributes: `popovertarget` on the button and `popover` on the element you want to show. The browser wires them together, handles the toggle, closes on outside click, closes on Escape. Perfect for tooltips, menus, dropdowns, and small modal-like panels. It works in every modern browser (Chrome, Safari, Firefox - all shipped in 2024).

```html
<button popovertarget="myPopover">Show popover</button>
<div popover id="myPopover">
  Hello from a popover!
</div>
```

Live (click it - notice that an outside click and Escape both close it):

<button popovertarget="myPopover" style="background: #d4af37; color: #1a1a1a; border: none; padding: 0.5rem 1rem; cursor: pointer; border-radius: 2px;">Show popover</button>
<div popover id="myPopover" style="border: 1px solid #d4af37; background: #262626; color: #d4d4d4; padding: 1rem; border-radius: 4px;">
  Hello from a popover. Click outside or hit Escape to close. Zero lines of JS.
</div>

### The rule
Before you reach for state in React, check whether HTML or CSS already does the thing. Less code means fewer bugs, better accessibility (screen readers understand native semantic elements out of the box), graceful degradation when JS fails, and a smaller bundle.

Other cases in the same spirit:
- `<dialog>` instead of a custom modal (above)
- `<input type="color">`, `<input type="date">`, `<input type="range">` instead of custom widgets
- `<datalist>` instead of a custom autocomplete
- `<details>` as a FAQ section or accordion
- CSS `:hover`, `:focus-within`, `:has()` instead of `useState` for hover/focus tracking
- CSS `scroll-snap` instead of a carousel library
- HTML `<form>` with `required`, `pattern`, `type="email"` instead of Yup/Zod for simple validation
- The `loading="lazy"` attribute on `<img>` instead of a custom IntersectionObserver

The general rule: if a piece of functionality is common enough that the HTML spec covers it, there's probably a native solution. Check first, install a package second.
