# Use Case: Basic Editor Setup with Persisted JSON Content

## What You're Trying to Do
You want a working rich text editor in a Svelte page, with toolbar controls and content persistence so reloading the page keeps prior edits.

## Complete Working Example
```svelte
<script lang="ts">
  import { browser } from '$app/environment';
  import type { Content, Editor } from '@tiptap/core';
  import { EdraEditor, EdraToolBar, EdraBubbleMenu } from '$lib/edra/shadcn/index.js';

  let editor: Editor | undefined;
  let content: Content = {
    type: 'doc',
    content: [{ type: 'paragraph', content: [{ type: 'text', text: 'Hello Edra' }] }]
  };

  if (browser) {
    const saved = localStorage.getItem('edra-content');
    if (saved) content = JSON.parse(saved) as Content;
  }

  function onUpdate() {
    content = editor?.getJSON() as Content;
    localStorage.setItem('edra-content', JSON.stringify(content));
  }
</script>

{#if editor && !editor.isDestroyed}
  <EdraToolBar {editor} />
  <EdraBubbleMenu {editor} />
{/if}

<div class="h-[30rem] overflow-y-auto border p-4">
  <EdraEditor bind:editor {content} {onUpdate} editable={true} autofocus={true} />
</div>
```

## Step-by-Step Breakdown
1. Import `EdraEditor` as the required editor surface, then optional UI helpers (`EdraToolBar`, `EdraBubbleMenu`).
2. Keep both `editor` instance and `content` JSON in local state because Edra expects external state ownership.
3. Load stored JSON only in browser context (`if (browser)`) so SSR does not access `localStorage`.
4. Use `onUpdate` callback to capture every editor transaction and replace `content` with `editor.getJSON()`.
5. Persist JSON immediately so state recovery is deterministic after reload.
6. Guard toolbar/bubble with `editor && !editor.isDestroyed` to avoid calling commands on missing editor.
7. Bind `editor` from `EdraEditor` using `bind:editor`; this is how parent components gain command access.

## Variations
- **Read-only mode**
```diff
- <EdraEditor bind:editor {content} {onUpdate} editable={true} autofocus={true} />
+ <EdraEditor bind:editor {content} {onUpdate} editable={false} autofocus={false} />
```

- **Headless UI instead of shadcn**
```diff
- import { EdraEditor, EdraToolBar, EdraBubbleMenu } from '$lib/edra/shadcn/index.js';
+ import { EdraEditor, EdraToolBar, EdraBubbleMenu } from '$lib/edra/headless/index.js';
```

- **Save remotely instead of localStorage**
```diff
 function onUpdate() {
   content = editor?.getJSON() as Content;
-  localStorage.setItem('edra-content', JSON.stringify(content));
+  void fetch('/api/content', { method: 'POST', body: JSON.stringify(content) });
 }
```

## Gotchas Specific to This Use Case
- If you skip `bind:editor`, toolbar commands cannot execute because no editor reference exists.
- If you parse saved content without validation and storage is corrupted, initialization can fail; guard parse errors in production.
- If you render toolbar without `editor` guard, null references occur before `onMount` initializes Tiptap.
