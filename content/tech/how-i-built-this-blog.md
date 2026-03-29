---
title: "How I Set Up This Blog Using Hugo, GitHub, and Cloudflare"
date: 2026-03-29
tags: ["hugo", "cloudflare", "github", "blogging", "static-site"]
categories: ["tech"]
summary: "A step-by-step guide to setting up a minimalistic, markdown-powered blog using Hugo, GitHub, and Cloudflare Pages — from zero to a live site."
---

I wanted a blog that is simple to write, fast to load, and cost effective to host. Here is what I needed:

- A single site for everything — articles, notes, quotes, brain dumps — all public, all at one domain
- Content spanning topics from technology (Flutter, Golang, microservices) to management to hardware (Arduino)
- A hierarchical navigation that mirrors how I think: `tech → flutter`, `tech → golang`, `management`, etc.
- Markdown as the writing format so I can use any editor (VS Code) or generate content with AI tools like Claude
- Support for code blocks, mermaid diagrams, tags, and categories
- A minimalistic UI that stays out of the way
- A workflow as simple as: write markdown locally, git push, site updates

After evaluating several static site generators — Hugo, Astro, Eleventy, Quartz — I landed on a stack that checks every box:

- **Hugo** — static site generator that turns markdown into a website, mature and actively maintained
- **GitHub** — version control and storage for all content
- **Cloudflare Pages** — free hosting with automatic deployments

The result: I write markdown files in VS Code, push to GitHub, and the site updates itself. No databases, no servers, no CMS logins, no Obsidian sync scripts.

Here is exactly how I set it up.

## Why This Stack?

| Concern | Solution |
|---|---|
| Writing format | Markdown — plain text, portable, works with any editor |
| Site generation | Hugo — fast builds, mature ecosystem, great taxonomy support |
| Version control | GitHub — track every change, rollback anytime |
| Hosting | Cloudflare Pages — free, fast CDN, auto-deploy on push |
| Domain | Cloudflare DNS — already managing my domain here |

No monthly costs. No vendor lock-in on content. If Hugo dies tomorrow, my posts are still plain markdown files.

## Step 0: Buy a Domain Name

You do not strictly need a custom domain — Cloudflare Pages gives you a free `*.pages.dev` subdomain. But a custom domain looks professional and costs very little.

Here is how the major registrars compare:

| Feature | GoDaddy | Namecheap | Cloudflare |
|---|---|---|---|
| `.com` renewal price | ~$22/year | ~$13/year | ~$10/year (at cost) |
| First year pricing | Low (promo) then jumps | Low (promo) then jumps | Same as renewal — no tricks |
| WHOIS privacy | Paid add-on | Free | Free |
| Upsells at checkout | Aggressive | Moderate | None |
| DNS management | Basic | Good | Excellent (you will use it anyway) |
| Domain transfers | Possible but slow | Easy | Easy |

**My recommendation: Cloudflare Registrar.** They sell domains at wholesale cost with zero markup — what they pay to the registry is what you pay. No introductory pricing gimmicks, no hidden fees, free WHOIS privacy. And since you will be using Cloudflare Pages for hosting, your domain and hosting are in the same dashboard with DNS pre-configured.

To register a domain on Cloudflare:

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Go to **Domain Registration** → **Register Domains**
3. Search for your desired domain and complete the purchase

If you already bought your domain elsewhere (GoDaddy, Namecheap, etc.), you can transfer it to Cloudflare later, or just point the nameservers to Cloudflare — both work fine.

## Setup

Before starting, you need:

