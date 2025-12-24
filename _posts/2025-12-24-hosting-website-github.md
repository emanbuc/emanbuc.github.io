For me it was time to refresh my personal website. I do not want to deal with virtual server and hosting maintenance so GitHub Pages hosting is my got to solution.
Hosting a personal website on GitHub is a powerful way to get a professional online presence with minimal cost and maintenance. This post walks through the key concepts, benefits, and a practical workflow for getting started with GitHub Pages and a simple static site.

## Why Use GitHub for Your Personal Website

GitHub offers a feature called GitHub Pages that can host static websites directly from a repository. This is ideal for personal portfolios, research pages, and blogs written in Markdown.

Key advantages include:
- **Free** hosting with automatic HTTPS and no bandwidth limits for typical personal use.  
- Tight integration with Git: your site is version-controlled, reviewable, and easy to roll back.  
- Native support for static site generators like Jekyll, making Markdown-based blogs and documentation straightforward to manage.

## Understanding GitHub Pages

GitHub Pages serves HTML, CSS, JavaScript, and other static assets from a special branch or repository. For a user site, the repository is typically named `username.github.io`, and GitHub automatically builds and publishes it.

Important characteristics:
- Content is static: no server-side code such as Python, PHP, or Node runs on GitHub Pages.  
- Sites can be either user/organization sites (`username.github.io`) or project-specific sites under any repository.  
- Configuration is done via repository settings and simple configuration files, so most changes are handled through Git commits.

## Choosing a Site Structure and Workflow

Before writing content, it helps to decide how you want to manage your site:
- For a minimal portfolio, a hand-written HTML page plus a stylesheet can be enough.
- For content-heavy sites or blogs, using a static site generator like Jekyll simplifies layout reuse, navigation, and Markdown-based authoring.
- A hybrid approach often works well: a Jekyll site with separate sections for an “About” page, a research or projects portfolio, and a blog.
- A template-based website could give more customization options.

**1. Simple HTML/CSS/JavaScript**
- Create a repository named `yourusername.github.io`
- Add `index.html`, CSS, and JS files
- Enable GitHub Pages in Settings
- Site goes live at `https://yourusername.github.io`

**2. Using Jekyll (GitHub's Built-in Generator)**
- Jekyll is automatically supported by GitHub Pages
- Supports Markdown-based blog posts and pages
- Choose from GitHub's supported themes (Minima, Hacker, etc.)
- Ideal if you want blogging capabilities

**3. Using Pre-built Templates**
- Clone a portfolio template repository (like `al-folio` or others from GitHub Collections)
- Customize with your content
- Deploy with a single push

### **Key Features Across Approaches**
- ✅ Free hosting
- ✅ Free HTTPS
- ✅ Custom domain support (optional)
- ✅ No backend server needed (static sites only)
- ✅ Version control built-in
- ✅ Easy updates via Git

