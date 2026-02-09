# AmpliconRepository-docs

Documentation for AmpliconRepository built with MkDocs and hosted on Read the Docs.

## Building Locally

To build and preview the documentation locally:

```bash
# Install dependencies
pip install -r docs/requirements.txt

# Serve the documentation locally
mkdocs serve

# Build the static site
mkdocs build
```

## Read the Docs Configuration

This project uses Read the Docs for hosting. The configuration is in `.readthedocs.yaml`.

- Build OS: Ubuntu 22.04
- Python: 3.12
- Theme: rtd-dropdown

## Project Structure

- `mkdocs.yml` - MkDocs configuration
- `.readthedocs.yaml` - Read the Docs build configuration
- `docs/` - Documentation source files
- `docs/requirements.txt` - Python dependencies for building
