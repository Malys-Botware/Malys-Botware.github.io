# safe space

my personal corner of the internet. a place to write, think out loud, and document what i'm building and learning.

**live at:** [malys-botware.github.io](https://malys-botware.github.io)

## what is this

a single-file personal site — pure HTML, CSS, and vanilla JS. no frameworks, no build tools. just a text editor and a browser.

the dashboard and journal sections are powered by [Supabase](https://supabase.com) (open source, MIT-licensed client) for authentication and storage. content is written with a [Quill](https://quilljs.com) rich text editor (BSD-3-Clause licensed). only i can log in and manage content. everything else is static.

built to be readable, learnable, and easy to modify. every section is commented so i can understand what the code does, not just that it works.

## structure

```
index.html          ← the entire site (HTML + CSS + JS in one file)
SETUP_GUIDE.md      ← step-by-step Supabase configuration
SECURITY_OVERVIEW.md ← full security rundown
LICENSE             ← MIT for code, CC BY-NC-SA 4.0 for content
README.md           ← you're here
```

## dependencies

- [Google Fonts](https://fonts.google.com) — Space Mono + IBM Plex Mono (loaded from CDN)
- [Supabase JS v2](https://github.com/supabase/supabase-js) — MIT license (loaded from CDN)
- [Quill v2](https://github.com/slab/quill) — BSD-3-Clause license (loaded from CDN)

no npm, no node_modules, no build step. all dependencies are loaded via `<script>` and `<link>` tags.

## features

**dashboard** — editable status tiles showing what i'm currently working on. supports draft/publish system. admin can add, edit, delete, publish/unpublish tiles with a rich text editor.

**journal** — blog-style entries with a draft/publish system. new entries default to private. admin can create, edit, delete, publish/unpublish, and share to X, Reddit, or copy a link. full rich text formatting (headers, bold, italic, lists, blockquotes, code blocks, links).

**security** — powered by Supabase Row Level Security (database-level enforcement). single admin account with public signups disabled. see SECURITY_OVERVIEW.md for full details.

## license

the **code** (HTML, CSS, JS) is [MIT](https://opensource.org/licenses/MIT) — use it, learn from it, do whatever.

the **content and artwork** (writing, journal entries, character design) is [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) — share it, but give credit, keep it non-commercial, and share alike.

## acknowledgments

built with assistance from [Claude](https://claude.ai) (Anthropic). site design, dashboard and journal system architecture, rich text editor integration, and Supabase integration developed collaboratively through conversation.
