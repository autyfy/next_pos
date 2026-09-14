# POS Modal Input Click Issue — Root Cause & Fix

## Symptom
- All text inputs inside POS modals unclickable on Frappe Cloud
- Tab key works to focus inputs
- Buttons inside modals work fine
- All browsers affected (Chrome, Safari, desktop, mobile)
- Local machine works fine — only Frappe Cloud affected

## Root Cause
The **PWA service worker on Frappe Cloud was serving old cached JS** (`index-B6U9YnGj.js`).

The old JS had `@pointerdown.prevent` (Vue's `.prevent` modifier) on a **parent wrapper element** in `POSSale.vue`. This called `preventDefault()` on every `pointerdown` event across the entire page — including inside all modals. Since `mousedown` and `focus` are suppressed when `pointerdown.preventDefault()` is called, inputs could not be focused by click.

Buttons still worked because they respond to `click` events which were not suppressed.

## Diagnostic Steps That Identified It

1. `elementFromPoint` → confirmed no overlay, input itself was at click location
2. Event listeners on input → `mousedown`/`focus`/`mouseup` never fired on Cloud; only `click` fired
3. `pointerdown` listener on document → `pointerdown` DID fire on INPUT
4. Override `preventDefault` on pointerdown → stack trace showed:
   ```
   prevent @ index-B6U9YnGj.js:18
   t.<computed>.t.<computed> @ index-B6U9YnGj.js:18
   ```
   `prevent` is Vue's built-in `.prevent` modifier — confirming old cached JS with `@pointerdown.prevent` on a parent element

## Current Code (Fixed)
`POSSale.vue` line 185 — the resize divider:
```html
<div @pointerdown="startResize" :class="{ 'pointer-events-none opacity-0': uiStore.isAnyDialogOpen }">
```
- `@pointerdown` without `.prevent` — `preventDefault()` called only inside `startResize()` function
- `pointer-events-none` applied to divider when any dialog is open — so divider never intercepts clicks during modals

## Fix
Deploy latest `pos_next` from GitHub to Frappe Cloud. New build generates new asset hashes → service worker detects changes and updates on next user visit.

Users still seeing the issue after deploy: DevTools → Application → Service Workers → Unregister → Reload.
