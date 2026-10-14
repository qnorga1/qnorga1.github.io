# Getting Started

URL: "qnorga1.github.io", which will also be your website's URL.

1. Edit site-wide configuration in `_config.yml` and double check that the `url` is the one that you just selected in the previous step and that `repository` reflects the correct path for your repository.
1. Add your site content, upload any files (like PDFs, .zip files, etc.) to the `files/` directory. They will appear at https://[your GitHub username].github.io/files/example.pdf.
1. Check status by going to the repository settings, under: Settings -> Pages -> Source: GitHub Actions

# Running locally

Run `jekyll serve -l -H localhost` to generate the HTML and serve it from `localhost:4000` the local server will automatically rebuild and refresh the pages on change to Markdown (*.md) and HTML files, while changes to the core template and configuration (i.e., `_config.yml`) will require stopping and restarting Jekyll.
    
You may also try `bundle exec jekyll serve -l -H localhost` to ensure jekyll to use specific dependencies on your own local machine.


# License and third-party logos

The code in this repository is licensed under the MIT License (see `LICENSE`).
This license does not cover third-party logos and trademarks, which remain the property of their respective owners and are used only to link to their services:

- `images/iacr-logo.png`: logo of the [International Association for Cryptologic Research (IACR)](https://www.iacr.org/), used on buttons linking to the [IACR Cryptology ePrint Archive](https://eprint.iacr.org/).
