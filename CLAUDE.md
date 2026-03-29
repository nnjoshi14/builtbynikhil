# Blog: builtbynikhil.com

This is a Hugo site using the Blowfish theme.

## Content structure
- Posts are markdown files in `content/` organized by section
- Sections: tech/ (flutter/, golang/, microservices/), management/, arduino/, quotes/, brain/
- Each section has an `_index.md` for its listing page

## Creating new posts
- Use frontmatter: title, date, tags, categories, summary, draft
- Place posts in the correct section folder based on topic
- Create new section folders and `_index.md` files as needed
- Tags are lowercase, hyphenated (e.g., "state-management", "clean-code")

## Creating new sections
When a post needs a section that does not exist yet:
1. Create the folder under `content/`
2. Add an `_index.md` with title and description frontmatter

## Mermaid diagrams
Blowfish does NOT support standard markdown fenced code blocks (` ```mermaid `) for diagrams.
Use the Hugo shortcode instead:

```
{{</* mermaid */>}}
graph LR
    A[Step 1] --> B[Step 2]
{{</* /mermaid */>}}
```

## Configuration
- Config lives in `config/_default/` (not a single root `hugo.toml`)
- Theme is installed as a git submodule in `themes/blowfish/`
