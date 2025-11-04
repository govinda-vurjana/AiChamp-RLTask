# RL Task for LLM Training (Anthropic-Powered)

## Project Overview

This project provides a framework for creating and evaluating Reinforcement Learning (RL) tasks for Large Language Models (LLMs) using **Anthropic's Claude API**. The goal is to teach an LLM a practical skill that a Machine Learning Engineer or researcher would use in their daily work.

### The Task: Fixing a Data Preprocessing Pipeline

The specific task in this repository challenges an AI agent to fix a broken data preprocessing pipeline. The agent is given a prompt explaining the requirements and a set of tools to interact with the environment. The agent's goal is to write a Python function called `preprocess_data` that correctly cleans, transforms, and prepares a dataset for model training, addressing common issues like missing values, categorical data, and data leakage.

The evaluation is run for 10 trials by default, and the target success rate is between **10% and 40%** to ensure the task is neither too easy nor too hard for effective RL training.

### RL Task Design Requirements

This task is designed to meet the following criteria:

1. **Relevant to ML Engineers/Researchers**: Data preprocessing is a fundamental skill in ML workflows
2. **Target Success Rate**: 10-40% pass rate ensures optimal difficulty for RL training
3. **Precise Grading**: The grader checks 9 specific requirements that match the prompt exactly
4. **Multiple Solution Approaches**: Allows various valid implementations (pipelines, manual transforms, etc.)
5. **Comprehensive Verification**: Every requirement in the prompt is verified by the grader
6. **Diverse Failure Modes**: Models can fail for different reasons (data leakage, missing values, encoding issues, etc.)
7. **Suitable Tools**: Simple, model-friendly tools (python_expression, file_reader, submit_answer)
8. **Concise & Reviewable**: Under 300 lines of core code (prompt + grader + tools)

<img width="1174" height="386" alt="image" src="https://github.com/user-attachments/assets/f17c1004-4a70-44fa-a98f-f94f620dfd2e" />

<img width="1744" height="590" alt="image" src="https://github.com/user-attachments/assets/629b0c34-359b-40e9-819b-e665d9008efd" />


### How It Works

1.  The `run_trials.py` script is executed.
2.  The script loads the task prompt from `task/prompt.txt` and presents it to the AI agent.
3.  The agent uses the tools available in `task/tool_api.py` (like `python_expression` and `file_reader`) to understand the problem and develop a solution.
4.  The agent submits its final code for the `preprocess_data` function using the `submit_answer` tool.
5.  The `task/grader.py` script executes the submitted function and evaluates it against a list of 9 rigorous requirements.
6.  The results are saved and appended to log files in the `results/` directory.

### File Breakdown

-   `run_trials.py`: The main entry point for running the evaluation. It handles command-line arguments, orchestrates the trials, and saves the results.
-   `task/prompt.txt`: A text file containing the detailed instructions and requirements for the AI agent.
-   `task/tool_api.py`: Defines the set of Python tools the agent can use to interact with the system, such as executing code or reading files.
-   `task/grader.py`: Contains the core logic for evaluating the agent's submitted code. This is the most critical part of the task's verification.
-   `task/data/sample_dataset.csv`: The default dataset used for the evaluation.
-   `results/`: This directory stores the output logs. `pass_rate.txt` contains a running summary, and `all_runs.json` contains detailed, append-only logs of every trial.
-   `README.md`: This file, providing documentation for the project.

### The Grading Process in Detail

The grader is designed to be robust and flexible, allowing for multiple correct solution approaches. It evaluates the submitted `preprocess_data` function based on the quality of the output data and the methods used. A submission is considered successful if it passes at least **5 out of the 9** requirements:

