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
```

#### `style.css`

```css
body {
  font-family: sans-serif;
  background: #1e1e1e;
  color: #eee;
  padding: 2rem;
  text-align: center;
}

#dropzone {
  border: 2px dashed #666;
  padding: 4rem;
  margin: 2rem auto;
  width: 80%;
  max-width: 600px;
  transition: 0.3s;
}

#dropzone.hover {
  border-color: #00ccff;
  background-color: #333;
}

pre {
  margin-top: 2rem;
  font-size: 1.2rem;
  background: #333;
  padding: 1rem;
  border-radius: 5px;
  overflow-x: auto;
}
```

#### `server.py`

```python
from flask import Flask, request, jsonify
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)
UPLOAD_FOLDER = os.path.join(os.getcwd(), "docs/images")
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/upload', methods=['POST'])
def upload_file():
    file = request.files['file']
    filename = secure_filename(file.filename)
    save_path = os.path.join(UPLOAD_FOLDER, filename)
    file.save(save_path)
    return jsonify({"name": filename})

@app.route('/')
def index():
    return app.send_static_file('index.html')

@app.route('/style.css')
def css():
    return app.send_static_file('style.css')

if __name__ == '__main__':
    app.static_folder = '.'
    app.run(debug=True)
```

---

## ▶️ How to Use

1. Make sure you have Flask installed:

    ```bash
    pip install flask
    ```

2. Run the server:

    ```bash
    python server.py
    ```

3. Open your browser to [http://127.0.0.1:5000](http://127.0.0.1:5000)

4. Drop an image into the box

5. It saves the image to `docs/images/` and copies the Markdown to your clipboard:

    ```markdown
    ![example.png](../../images/example.png)
    ```

---

## ✅ Tips

- Works great for MkDocs setups where you're editing files in deep folders
- No more typing `../../images/foo.png` manually!
- You can extend this to rename files, resize them, or even upload to S3 if you want

---

> 👷‍♂️ Built for local use — no external services or internet connection required.
