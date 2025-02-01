# ClickHouse SQL Reference Sheet

A comprehensive SQL command reference sheet for ClickHouse Cloud field engineers. This documentation is built using Material for MkDocs and contains essential SQL commands, best practices, and optimization techniques.

[![Deploy MkDocs to GitHub Pages](https://github.com/your-username/click_reference_sheet/actions/workflows/deploy.yml/badge.svg)](https://github.com/your-username/click_reference_sheet/actions/workflows/deploy.yml)

## 📚 Contents

- Basic Operations (Connection, Databases, Tables)
- Data Operations (Insert, Select, Update/Delete)
- System Commands (Monitoring, Performance)
- Advanced Operations (Materialized Views, Distributed Tables)

## 🚀 Quick Start

### Prerequisites

- Python 3.8 or higher
- pip (Python package installer)
- Git

### Local Development Setup

1. Clone the repository:

```bash
git clone https://github.com/maruthiprithivi/click_reference_sheet
cd click_reference_sheet
```

2. Create and activate a virtual environment:

```bash
# On macOS/Linux
python -m venv venv
source venv/bin/activate

# On Windows
python -m venv venv
.\venv\Scripts\activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Run the documentation locally:

```bash
mkdocs serve
```

5. Open your browser and visit `http://127.0.0.1:8000`

### Building the Documentation

To build the static site:

```bash
mkdocs build
```

The built documentation will be in the `site` directory.

## 🤝 Contributing

We welcome contributions to improve the ClickHouse SQL Reference Sheet! Here's how you can contribute:

### Creating Issues

1. Go to the [Issues](https://github.com/maruthiprithivi/click_reference_sheet/issues) page
2. Click "New Issue"
3. Choose the appropriate issue template:
   - Bug Report
   - Feature Request
   - Documentation Improvement
4. Fill in the template with detailed information
5. Submit the issue

### Making Changes

1. Fork the repository
2. Create a new branch for your changes:

```bash
git checkout -b feature/your-feature-name
```

3. Make your changes
4. Test locally using `mkdocs serve`
5. Commit your changes:

```bash
git add .
git commit -m "Add: brief description of your changes"
```

6. Push to your fork:

```bash
git push origin feature/your-feature-name
```

### Creating Pull Requests

1. Go to the [Pull Requests](https://github.com/maruthiprithivi/click_reference_sheet/pulls) page
2. Click "New Pull Request"
3. Select your fork and branch
4. Fill in the PR template:
   - Description of changes
   - Related issues
   - Testing performed
   - Screenshots (if applicable)
5. Submit the PR

### Code Style Guidelines

- Follow Markdown best practices
- Use clear and concise commit messages
- Keep SQL examples consistent with the existing format
- Include comments in SQL examples where necessary
- Test all SQL commands before submitting

## 📝 Documentation Structure

```
docs/
├── index.md              # Main documentation page
├── basic-operations/     # Basic SQL operations
├── data-operations/      # Data manipulation operations
├── system-commands/      # System monitoring and management
└── advanced-operations/  # Advanced features and operations
```

## 🛠 Development

### Setting Up for Development

1. Install development dependencies:

```bash
pip install -r requirements-dev.txt
```

2. Set up pre-commit hooks:

```bash
pre-commit install
```

The pre-commit hooks will:

- Check for trailing whitespace
- Fix end of files
- Check YAML formatting
- Prevent large file commits
- Check for merge conflicts
- Enforce consistent line endings
- Format Markdown files using mdformat (with GitHub Flavored Markdown support)
- Format and lint Python code using Ruff (combines functionality of Black, isort, and Flake8)

3. Run pre-commit manually (optional):

```bash
pre-commit run --all-files
```

4. Run formatters manually (optional):

```bash
# Format Markdown files
mdformat .

# Format and lint Python code
ruff format .
ruff check . --fix
```

### Testing Changes

1. Run the documentation locally:

```bash
mkdocs serve
```

2. Check for broken links:

```bash
mkdocs build --strict
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [ClickHouse](https://clickhouse.com/) - The open-source columnar database
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) - The documentation framework
- All contributors who help improve this reference sheet

## 📞 Support

If you need help or have questions:

1. Check the [Issues](https://github.com/maruthiprithivi/click_reference_sheet/issues) page for existing problems and solutions
2. Create a new issue if you can't find an answer
3. Join our [Discussion](https://github.com/maruthiprithivi/click_reference_sheet/discussions) forum

## 🔄 Continuous Integration

This project uses GitHub Actions for continuous integration:

- Automatic deployment to GitHub Pages
- Link checking
- Markdown linting
- Documentation building

The status of the latest deployment can be seen at the top of this README.

## 📈 Project Status

This is an active project that is regularly maintained and updated. We welcome contributions from the community to help improve and expand the reference sheet.
