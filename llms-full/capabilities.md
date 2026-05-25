# Capabilities Deep Dive

## Override File-Drop Behavior at Runtime
**What it enables**: Change upload handling after editor creation without recreating editor.
**Why it matters**: Useful when auth tokens or tenant context changes.
```ts
// Update handler dynamically
editor
  .chain()
  .setHandleFileDrop(async (path: string) => {
    const res = await fetch('/api/upload-from-path', {
      method: 'POST',
      body: JSON.stringify({ path })
    });
    return (await res.json()).url;
  })
  .run();
```
**Key details**:
- Backed by `editor.storage.fileDrop.handler` in custom extension.
- Command returns sync boolean; async effect is intentionally fire-and-forget.

## Click-to-Edit Inline and Block Math
**What it enables**: Clicking rendered equations opens edit state with captured position and LaTeX.
**Why it matters**: Keeps WYSIWYG flow while preserving source-level math editing.
```ts
Mathematics.configure({
  blockOptions: {
    onClick: (node, pos) => {
      blockMathPos = pos;
      blockMathLatex = node.attrs.latex;
    }
  },
  inlineOptions: {
    onClick: (node, pos) => {
      inlineMathPos = pos;
      inlineMathLatex = node.attrs.latex;
    }
  }
});
```
**Key details**:
- Same pattern exists in both headless and shadcn editor components.
- Position is needed to update the exact math node later.

## Built-in Table of Contents Index Stream
**What it enables**: Real-time heading index generation from editor document.
**Why it matters**: You can build docs-like navigation without writing custom ProseMirror traversals.
```ts
TableOfContents.configure({
  getIndex: getHierarchicalIndexes,
  onUpdate: (indexes) => {
    tocItems = indexes;
  },
  scrollParent: () => element || window
});
```
**Key details**:
- Shadcn editor stores `tocItems` state and renders `ToC` component.
- `scrollParent` wiring avoids wrong scroll container behavior.

## Slash Command UI as Svelte Node View
**What it enables**: Command palette style insertion workflow inside editor.
**Why it matters**: Faster block insertion than toolbar-only workflows.
```ts
import slashcommand from '../extensions/slash-command/slashcommand.js';
import SlashCommandList from './components/SlashCommandList.svelte';

const extensions = [
  slashcommand(SlashCommandList)
];
```
**Key details**:
- Slash behavior is extension-driven, not hardcoded in toolbar.
- You can swap `SlashCommandList` UI while keeping extension contract.
