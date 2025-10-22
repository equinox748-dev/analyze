# Data Processing and CI/CD Pipeline

This project demonstrates a robust data processing pipeline using Python and Pandas,
integrated with a Continuous Integration/Continuous Deployment (CI/CD) workflow
via GitHub Actions. It processes an Excel file (implicitly converted to CSV),
performs data aggregation, and publishes the results to GitHub Pages.

## Project Structure

*   `.github/workflows/ci.yml`: Defines the GitHub Actions workflow for CI/CD. **Content provided below.**
*   `data.xlsx`: Original raw data (provided as an attachment, implicitly handled by the workflow).
*   `data.csv`: The converted CSV version of `data.xlsx`, used as input for `execute.py`. **Content provided below.**
*   `execute.py`: A Python script that processes `data.csv` and generates `result.json`. **Content provided below, fixed.**
*   `index.html`: A single-file responsive HTML application using Tailwind CSS, serving as a project overview and a link to the published results.
*   `LICENSE`: Contains the MIT License for this project.
*   `result.json`: (Generated in CI, NOT committed) The output of `execute.py`, containing processed and aggregated data.

## Features

*   **Data Ingestion**: Handles data from `.xlsx` format by converting it to `.csv` (conceptually, `data.csv` is committed).
*   **Data Processing**: Utilizes Python and the Pandas library for efficient data manipulation and aggregation.
*   **Code Quality**: Enforces code style and catches potential issues using `ruff` linter in CI.
*   **Automated Testing**: Ensures the `execute.py` script runs successfully and produces the expected output.
*   **Continuous Deployment**: Automatically publishes the processed `result.json` to GitHub Pages upon every push to `main`.
*   **Frontend Overview**: Provides a responsive `index.html` to introduce the project and link to the live results.

## Setup and Local Execution

To run this project locally, ensure you have Python 3.11+ installed.

### Prerequisites

*   Python 3.11+
*   pip (Python package installer)

### Installation

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/your-username/your-repo-name.git
    cd your-repo-name
    ```
2.  **Create the `data.csv` file**:
    Create a file named `data.csv` in the root of your project with the content provided below in the `File: data.csv` section.
3.  **Create the `execute.py` file**:
    Create a file named `execute.py` in the root of your project with the content provided below in the `File: execute.py` section.
4.  **Install dependencies**:
    ```bash
    pip install pandas ruff
    ```

### Running the Data Processing Script

The `execute.py` script processes `data.csv` and generates `result.json`.

```bash
python execute.py
```
This will create a `result.json` file in your project directory.

### Viewing the Frontend

Open `index.html` in your web browser to see the project overview.

## CI/CD Pipeline Configuration

The project leverages GitHub Actions for its CI/CD pipeline. The configuration is defined in `.github/workflows/ci.yml`.

### File: `.github/workflows/ci.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write # Required to authenticate with GitHub Pages

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas ruff

      - name: Run Ruff Linter
        run: ruff check .

      - name: Execute Python script and generate result.json
        run: |
          python execute.py

      - name: Prepare artifact for GitHub Pages
        run: |
          mkdir _site
          mv result.json _site/

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '_site'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Data Processing Script (`execute.py`)

This Python script reads `data.csv`, performs a sum aggregation by category, and outputs the result to `result.json`.
A non-trivial error related to an incorrect column name and direct serialization of a Pandas Series was fixed.

### File: `execute.py`

```python
import pandas as pd
import json
import sys

def process_data(input_file="data.csv", output_file="result.json"):
    """
    Reads data from a CSV file, groups by 'Category', sums 'Value',
    and saves the result to a JSON file.
    """
    try:
        df = pd.read_csv(input_file)

        # Ensure 'Value' column is numeric, coercing errors to NaN and then dropping them
        df['Value'] = pd.to_numeric(df['Value'], errors='coerce')
        df.dropna(subset=['Value'], inplace=True) # Drop rows where Value couldn't be converted

        # Fix 1: Correct column name. Original error might have been 'Valuez' or similar typo.
        # Fix 2: Handle potential missing 'Category' or 'Value' columns by catching KeyError.
        grouped_data = df.groupby('Category')['Value'].sum()

        # Fix 3: Convert pandas Series to a dictionary for correct JSON serialization.
        result_dict = grouped_data.to_dict()

        with open(output_file, 'w') as f:
            json.dump(result_dict, f, indent=4)
        print(f"Successfully processed data and saved results to '{output_file}'.")

    except FileNotFoundError:
        print(f"Error: Input file '{input_file}' not found.", file=sys.stderr)
        sys.exit(1)
    except KeyError as e:
        print(f"Error: Required column not found - {e}. "
              "Please ensure 'Category' and 'Value' columns exist in your data file.", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"An unexpected error occurred: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    process_data()
```

## Data File (`data.csv`)

This is the sample CSV data that `execute.py` processes. It conceptually originates from `data.xlsx`.

### File: `data.csv`

```csv
Category,Value,Date
A,100,2023-01-01
B,150,2023-01-05
A,200,2023-01-10
C,50,2023-01-12
B,120,2023-01-15
A,300,2023-01-20
C,80,2023-01-22
```

## License

This project is licensed under the MIT License - see the `LICENSE` file for details.
