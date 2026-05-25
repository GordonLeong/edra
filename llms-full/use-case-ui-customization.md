# Use Case: Customize Toolbar Commands and UI Flavor

## What You're Trying to Do
You want Edra’s core editor behavior but need custom toolbar controls and ability to switch between headless and styled UI.

## Complete Working Example
```svelte
<script lang="ts">
  import type { Content, Editor } from '@tiptap/core';
  import { EdraEditor, EdraToolBar } from '$lib/edra/headless/index.js';
  import EdraToolBarIcon from '$lib/edra/headless/components/ToolBarIcon.svelte';
  import Bold from '@lucide/svelte/icons/bold';

  let editor: Editor | undefined;
  let content: Content = { type: 'doc', content: [{ type: 'paragraph' }] };
</script>

{#if editor}
  <EdraToolBar {editor} excludedCommands={['math', 'table']}>
    <div class="px-2 text-xs">Custom block</div>
    <EdraToolBarIcon
      {editor}
      command={{
        name: 'bold',
        icon: Bold,
        tooltip: 'Bold',
        onClick: (ed) => ed.chain().focus().toggleBold().run(),
        isActive: (ed) => ed.isActive('bold')
      }}
    />
  </EdraToolBar>
{/if}

<EdraEditor bind:editor {content} />
```

## Step-by-Step Breakdown
1. Import from `headless/index.js` when you want unstyled primitives.
2. Use `excludedCommands` to remove entire default command groups.
3. Slot custom content as children inside `EdraToolBar`.
4. Provide command object with `name`, `icon`, and `onClick`; this mirrors internal command schema.
5. Use Tiptap chain syntax `ed.chain().focus().toggleBold().run()` so command is queued and executed safely.

## Variations
- **Switch to shadcn UI**
```diff
- import { EdraEditor, EdraToolBar } from '$lib/edra/headless/index.js';
+ import { EdraEditor, EdraToolBar } from '$lib/edra/shadcn/index.js';
```

- **Disable editor while keeping UI mounted**
```diff
- <EdraEditor bind:editor {content} />
+ <EdraEditor bind:editor {content} editable={false} />
```

- **Add custom heading command**
```diff
 onClick: (ed) => ed.chain().focus().toggleBold().run(),
+turnInto: (ed, _node, pos) => ed.chain().setNodeSelection(pos).setHeading({ level: 2 }).run(),
```

## Gotchas Specific to This Use Case
- If custom command omits `focus()`, command can fail when editor selection is not active.
- If you exclude command groups that your users expect (e.g., media), slash command may still expose related insertions unless separately customized.
- Headless components require your own styling; otherwise controls render unstyled.
