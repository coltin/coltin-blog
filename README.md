# Coltin's Blog

Welcome to my personal blog! This repository contains the source code and content for my static site, powered by Hugo.

## Table of Contents
- [About](#about)
- [Getting Started](#getting-started)
- [Development](#development)
- [Deployment](#deployment)
- [Content Structure](#content-structure)
- [Contributing](#contributing)
- [License](#license)

## About

This is a static blog site built with [Hugo](https://gohugo.io/). It features posts about programming, technology, and my personal interests.

## Getting Started

### Prerequisites
- [Hugo](https://gohugo.io/getting-started/installing/) (recommended: use the extended version)
- [Git](https://git-scm.com/)

### Installation

1. Clone the repository:

   ```sh
   git clone https://github.com/coltin/coltin-blog.git
   cd coltin-blog
   ```

2. Install Hugo if you haven't already.

3. Run the development server:

   ```sh
   hugo server -D
   ```

   The site will be available at `http://localhost:1313/`.

## Development

- Content files are in `/content`.
- Configuration is in `config.toml`.
- Themes are in `/themes`.
- Static assets (images, etc.) are in `/static`.

To add a new post:

```sh
hugo new posts/my-new-post.md
```

Then edit the new file under `content/posts/`.

## Deployment

This site can be deployed to any static hosting provider (e.g., GitHub Pages, Netlify, Vercel, Cloudflare Pages).

Recommended steps:
1. Build the site:

   ```sh
   hugo --minify
   ```

2. The static files are in `public/`. Deploy this folder to your hosting provider.

You can automate deployments using GitHub Actions or your preferred CI/CD tool.

## Content Structure

- `/archetypes`: Archetype templates for new content
- `/content`: Blog posts and pages
- `/static`: Static files (images, CSS, JS)
- `/themes`: Hugo themes
- `config.toml`: Site configuration

## Contributing

Contributions, suggestions, or corrections are welcome!  
Feel free to submit issues or pull requests.

---

> _Built with love and Hugo. Thanks for visiting!_
