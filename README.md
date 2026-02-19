# GitHub Actions Hello World

A simple Python project to learn GitHub Actions basics.

## Running Locally

```bash
python src/main.py
```

## GitHub Actions Concepts

- **Workflow**: Defined in `.github/workflows/run-hello-world.yml`
- **Events**: Triggers on push, pull_request, and manual dispatch
- **Runner**: Uses Ubuntu (latest)
- **Jobs**: Single job that runs the Python script
- **Steps**: Checkout code, setup Python, run script
