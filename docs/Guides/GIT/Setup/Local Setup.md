## How to Set Up a Self-Hosted MkDocs Wiki with Git Integration

**Author**: Brian F.
**Date**: 2025-05-30
**Purpose**: Provide a repeatable guide to deploy and manage a self-hosted MkDocs-based wiki site using Git for version control.

---

### Overview test


This guide describes how to:

* Host an internal documentation wiki using MkDocs
* Version-control all content using Git
* Serve the static site over your internal network
* Automatically rebuild and deploy the site after Git pushes

This setup is intended for internal-only use and contains **no ITAR or proprietary data**.

---

### Requirements

#### Server

* Linux server (e.g., Ubuntu or Rocky Linux)
* SSH access
* Python 3.8+
* nginx or Apache for static file hosting

#### Software

* [MkDocs](https://www.mkdocs.org/): static site generator
* Git: version control

---

### Step-by-Step Setup

#### 1. Create a Git Repository (Bare)

On the server:

```bash
mkdir -p /srv/git/wiki.git
cd /srv/git/wiki.git
git init --bare
```

#### 2. Create a Working Directory

This is where the MkDocs build will happen:

```bash
mkdir -p /srv/wiki-working
cd /srv/wiki-working
git clone /srv/git/wiki.git .
```

#### 3. Set Up MkDocs

Still inside `/srv/wiki-working`:

```bash
pip install mkdocs mkdocs-material
mkdocs new .
```

Customize your `mkdocs.yml` and add documentation to the `docs/` folder.

#### 4. Create a Git Hook to Auto-Build

In `/srv/git/wiki.git/hooks/post-receive`:

```bash
#!/bin/bash
GIT_WORK_TREE=/srv/wiki-working GIT_DIR=/srv/git/wiki.git git checkout -f
cd /srv/wiki-working
mkdocs build --clean
cp -r site/* /var/www/html/wiki
```

Make it executable:

```bash
chmod +x /srv/git/wiki.git/hooks/post-receive
```

#### 5. Configure Web Server (e.g., nginx)

Example nginx block:

```nginx
server {
    listen 80;
    server_name wiki.mycompany.local;
    root /var/www/html/wiki;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Reload nginx:

```bash
sudo systemctl reload nginx
```

#### 6. Push Changes from Your Workstation

From your local copy:

```bash
git remote add origin ssh://wiki-server:/srv/git/wiki.git
git push -u origin main
```

This triggers the `post-receive` hook and updates the site.

---

### Usage Notes

* Edit `.md` files locally using Zettlr or VS Code
* Use `mkdocs serve` for live preview
* Commit and push changes to deploy

---

### Troubleshooting

* Verify permissions on `/srv/wiki-working` and `/var/www/html/wiki`
* Make sure `post-receive` is executable
* Check logs in `/var/log/nginx/` if the site won't load

---

### Optional Enhancements

* HTTPS via Let's Encrypt (internal cert authority)
* Password-protected access via nginx or SSO
* Git web interface (e.g., Gitea or GitWeb)

---

### Summary

This guide sets up a version-controlled, internal documentation wiki suitable for CNC shops, manufacturing processes, or engineering references. It's fast, secure, and requires no external cloud services.

---
