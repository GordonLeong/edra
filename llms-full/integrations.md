# Integration Points

## SvelteKit + localStorage persistence
**Use case**: Keep draft content across reloads in a SvelteKit route.
```svelte
<script lang="ts">
  import { browser } from '$app/environment';
  import type { Content } from '@tiptap/core';

  let content: Content = { type: 'doc', content: [] };

  if (browser) {
    const raw = localStorage.getItem('edra-content');
    if (raw) content = JSON.parse(raw) as Content;
  }

  function persist(next: Content) {
    content = next;
    localStorage.setItem('edra-content', JSON.stringify(next));
  }
</script>
```
**Notes**: Browser guard is mandatory in SSR routes.

## Tiptap editor command chaining
**Use case**: Add custom buttons that behave like built-in toolbar actions.
```ts
function toggleH2(editor: Editor) {
  editor.chain().focus().toggleHeading({ level: 2 }).run();
}

function insertInlineMath(editor: Editor, latex: string) {
  editor.chain().focus().insertInlineMath({ latex }).run();
}
```
**Notes**: Always call `focus()` before mutation commands for reliable execution.

## Backend upload service
**Use case**: Convert pasted or dropped files into persistent CDN URLs.
```ts
async function onDropOrPaste(file: File): Promise<string> {
  const form = new FormData();
  form.append('file', file);
  const res = await fetch('/api/upload', { method: 'POST', body: form });
  if (!res.ok) throw new Error('upload failed');
  return (await res.json()).url;
}
```
**Notes**: Same callback is used by image/video/audio extended nodes when user pastes or drops files.
