# Advancing Code Generation Large Language Models with Novel Fine-Tuning Techniques

This repository contains the official implementation, datasets, and evaluation results for the paper: **"Advancing Code Generation Large Language Models with Novel Fine-Tuning Techniques."**

## 📄Abstract

Large Language Models (LLMs) are proficient at generating code, but this code often lacks **maintainability**, leading to significant "technical debt" in real-world software projects. Current alignment techniques focus mainly on functional correctness and security, leaving crucial quality attributes like maintainability underexplored.

We introduce a simple yet effective **two-stage framework** to improve the maintainability of LLM-generated code:

1.  **Stage 1 (Bootstrapping Alignment):** We use Direct Preference Optimisation (DPO) to align base models with a "global" preference dataset. This dataset is automatically generated using a powerful generator model and then labelled using the **Maintainability Index (MI)**, a standard software engineering metric, to teach models a foundational preference for maintainable code.
2.  **Stage 2 (Iterative Self-Tuning):** We introduce a novel self-improvement step where each aligned model from Stage 1 generates its own specialised "local" preference data. This new dataset, which is intended to be more "in-distribution" for the model, is then used for a second round of DPO fine-tuning.

Our experiments on four open-weight models (Gemma-3 1B, 4B, 12B, and Qwen3 4B) show that this framework consistently improves both the average **Maintainability Index** and, to a lesser degree, **functional correctness** (pass@10), suggesting a mutually reinforcing relationship between these two attributes, or at minimum, demonstrating a method to improve maintainability without adversely impacting functionality.

## 📂 Repository Structure

The project is divided into three main directories, which represent the sequential stages of the framework.

### 1\. `1_Global_Dataset_Generation/`

This directory contains the scripts for **Stage 1a (Initial Data Generation)** and **Stage 1b (Automated Labelling)**.

  * **Function:** This workflow is responsible for creating the "global" DPO preference dataset. It involves preprocessing the MBPP benchmark, using a powerful generator model (Gemini 2.5 Pro) to create pairs of functionally correct solutions, and then automatically labelling these pairs (`chosen`/`rejected`) based on their Maintainability Index scores.
  * **Note:** For a detailed breakdown of the notebooks and preprocessing steps, please see the `1_Global_Dataset_Generation/readme.md` file.

### 2\. `2_Fine_Tuning_Process/`

This directory holds the core fine-tuning logic for **Stage 1c (Bootstrapping Alignment)** and **Stage 2 (Iterative Self-Tuning)**. The code is organised into subfolders for each of the four models used in our study.

  * **Function:**
      * **Stage 1c:** Fine-tunes the base models on the "global" DPO dataset (from directory 1) to create the first-stage `-dpo` models.
      * **Stage 2:** Uses the new `-dpo` models to generate their own "local" DPO datasets (Stage 2a) and then fine-tunes them a second time on this self-generated data to create the final `-dpo-further` models (Stage 2c).
  * **Note:** For specific implementation details and model-specific notebooks, please refer to the `2_Fine_Tuning_Process/readme.md` file.

### 3\. `3_Evaluation/`

This directory contains the complete evaluation pipeline, raw output files, and unaggregated statistics used to generate the results in the paper.

  * **Function:** Runs the `pass@10` (functional correctness) and average Maintainability Index (MI) evaluations for all three model variants (`baseline`, `-dpo`, and `-dpo-further`). Each evaluation is run 5 times for each model to ensure robust and reliable results.
  * **Note:** For details on the evaluation scripts and to view the raw, unaggregated statistics for each model run, please see the `3_Evaluation/readme.md` file.

<!-- ## ✍️ How to Cite

If you find this work useful in your research, please consider citing our paper:

```bibtex
@article{truong2025advancing,
  title={Advancing Code Generation Large Language Models with Novel Fine-Tuning Techniques},
  author={Truong, Jacob and Nguyen, Van and Nguyen, Thanh Thi},
  year={2025}
}
``` -->