- [Git](https://git-scm.com/downloads) installed
- [Hugo](https://gohugo.io/installation/) extended edition installed
- A [GitHub](https://github.com) account
- A [Cloudflare](https://cloudflare.com) account
- A domain pointed to Cloudflare DNS (optional but recommended)

### Installing Hugo (Extended Edition)

Hugo comes in two editions: standard and extended. You need the **extended** edition — it includes support for SCSS/SASS processing which most themes (including Blowfish) require.

**macOS** (using Homebrew):

```bash
brew install hugo
```

Homebrew installs the extended edition by default. If you do not have Homebrew, install it first:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Windows** (using Chocolatey):

```bash
choco install hugo-extended
```

Or using Winget:

```bash
winget install Hugo.Hugo.Extended
```

**Linux** (Debian/Ubuntu):

```bash
sudo apt install hugo
```

If your distro ships an older version, install the latest from the [GitHub releases](https://github.com/gohugoio/hugo/releases). Download the `hugo_extended_*_linux-amd64.deb` file and install:

```bash
sudo dpkg -i hugo_extended_*.deb
```

**Verify the installation:**

```bash
git --version
hugo version
```

The `hugo version` output should include the word `extended`. For example:

```
hugo v0.147.0+extended linux/amd64 ...
```

If you see `extended` in the output, you are good to go.

## Step 1: Create the Hugo Site

```bash
hugo new site builtbynikhil
cd builtbynikhil
git init
```

This creates the project skeleton:

```
builtbynikhil/
├── archetypes/     # templates for new content
├── config/         # site configuration (created in Step 3)
├── content/        # your markdown posts live here
├── hugo.toml       # default config (replaced in Step 3)
├── layouts/        # layout overrides
├── static/         # images, files
└── themes/         # installed themes
```

## Step 2: Install a Theme

I chose [Blowfish](https://blowfish.page/) for its clean design, built-in navigation, mermaid diagram support, and taxonomy pages. Install it as a git submodule:

```bash
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
```

Using a submodule keeps the theme as a separate concern — your content and the theme code never mix. You can update the theme independently or swap it entirely without touching your posts.

## Step 3: Configure the Site

Blowfish does not work with a single `hugo.toml` file. It expects a multi-file configuration structure under `config/_default/`. If you try using a single `hugo.toml`, the site will build but render an empty page — this is the most common gotcha when setting up Blowfish.

Delete the root `hugo.toml` that Hugo generated and create the config directory:

```bash
rm hugo.toml
mkdir -p config/_default
```

Now create the following configuration files:

**`config/_default/hugo.toml`** — core Hugo settings:

```toml
baseURL = "https://builtbynikhil.com/"
defaultContentLanguage = "en"

enableRobotsTXT = true
summaryLength = 0
enableEmoji = true

[pagination]
  pagerSize = 100

[imaging]
  anchor = 'Center'

[taxonomies]
  tag = "tags"
  category = "categories"

[sitemap]
  changefreq = 'daily'
  filename = 'sitemap.xml'
  priority = 0.5

[outputs]
  home = ["HTML", "RSS", "JSON"]
```

**`config/_default/languages.en.toml`** — site title and language settings:

```toml
disabled = false
languageCode = "en"
languageName = "English"
weight = 1
title = "Built by Nikhil"

[params]
  displayName = "EN"
  isoCode = "en"
  dateFormat = "2 January 2006"
  description = "Articles, notes, and ideas on technology, management, and more."
```

**`config/_default/params.toml`** — Blowfish theme parameters:

```toml
colorScheme = "blowfish"
defaultAppearance = "light"
autoSwitchAppearance = true

enableSearch = true
enableCodeCopy = true

mainSections = ["tech", "management", "arduino", "quotes", "brain"]

[header]
  layout = "basic"

[footer]
  showMenu = true
  showCopyright = true
  showThemeAttribution = true
  showAppearanceSwitcher = true
  showScrollToTop = true

[homepage]
  layout = "page"
  showRecent = true
  showRecentItems = 10
  showMoreLink = true
  showMoreLinkDest = "/tech/"

[article]
  showDate = true
  showAuthor = true
  showBreadcrumbs = true
  showDraftLabel = true
  showHeadingAnchors = true
  showPagination = true
  showReadingTime = true
  showTableOfContents = true
  showTaxonomies = true
  showWordCount = true

[list]
  showBreadcrumbs = true
  showSummary = true
  groupByYear = true

[taxonomy]
  showTermCount = true

[term]
  showTableOfContents = true
  groupByYear = false
```

The `[homepage]` section is critical. Blowfish defaults to a `profile` layout that shows an author card — if you have not configured author info, the page renders empty. Setting `layout = "page"` with `showRecent = true` gives you a straightforward homepage that lists your latest posts.

**`config/_default/menus.en.toml`** — navigation menu:

```toml
[[main]]
  name = "Tech"
  pageRef = "tech"
  weight = 10

[[main]]
  name = "Tags"
  pageRef = "tags"
  weight = 30
```

**`config/_default/markup.toml`** — code highlighting and markdown rendering:

```toml
[highlight]
  noClasses = false
  lineNos = true

[goldmark]
  [goldmark.renderer]
    unsafe = true
```

**`config/_default/module.toml`** — tells Hugo to load Blowfish:

```toml
[[imports]]
  path = "blowfish"
```

The `[taxonomies]` block in `hugo.toml` tells Hugo to generate tag and category pages automatically. Every post tagged with `hugo` gets listed at `/tags/hugo/`.

## Step 4: Set Up the Content Hierarchy

Hugo uses your folder structure as the site's URL structure. For this first post, we only need the `tech` section:

```bash
mkdir -p content/tech
```

Create an `_index.md` file to define the section's listing page:

```bash
cat <<'EOF' > content/tech/_index.md
---
title: "Tech"
description: "Articles and notes on software engineering and technology."
---
EOF
```

As you write more, you add new sections by simply creating folders — `content/management/`, `content/arduino/`, `content/tech/flutter/`, etc. Each folder with an `_index.md` becomes a navigable section on your site. The hierarchy grows organically with your content — no need to set everything up upfront.

## Step 5: Write Your First Post

Create a new post:

```bash
hugo new content/tech/how-i-built-this-blog.md
```

Open the file and write using standard markdown. Hugo supports everything you would expect:

**Code blocks** with syntax highlighting:

```go
func main() {
    fmt.Println("Hello from the blog")
}
```

**Mermaid diagrams** — Blowfish uses a shortcode, not the standard markdown fenced code block:

```text
{{</*/* mermaid */*/>}}
graph LR
    A[Write Markdown] --> B[Git Push]
    B --> C[Cloudflare Builds]
    C --> D[Site Live]
{{</*/* /mermaid */*/>}}
```

> **Note:** Replace `/* mermaid */` with `mermaid` — the `/* */` wrapping above is just Hugo's escape syntax to display the shortcode as text.

Which renders as:

{{< mermaid >}}
graph LR
    A[Write Markdown] --> B[Git Push]
    B --> C[Cloudflare Builds]
    C --> D[Site Live]
{{< /mermaid >}}

**Frontmatter** at the top of every post controls metadata:

```yaml
---
title: "Your Post Title"
date: 2026-03-29
tags: ["hugo", "tutorial"]
categories: ["tech"]
summary: "A short description for listing pages."
draft: false
---
```

## Step 6: Test Locally

Run the development server:

```bash
hugo server -D
```

The `-D` flag includes draft posts. Open `http://localhost:1313` in your browser. Hugo watches for file changes and reloads the page automatically — you see updates as you type.

## Step 7: Push to GitHub

Create a new repository on GitHub (e.g., `builtbynikhil`). Then push:

```bash
git add .
git commit -m "Initial Hugo site setup"
git remote add origin https://github.com/YOUR_USERNAME/builtbynikhil.git
git push -u origin main
```

Your entire site — config files, content, theme reference — lives in this single repository.

## Step 8: Deploy on Cloudflare Pages

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Go to **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
3. Select your `builtbynikhil` repository
4. Configure the build settings:
   - **Build command:** `hugo`
   - **Build output directory:** `public`
   - **Environment variable:** `HUGO_VERSION` = `0.147.0` (or your installed version)
5. Click **Save and Deploy**

Cloudflare will clone your repo, run Hugo, and publish the output. Every future `git push` triggers a new build automatically.

## Step 9: Connect Your Domain

Since your domain is already on Cloudflare:

1. In your Cloudflare Pages project, go to **Custom domains**
2. Add `builtbynikhil.com`
3. Cloudflare sets up the DNS record automatically
4. SSL is provisioned within minutes

Your site is now live at `https://builtbynikhil.com`.

## The Daily Workflow

Once everything is set up, publishing a new post is three commands:

```bash
# Write your post (or have Claude generate it)
hugo new content/tech/flutter/state-management-patterns.md

# Edit the file in VS Code, then publish
git add content/
git commit -m "Add post: state management patterns in Flutter"
git push
```

That is it. Cloudflare picks up the push, rebuilds the site, and your post is live within a minute.

## Repository Structure

Here is how the single repository keeps concerns separated:

```
builtbynikhil/
├── config/_default/       # site configuration (yours)
│   ├── hugo.toml          # core Hugo settings
│   ├── languages.en.toml  # title, language, description
│   ├── params.toml        # Blowfish theme parameters
│   ├── menus.en.toml      # navigation menu
│   ├── markup.toml        # code highlighting, markdown
│   └── module.toml        # theme import
├── content/               # all your writing (yours)
│   ├── tech/
│   │   ├── flutter/
│   │   ├── golang/
│   │   └── microservices/
│   ├── management/
│   ├── arduino/
│   ├── quotes/
│   └── brain/
├── themes/blowfish/       # theme (git submodule, not yours)
├── layouts/               # your layout overrides (if any)
├── static/                # images, downloads
└── archetypes/            # templates for new posts
```

- **Content** is just markdown in `content/` — portable, editor-agnostic
- **Theme** is a git submodule — update or replace without touching content
- **Config** is split across files in `config/_default/` — each file handles one concern

No build scripts, no sync tools, no separate repositories to manage. Note the absence of a root `hugo.toml` — Blowfish requires the config to live in `config/_default/`.

## Bonus: Using Claude Code to Manage Content

[Claude Code](https://claude.ai/claude-code) is Anthropic's CLI tool that can read, write, and manage files in your project. Once you initialize it in your Hugo repository, it understands your site's structure and can create content that follows your conventions automatically.

### Set Up Claude Code

Run `claude` in your project root to start a session, then initialize it:

```bash
cd builtbynikhil
claude
```

On first run, Claude Code scans your project and creates a `CLAUDE.md` file — a project context file that tells it how your site works. Add instructions like this to your `CLAUDE.md`:

```markdown
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
Blowfish uses a `{{</* mermaid */>}}` shortcode — do NOT use fenced ```mermaid code blocks.
```

### Generate Content with Claude

Now you can ask Claude Code to create posts, and it will place them in the right section with correct frontmatter — creating new folders and index files if the section does not exist yet:

```bash
# Ask Claude to write an article
> Write a blog post about Flutter state management patterns

# Claude creates:
#   content/tech/flutter/_index.md  (if it didn't exist)
#   content/tech/flutter/state-management-patterns.md
```

```bash
# Ask for a new topic area entirely
> Write a note about Arduino sensor wiring basics

# Claude creates:
#   content/arduino/_index.md  (if it didn't exist)
#   content/arduino/sensor-wiring-basics.md
```

Claude reads the existing folder structure, follows the conventions in `CLAUDE.md`, and handles the hierarchy for you. You review the output, then push:

```bash
git add content/
git commit -m "Add post: Flutter state management patterns"
git push
```

This turns content creation into a conversation. You focus on what to write about — Claude handles where it goes and how it is structured.

## What is Next

From here, I plan to:

- Customize the theme colors and layout to match my style
- Set up archetypes so `hugo new` generates posts with my preferred frontmatter template
- Add an about page and a landing page
- Write more posts

The entire setup took under an hour. The best part: the writing experience is just editing markdown files in VS Code. No context switching, no web interfaces, no databases.

If you are looking for a simple, fast, and free way to publish your writing — this stack is hard to beat.

The full source for this site is on GitHub: [nnjoshi14/builtbynikhil](https://github.com/nnjoshi14/builtbynikhil). Feel free to use it as a reference.
