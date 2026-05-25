# Use Case: Custom Media Upload for File Select, Paste, and Drop

## What You're Trying to Do
You want users to insert images/video/audio, but URLs must come from your upload service, not local temporary paths.

## Complete Working Example
```svelte
<script lang="ts">
  import type { Content, Editor } from '@tiptap/core';
  import { EdraEditor } from '$lib/edra/shadcn/index.js';

  let editor: Editor | undefined;
  let content: Content = { type: 'doc', content: [{ type: 'paragraph' }] };

  async function uploadFile(file: File): Promise<string> {
    const form = new FormData();
    form.append('file', file);
    const res = await fetch('/api/upload', { method: 'POST', body: form });
    if (!res.ok) throw new Error('Upload failed');
    const json = (await res.json()) as { url: string };
    return json.url;
  }

  async function onDropOrPaste(file: File): Promise<string> {
    return uploadFile(file);
  }

  async function onFileSelect(localFilePath: string): Promise<string> {
    // If your file picker gives a path/string, translate it via backend endpoint.
    const res = await fetch('/api/upload-from-path', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ path: localFilePath })
    });
    const json = (await res.json()) as { url: string };
    return json.url;
  }
</script>

<EdraEditor bind:editor {content} {onDropOrPaste} {onFileSelect} />
```

## Step-by-Step Breakdown
1. Define one upload primitive (`uploadFile`) that returns final URL string.
2. Implement `onDropOrPaste(file)` to route drag/paste media through the same primitive.
3. Implement `onFileSelect(path)` separately because this callback receives a string path, not `File`.
4. Pass both callbacks to `EdraEditor`; the internal media extensions and file-drop extension call them.
5. Return resolved URL every time; node views depend on resolved URL for rendering media nodes.

## Variations
- **Client-side only (temporary object URLs)**
```diff
 async function onDropOrPaste(file: File): Promise<string> {
-  return uploadFile(file);
+  return URL.createObjectURL(file);
 }
```

- **Signed upload flow**
```diff
- const res = await fetch('/api/upload', { method: 'POST', body: form });
+ const sign = await fetch('/api/upload-sign');
+ const { putUrl, publicUrl } = await sign.json();
+ await fetch(putUrl, { method: 'PUT', body: file });
+ return publicUrl;
```

- **Reject non-image drops**
```diff
 async function onDropOrPaste(file: File): Promise<string> {
+  if (!file.type.startsWith('image/')) throw new Error('Only images allowed');
   return uploadFile(file);
 }
```

## Gotchas Specific to This Use Case
- Returning empty string breaks user expectations because placeholder remains without valid source.
- Throwing errors without UI feedback hides failure; add toast/error reporting in app layer.
- `onFileSelect` and `onDropOrPaste` inputs have different types; do not share function signature blindly.
