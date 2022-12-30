---
---

# Markdown Tips and Tricks

## generate pdf from markdown
```sh
pandoc GETTING_STARTED.md -o README.pdf "-fmarkdown-implicit_figures -o" --from=markdown --toc --highlight-style=espresso
```