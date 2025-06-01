# 🖼️ Drag-and-Drop Image → Markdown Inserter

This tool lets you drag an image onto a local web page, and it will:

1. Save the image into `docs/images/`
2. Generate the correct Markdown for linking the image
3. Automatically copy the Markdown to your clipboard ✅

---

## 🔧 Setup

### 1. 📁 Project Structure

Create a folder anywhere (e.g., `~/image-dropper/`) and inside it, put these files:

#### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Image Drop to Markdown</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Drop image to generate Markdown</h1>
  <div id="dropzone">Drop image here</div>
  <pre id="output"></pre>

  <script>
    const dropzone = document.getElementById('dropzone');
    const output = document.getElementById('output');

    dropzone.addEventListener('dragover', e => {
      e.preventDefault();
      dropzone.classList.add('hover');
    });

    dropzone.addEventListener('dragleave', () => {
      dropzone.classList.remove('hover');
    });

    dropzone.addEventListener('drop', async e => {
      e.preventDefault();
      dropzone.classList.remove('hover');

      const file = e.dataTransfer.files[0];
      const formData = new FormData();
      formData.append('file', file);

      const res = await fetch('/upload', {
        method: 'POST',
        body: formData
      });

      const result = await res.json();
      const markdown = `![${result.name}](../../images/${result.name})`;
      output.textContent = markdown;

      navigator.clipboard.writeText(markdown)
        .then(() => alert("Copied to clipboard!"))
        .catch(() => alert("Markdown ready, but couldn't copy to clipboard."));
    });
  </script>
</body>
</html>
