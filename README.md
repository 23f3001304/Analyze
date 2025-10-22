```markdown
# Project Summary

This project demonstrates a CI/CD pipeline for a static web application that analyzes sales data. A Python script processes a CSV file to generate key business metrics. The results are automatically published to GitHub Pages using a GitHub Actions workflow that also includes code linting.

# Setup

Follow these instructions to set up the project environment locally.

### Prerequisites

*   Git
*   Python 3.11 or newer

### Installation

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/your-username/your-repository.git
    cd your-repository
    ```

2.  **Create and activate a virtual environment:**
    ```sh
    # For macOS/Linux
    python3 -m venv venv
    source venv/bin/activate

    # For Windows
    python -m venv venv
    .\venv\Scripts\activate
    ```

3.  **Install the required dependencies:**
    ```sh
    pip install pandas==2.3 ruff
    ```

# Usage

### Local Execution

You can run the analysis script locally to generate the JSON output. From the root of the project directory, run:

```sh
python execute.py > result.json
```

This command reads `data.csv`, performs the analysis, and prints the resulting JSON to a `result.json` file. Note that this file is intentionally not tracked by Git and should not be committed.

### CI/CD Pipeline

The primary workflow is automated through GitHub Actions. When changes are pushed to the `main` branch, the following actions occur automatically:

1.  The Python code is linted using `ruff`.
2.  The `execute.py` script is run to generate `result.json`.
3.  The `result.json` artifact is deployed to GitHub Pages.

The final output is accessible at the GitHub Pages URL for this repository.

# Code Explanation

### `execute.py`

This Python script uses the `pandas` library to perform data analysis on the sales data contained in `data.csv`.

*   **Functionality:** It reads the data, computes the total revenue for each transaction, and calculates several key metrics:
    *   Total number of rows.
    *   Count of unique sales regions.
    *   The top 3 products by total revenue.
    *   The final 7-day rolling average revenue for each region.
*   **Output:** The script aggregates these metrics into a dictionary and prints it to standard output as a formatted JSON string.
*   **Fixes:** The script has been updated from its original version to correct a typo in a column name (`revenew` -> `revenue`) and to read data from `data.csv` instead of the original Excel file.

### `data.csv`

This file contains the raw sales data. It was converted from `data.xlsx` for easier processing. The columns are: `date`, `region`, `product`, `units`, and `price`.

### `.github/workflows/ci.yml`

This file defines the GitHub Actions workflow that automates the linting, analysis, and deployment process.

```yaml
name: CI and Deploy

on:
  push:
    branches:
      - main

# Grant permissions for deployment
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas==2.3 ruff

      - name: Lint with ruff
        run: ruff check .

      - name: Run analysis script
        run: python execute.py > result.json

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          # Upload the entire repository, including result.json
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

*   **Trigger:** The workflow runs on every `push` to the `main` branch.
*   **Permissions:** It is granted the necessary permissions to write to GitHub Pages.
*   **Steps:**
    1.  **Checkout:** Clones the repository content.
    2.  **Set up Python:** Configures the environment with Python 3.11.
    3.  **Install dependencies:** Installs `pandas` and `ruff`.
    4.  **Lint with ruff:** Runs `ruff check` to ensure code quality and prints the results to the CI log.
    5.  **Run analysis script:** Executes `execute.py` and redirects its standard output to create `result.json`.
    6.  **Setup & Deploy Pages:** Configures and deploys the contents of the root directory (which now includes `result.json`) as a GitHub Pages artifact.

# License

This project is licensed under the MIT License.
```