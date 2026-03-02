# Bot Resources 📚

Personal bot resources and documentation storage.

## Overview

This repository stores documents, notes, configurations, and other resources created by nanobot. All files are automatically synced to local storage at `~/.nanobot/personal-resources/bot-resources`.

## Usage

### Save a Document

```bash
python3 scripts/save_doc.py save <filename.md> "<content>"
```

### Sync to GitHub

```bash
python3 scripts/save_doc.py sync
```

### List All Documents

```bash
python3 scripts/save_doc.py list
```

## Features

- ✅ **Auto-sync before write** - Pulls latest changes from GitHub before saving
- ✅ **Auto-sync after write** - Pushes changes to GitHub automatically
- ✅ **Version control** - All changes tracked with Git
- ✅ **Markdown format** - All documents in GitHub-flavored Markdown

## Repository

- **GitHub**: [takethathe/bot-resources](https://github.com/takethathe/bot-resources)
- **Local Path**: `~/.nanobot/personal-resources/bot-resources`

## License

MIT
