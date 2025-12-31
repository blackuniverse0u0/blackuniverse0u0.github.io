# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **personal academic/portfolio website** built with the **al-folio Jekyll theme**, deployed to GitHub Pages. The site showcases robotics projects, publications, CV, and blog posts. The owner's primary focus is on robotics engineering with projects in locomotion, perception, planning, and control systems.

## Key Technologies

- **Jekyll** (Ruby-based static site generator)
- **Liquid** templating engine
- **Jekyll Scholar** for publications/bibliography
- **Prettier** with Liquid plugin for code formatting
- **GitHub Actions** for CI/CD
- **GitHub Pages** for hosting

## Essential Commands

### Local Development

```bash
# Install dependencies
bundle install
npm install

# Build the site locally
bundle exec jekyll build

# Serve locally with live reload (default: http://localhost:4000)
bundle exec jekyll serve

# Serve in production mode
JEKYLL_ENV=production bundle exec jekyll build
```

### Code Quality

```bash
# Format code with Prettier (runs automatically in CI)
npx prettier --write .

# The site automatically runs these checks on push:
# - Prettier formatting validation
# - Broken link checking (lychee)
# - Site deployment to GitHub Pages
```

### Deployment

- **Automatic**: Push to `main` branch triggers GitHub Actions deployment
- The deploy workflow (.github/workflows/deploy.yml) handles building and publishing to GitHub Pages
- ImageMagick and Python's nbconvert are installed during CI for image processing and Jupyter notebook support

## Architecture & Structure

### Content Organization

```
├── _pages/           # Static pages (about, projects, cv, blog index, etc.)
├── _projects/        # Project markdown files (displayed on /projects/ page)
├── _posts/           # Blog posts in Markdown
├── _news/            # News/announcements (displayed on homepage)
├── _bibliography/    # BibTeX files for publications
├── assets/
│   ├── img/          # Images for projects, posts, profile
│   ├── json/         # resume.json (CV data), other JSON data
│   └── pdf/          # PDF files (papers, CV, etc.)
└── _config.yml       # Main Jekyll configuration
```

### Projects System

Projects are managed through Jekyll collections:

1. **Project Files**: Create markdown files in `_projects/` with front matter:
   ```yaml
   ---
   layout: page
   title: Project Title
   description: Short description
   img: assets/img/project_image.png
   importance: 1  # Lower numbers appear first
   category: robotics  # Used for filtering on projects page
   ---
   ```

2. **Projects Page**: `_pages/projects.md` displays projects filtered by categories defined in `display_categories`

3. **CV Integration**: Add projects to `assets/json/resume.json` under the `projects` section to display them on the CV page

### CV System

The CV page has **two data sources** (order of precedence):

1. **Primary**: `assets/json/resume.json` - JSON Resume format (recommended)
2. **Fallback**: `_data/cv.yml` - YAML format (used if JSON not found)

The JSON format is preferred as it's a standard format and can be programmatically generated.

### Configuration Hierarchy

- `_config.yml` - Main site configuration (title, URLs, collections, plugins)
- `_data/` - Data files for repositories, navigation, etc.
- Front matter in individual files overrides global settings

### Important Directories to Exclude

- `projects/` directory at root is in `.gitignore` - this contains actual project repositories and should NOT be committed
- Images from `projects/` should be copied to `assets/img/` for use in the site

## Common Workflows

### Adding a New Project

1. Create project markdown file in `_projects/project-name.md`
2. Copy project image to `assets/img/`
3. Reference image in project front matter
4. Add to `resume.json` if you want it in CV
5. Ensure `_pages/projects.md` has the correct category in `display_categories`

### Modifying Site Configuration

- **Site metadata**: Edit top section of `_config.yml`
- **Navigation**: Controlled by `nav: true` and `nav_order` in page front matter
- **Theme colors**: Modify `_sass/_themes.scss` (variable: `--global-theme-color`)
- **Collections**: Add/modify in `_config.yml` under `collections:`

### Working with Publications

1. Add BibTeX entries to `_bibliography/papers.bib`
2. Place PDFs in `assets/pdf/`
3. Publications page (`_pages/publications.md`) auto-generates from BibTeX
4. Supported BibTeX fields: `pdf`, `code`, `slides`, `poster`, `video`, `website`, `abstract`, `arxiv`, etc.

### Formatting Code

- Prettier runs automatically on push via GitHub Actions
- Configuration: `.prettierrc` (if exists) or defaults
- Liquid plugin handles Jekyll/Liquid syntax: `@shopify/prettier-plugin-liquid`

## GitHub Actions Workflows

Key workflows in `.github/workflows/`:

- **deploy.yml**: Main deployment workflow (builds and publishes site)
- **prettier.yml**: Code formatting validation
- **broken-links-site.yml**: Checks for broken links in deployed site
- **update-citations.yml**: Auto-updates Google Scholar citations

The deploy workflow triggers on:
- Push to `main` branch
- Changes to assets, layouts, pages, posts, or config files
- Manual trigger via workflow_dispatch

## Jekyll Plugins & Features

Key plugins enabled:

- **jekyll-scholar**: Manage publications from BibTeX
- **jekyll-imagemagick**: Responsive image generation
- **jekyll-jupyter-notebook**: Embed Jupyter notebooks in posts
- **jekyll-paginate-v2**: Blog post pagination
- **jekyll-feed**: RSS/Atom feed generation
- **jekyll-sitemap**: Sitemap generation
- **jemoji**: GitHub emoji support

## Critical Files

- `_config.yml` - Core site configuration
- `Gemfile` - Ruby dependencies
- `package.json` - Node.js dependencies (Prettier)
- `assets/json/resume.json` - CV data
- `.gitignore` - Includes `projects/` to prevent committing large project repos

## Development Notes

- **Ruby version**: 3.3.5 (specified in deploy workflow)
- **Python version**: 3.13 (for nbconvert)
- **ImageMagick**: Required for responsive image generation
- The site uses CommonMark (via kramdown) for Markdown parsing
- MathJax for LaTeX math rendering
- Syntax highlighting via Rouge (GitHub style)

## Deployment Architecture

1. Push to `main` triggers GitHub Actions
2. Workflow installs Ruby, Python, and system dependencies
3. Jekyll builds site to `_site/` directory
4. PurgeCSS removes unused CSS
5. Artifact uploaded and deployed to GitHub Pages
6. Site available at: https://blackuniverse0u0.github.io
