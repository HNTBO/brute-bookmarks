# Dev Notes

## Modal Click-Outside-to-Close: The Drag Selection Problem

**Date**: 2025-01-05
**File**: `public/index.html` (lines ~2209-2231)

### The Problem

When a modal uses a simple `click` event on the backdrop to close, it creates a frustrating UX bug:

1. User starts selecting text inside an input field (mousedown inside modal content)
2. User drags to select all text, accidentally moving cursor outside the modal
3. User releases mouse button (mouseup on backdrop)
4. Modal closes unexpectedly

This happens because a `click` event fires when mouseup occurs, and the event target is the backdrop.

### The Naive Implementation (Problematic)

```javascript
document.getElementById('bookmark-modal').addEventListener('click', (e) => {
    if (e.target.id === 'bookmark-modal') closeBookmarkModal();
});
```

This closes the modal whenever a click completes on the backdrop, regardless of where the click started.

### The Fix: Track Mousedown Origin

```javascript
let bookmarkModalMouseDownOnBackdrop = false;

document.getElementById('bookmark-modal').addEventListener('mousedown', (e) => {
    bookmarkModalMouseDownOnBackdrop = (e.target.id === 'bookmark-modal');
});

document.getElementById('bookmark-modal').addEventListener('mouseup', (e) => {
    if (bookmarkModalMouseDownOnBackdrop && e.target.id === 'bookmark-modal') {
        closeBookmarkModal();
    }
    bookmarkModalMouseDownOnBackdrop = false;
});
```

### How It Works

- **Mousedown on backdrop** → flag = `true`
- **Mousedown inside modal content** → flag = `false`
- **Mouseup**: Only close if flag is `true` AND mouseup target is backdrop
- **Reset flag** after every mouseup

### Behavior Matrix

| Mousedown Location | Mouseup Location | Result |
|-------------------|------------------|--------|
| Backdrop | Backdrop | Modal closes (intentional click) |
| Backdrop | Modal content | Modal stays open |
| Modal content | Backdrop | Modal stays open (drag selection) |
| Modal content | Modal content | Modal stays open |

### Why This Matters

This is a common UX pattern issue. Many modal implementations get this wrong. The fix is simple but easy to overlook during initial development.

### Related Concepts

- Event bubbling and capturing
- Difference between `click`, `mousedown`, `mouseup` events
- Modal/dialog accessibility patterns
- Pointer events vs mouse events (for touch support)

### Further Reading

- Consider using `pointerdown`/`pointerup` instead of mouse events for unified touch/mouse handling
- Some UI libraries use a "clickStartedInside" pattern for this exact issue
- React Aria and Radix UI handle this edge case in their dialog primitives
