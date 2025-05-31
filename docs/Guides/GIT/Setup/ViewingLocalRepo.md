## Viewing Git Repositories Locally Without GitHub

**Author**: Brian F.
**Date**: 2025-05-30
**Purpose**: Provide easy, internal-only options for browsing Git repositories with or without a web interface.

---

### Overview

In a secure or air-gapped CNC environment, it's important to have simple, reliable ways to view Git commit history, diffs, and file changes without relying on GitHub or a full Git server like GitLab.

This guide explains several local tools for viewing Git history and comparing file changes. These methods work entirely offline and are suitable for internal documentation, CNC program versioning, and compliance tracking.

---

### 🔧 Option 1: Git InstaWeb (One-Liner Git Viewer)

#### Description

Starts a temporary local website to browse your Git repo with a GUI, using the `git instaweb` command.

#### Setup

Install Ruby (required by WEBrick):

```bash
sudo apt install ruby
```

Launch the viewer from any Git repo:

```bash
git instaweb --httpd=webrick
```

Then visit:

```
http://localhost:1234
```

To shut it down:

```bash
pkill -f instaweb
```

#### Pros

* Built into Git
* Lightweight
* No server config needed

#### Cons

* Basic UI
* Requires Ruby

---

### 📺 Option 2: `tig` (Terminal Git Viewer)

#### Description

`tig` is a full-screen terminal interface to Git. Great for CLI users.

#### Installation

```bash
sudo apt install tig
```

#### Usage

```bash
tig
```

#### Features

* Browse commits, diffs, and file changes
* Easily view logs and blame
* Fully keyboard-driven

#### Pros

* Fast, no dependencies
* Great for quick log reviews

#### Cons

* Terminal only, no web UI

---

### 🌐 Option 3: Gitea (Optional Web Interface for Teams)

#### Description

[Gitea](https://gitea.io) is a lightweight, self-hosted Git service with a GitHub-style web UI.

#### Highlights

* Host Git repositories internally
* Web-based editing, diffs, commit history
* Great for multi-user teams

#### Installation (optional, not required for local-only usage)

```bash
# Example Docker-based install
docker run -d --name=gitea -p 3000:3000 -v /srv/gitea:/data gitea/gitea:latest
```

#### Pros

* Fully featured web UI
* Good for future scaling

#### Cons

* Requires setup and server to run continuously

---

### ✅ Summary Table

| Tool         | Interface | Setup | Pros                       | Use Case                         |
| ------------ | --------- | ----- | -------------------------- | -------------------------------- |
| Git InstaWeb | Web       | Low   | One-command viewer         | Browsing commits quickly         |
| tig          | Terminal  | Low   | Fast, rich terminal UI     | Engineers and techs at the bench |
| Gitea        | Web       | High  | Full GitHub-style platform | Teams, CI/CD, remote editing     |

---

### Recommendation

Start with `git instaweb` or `tig` for simplicity. If the team needs a full web interface later, consider deploying Gitea internally.

All tools listed here are fully offline and compliant with internal use in a secure manufacturing environment.

---
