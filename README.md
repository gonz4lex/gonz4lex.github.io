# Gonz4lex Blog

This repository hosts the source files for my personal blog and website, built with [Quarto](https://quarto.org). The blog covers topics on data science, machine learning engineering, visualization, and software development.

## Project Structure

- `posts/`: Contains blog posts organized in date-prefixed folders.
- `assets/images/`: Static images used across the site.
- `_extensions/quarto-ext/`: Custom Quarto extensions.
- `*.qmd` files: Quarto markdown documents for pages.
- `_quarto.yml`: Quarto project configuration.
- `styles.css` and `styles.scss`: Custom styles.

## Getting Started

### Prerequisites

- [Quarto CLI](https://quarto.org/docs/get-started/)
- [Pandoc](https://pandoc.org/) (usually bundled with Quarto)
- [Git](https://git-scm.com/)

### Local Preview

To preview the site locally:

```bash
quarto preview
```

This will launch a local development server.

### Build the Site

To render the full website:

```bash
quarto render
```