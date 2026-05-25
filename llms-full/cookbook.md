# Cookbook

## Read current editor JSON
**Task**: Get full content payload for persistence.
```ts
import type { Editor } from '@tiptap/core';

function toJson(editor: Editor) {
  return editor.getJSON();
}
```
Use this in `onUpdate` or before save.

## Restore JSON content on load
**Task**: Hydrate editor from stored content.
```ts
import type { Content } from '@tiptap/core';

const raw = localStorage.getItem('edra-content');
const content: Content = raw ? (JSON.parse(raw) as Content) : { type: 'doc', content: [] };
```
Parse only in browser context.

## Toggle editable mode
**Task**: Switch between edit and read modes.
```ts
let editable = true;

$: editor?.setEditable(editable);
```
Matches Notion example behavior.

## Insert image placeholder command
**Task**: Open image insertion flow from custom button.
```ts
function insertImage(editor: Editor) {
  editor.chain().focus().insertImagePlaceholder().run();
}
```
Requires image placeholder extension from Edra editor bundle.

## Insert table quickly
**Task**: Add 3x3 table.
```ts
editor.chain().focus().insertTable({ cols: 3, rows: 3, withHeaderRow: false }).run();
```
Same pattern used in default toolbar commands.

## Add custom toolbar icon command
**Task**: Extend toolbar with your own action.
```svelte
<EdraToolBarIcon
  {editor}
  command={{
    name: 'bold',
    icon: Bold,
    onClick: (ed) => ed.chain().focus().toggleBold().run()
  }}
/>
```
Command object must follow `EdraToolBarCommands` shape.

## Handle drop upload
**Task**: Upload dropped file and return URL.
```ts
async function onDropOrPaste(file: File): Promise<string> {
  const body = new FormData();
  body.append('file', file);
  const res = await fetch('/api/upload', { method: 'POST', body });
  return (await res.json()).url;
}
```
Return string URL, not blob/file.

## Exclude default command groups
**Task**: Hide command groups from toolbar.
```svelte
<EdraToolBar {editor} excludedCommands={['math', 'media']} />
```
Group names map to command registry keys.

## Watch for content corruption
**Task**: Detect content load issues in shadcn editor.
```ts
onContentError: (error) => {
  toast.error('Unable to load the content');
  console.error(error);
}
```
This hook exists in shadcn editor setup.