1.  **No Missing Values**: Checks that the final `X_train` and `X_test` dataframes contain no `NaN` values.
2.  **No Infinite Values**: Ensures the final dataframes do not contain any infinite values.
3.  **Correct Train/Test Split**: Verifies that the data was split into the correct proportions (e.g., an 80/20 split).
4.  **All Features Numeric**: Confirms that all categorical features have been successfully converted to a numeric format (e.g., through one-hot encoding).
5.  **Feature Normalization**: Checks that the numerical features have been standardized (e.g., using `StandardScaler`), indicated by a mean close to 0 and a standard deviation close to 1.
6.  **No Data Leakage**: Looks for evidence that preprocessing steps (like scaling) were fitted on the training data *before* being applied to the test data. It checks for the correct usage of `fit_transform` and `transform` or a `Pipeline`.
7.  **Categorical Variables Handled**: Ensures that categorical variables were not simply dropped, typically by checking if the number of features has increased (due to one-hot encoding).
8.  **Target Distribution Preserved**: Verifies that stratified sampling was used during the train/test split by checking for the `stratify` keyword and comparing the target distribution between the original and the training sets.
9.  **Input Validation**: Checks if the code includes basic error handling, such as `try-except` blocks or checks for the existence of the file or target column.

---

## Setup Instructions

1. Clone the repository:
   ```bash
   git clone <your-repo-url>
   cd AiChamp-RLTask
   ```

2. Set up your Anthropic API key in `.env` file:
   ```bash
   ANTHROPIC_API_KEY=your_anthropic_key_here
   ```

3. Install dependencies (using uv or pip):
   ```bash
   # Using uv (recommended)
   uv sync
   
   # Or using pip
   pip install -e .
   ```

4. Run the task evaluation:

   The `run_trials.py` script is the single entrypoint for running evaluations.

   **Default Dataset**

   To run the evaluation with the default local dataset (`task/data/sample_dataset.csv`):
   ```bash
   uv run run_trials.py
   ```

   **Custom Dataset**

   You can specify a custom dataset using command-line arguments. The script can be pointed to any local CSV file.
   ```bash
   uv run run_trials.py --dataset-path "path/to/your/data.csv" --target-column "your_target_column_name"
   ```

## Execution Modes

You can control the execution mode with command-line flags:

- **Concurrent (Default)**: Runs trials in parallel for speed.
  ```bash
  uv run run_trials.py
  ```
- **Sequential**: Runs trials one by one. Useful for debugging.
  ```bash
  uv run run_trials.py --no-concurrent
  ```

## Logging

- **`results/pass_rate.txt`**: A summary of each run, including the data source and pass rate, is appended here.
- **`results/all_runs.json`**: Detailed results for every trial in every run are appended to this single file for a complete chronological record.

## Example Log Output (`results/pass_rate.txt`)

```
--- 2025-11-04 03:09:38 ---
Dataset: task/data/sample_dataset.csv
Pass Rate: 40.0%
Target Range: 10-40%
Status: ✓ GOOD

--- 2025-11-04 03:12:46 ---
Dataset: data/sample.csv
Pass Rate: 20.0%
Target Range: 10-40%
Status: ✓ GOOD
```

## Test Results

**Latest Test Runs** (Anthropic-only implementation with claude-3-5-haiku-latest):

| Dataset | Records | Features | Pass Rate | Avg Time | Status |
|---------|---------|----------|-----------|----------|--------|
| sample_dataset.csv | 25 | 7 | 20.0% | 25.0s | ✓ GOOD |
| ml_dataset.csv | 50 | 9 | 20.0% | 25.2s | ✓ GOOD |
| sample.csv | 25 | 4 | 20.0% | 35.8s | ✓ GOOD |

**Key Finding**: Task maintains consistent 20% pass rate across different dataset complexities, proving robust grading criteria.

### Common Failure Modes Observed

1. **Syntax errors** in generated code (try-except blocks)
2. **Type errors** with numpy arrays (len() on float64)
3. **Data leakage** (fitting on entire dataset)
4. **Missing stratification** parameter

### Successful Patterns

- Proper use of `fit_transform` on train, `transform` on test
- Correct handling of categorical variables with OneHotEncoder
- StandardScaler for normalization
- Stratified train_test_split

## RL Task Compliance

✅ **All requirements met**:
- Relevant to ML engineers (data preprocessing)
- 10-40% success rate achieved (20%)
- Precise grading (9 requirements checked)
- Multiple solution approaches accepted
- Comprehensive verification
- Diverse failure modes
- Simple, model-friendly tools
- Concise codebase (~490 lines core task)
