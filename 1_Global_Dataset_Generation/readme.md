# 1\_Global\_Dataset\_Generation

This directory contains the scripts and notebooks responsible for executing **Stage 1a - Initial Data Generation** and **Stage 1b - Automated Labelling**, as described in our paper.

The primary objective of this workflow is to programmatically generate a "global" preference dataset for Direct Preference Optimisation (DPO). This dataset is designed to teach base models an initial, general preference for generating code with high maintainability.

## Workflow

The process is divided into three main steps, each corresponding to a Jupyter Notebook in this folder.

### 1\. Dataset Preprocessing

**Script:** `1_dataset_preprocessing.ipynb`

This initial step prepares the source prompts from the [Mostly Basic Python Programming (MBPP) dataset](https://arxiv.org/pdf/2108.07732).

  * The script loads the `mbpp` dataset.
  * It filters the training split to exclude examples that involve class definitions, simplifying the problem set.
  * Critically, it defines a helper function `add_function_signature` which extracts the function definition (e.g., `def my_func(arg1, arg2)`) from the canonical solution and appends it to the natural language prompt. This enriches the prompt to ensure the generator model adheres to the correct function signature.
  * The preprocessed training and test datasets are saved to disk (e.g., `mbpp_preprocessed_dataset` and `mbpp_test_with_signatures`).

### 2\. Initial Data Generation

**Script:** `2_acquire_dpo_from_gemini.ipynb`

This script uses the preprocessed prompts to generate the candidate code solutions.

  * It loads the `mbpp_preprocessed_dataset` created in the previous step.
  * It uses a powerful generator model (Gemini 2.5 Pro, June 2025 Ver.) to generate two distinct, functionally correct implementations for each prompt.
  * The script formats requests for the Gemini API, specifying a JSON schema for the output, which must contain `code_output_1` and `code_output_2`.
  * After acquiring the generated pairs, it runs the unit tests associated with each prompt against both generated solutions.
  * Only pairs where **both** implementations pass all unit tests are retained, ensuring functional correctness.
  * The final set of prompts with their two corresponding correct solutions is saved to disk (e.g., `mbpp_custom_dataset_with_correct_gemini_pairs`).

### 3\. Automated Preference Labeling

**Script:** `3_automated_global_dataset_labelling.ipynb`

This final script applies an objective software metric to create the preference labels.

  * It loads the dataset of functionally correct pairs generated in the previous step.
  * For each pair of solutions (`code_output_1` and `code_output_2`), it calculates the **Maintainability Index (MI)** using the `radon.metrics.mi_visit` function.
  * It compares the two MI scores. The implementation with the **higher** MI score is labeled as `"chosen"`, and the one with the **lower** MI score is labeled as `"rejected"`.
  * The data is formatted into a final DPO dataset containing `(prompt, chosen, rejected)` triplets.
  * This "global DPO dataset" is saved to disk (e.g., `mbpp_custom_dataset_with_correct_gemini_pairs_dpo_ready_with_messages`), making it ready for the first stage of fine-tuning.