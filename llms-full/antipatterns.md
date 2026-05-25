# Antipatterns & Gotchas

## Rendering command UI before editor exists
**What you might do**:
```svelte
<EdraToolBar {editor} />
<EdraBubbleMenu {editor} />
<EdraEditor bind:editor {content} />
```
**What goes wrong**: Toolbar actions execute against `undefined` editor during initial render.
**Do this instead**:
```svelte
{#if editor && !editor.isDestroyed}
  <EdraToolBar {editor} />
  <EdraBubbleMenu {editor} />
{/if}
<EdraEditor bind:editor {content} />
```
**Why**: Edra initializes editor in `onMount`, so first render has no instance.

## Assuming `onFileSelect` gets a `File`
**What you might do**:
```ts
async function onFileSelect(file: File): Promise<string> {
  return uploadFile(file);
}
```
**What goes wrong**: Type mismatch; Edra `onFileSelect` contract expects `string` path.
**Do this instead**:
```ts
async function onFileSelect(path: string): Promise<string> {
  const res = await fetch('/api/upload-from-path', { method: 'POST', body: JSON.stringify({ path }) });
  return (await res.json()).url;
}
```
**Why**: File selector callback in Edra types is string-based.

## Skipping `focus()` in toolbar commands
**What you might do**:
```ts
onClick: (editor) => editor.chain().toggleBold().run()
```
**What goes wrong**: Command may no-op when selection focus is outside editor.
**Do this instead**:
```ts
onClick: (editor) => editor.chain().focus().toggleBold().run()
```
**Why**: Tiptap commands typically need active selection context.

## Treating async file-drop command as awaited
**What you might do**:
```ts
await editor.chain().handleFileDrop(path).run();
```
**What goes wrong**: `run()` returns boolean; actual handler is launched via `void` and not awaited.
**Do this instead**:
```ts
editor.chain().setHandleFileDrop(async (path) => uploadPath(path)).run();
// Track upload lifecycle in your own callback/UI state.
```
**Why**: Extension command intentionally wraps async call without promise chaining.
