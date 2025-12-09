# QBank Connect API Documentation

[![GitHub Pages Deployment Status](https://img.shields.io/github/actions/workflow/status/aitechnicalwriter/qapi-docs/pages/badge.svg)](https://github.com/aitechnicalwriter/qapi-docs/actions/workflows/pages.yml)

[![Documentation Build](https://img.shields.io/badge/docs-deployed-success.svg)](https://aitechnicalwriter.github.io/qapi-docs/)

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

Source repository for the website at **https://aitechnicalwriter.github.io/qapi-docs**.

> Note: All the products, entities, and data in this guide are purely fictional. This is a tech writing **portfolio demo**.

### Contents:

- **Tutorials:** Quickstart guide

- **Explanations:** Architecture, Idempotency

- **Reference:** Endpoints, SDKs, Compliance, and Glossary


## Tooling

The documentation project has been built on the following tools:

* **Documentation Generation Tool:** [MkDocs](https://www.mkdocs.org/)

* **Theme:** [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)

* **Styling:** Custom CSS styling for heading borders, typography

* **Tooling for Deployment:** GitHub Pages (via `$\text{mkdocs gh-deploy}$`)


## System Requirements

For local runs, you need Python and pip.


### Step 1. Clone The Repository

```bash

git clone [https://github.com/aitechnicalwriter/qapi-docs.git](https://github.com/aitechnicalwriter/qapi-docs.git)

cd qapi-docs

```

### Step 2. Install Dependencies

Install **MkDocs**, and **Material** theme:

```bash

pip install mkdocs mkdocs-material

```

### Step 3. Run The Server

Start the **development server** so you can see the changes immediately:

```bash

mkdocs serve

```

Now you should be able to view the output at `http://127.0.0.1:8000`.

### Step 4. Deploy

Once you've finished editing, commit your changes to `master`. Then, use the **built-in** **MkDocs** **GitHub Pages deployment** tool to publish your site. This will publish the generated static site to `gh-pages`:

```bash

mkdocs gh-deploy

```

## License

Documentation is licensed under the **MIT License**. For more details on the licensing terms, please refer to the [LICENSE](LICENSE) file.