---
name: download-analyze-paper
description: Downloads an arXiv paper given its ID/URL, creates a target directory, extracts the LaTeX source, and generates a reviewer-level analytical note. Use when the user wants to download a paper and generate notes for it.
---

# Paper Download and Analysis Workflow

This skill provides a self-contained workflow for downloading arXiv papers and performing a deep reviewer-level analysis.

## Instructions

When the user asks to download an arXiv paper and generate a note, follow these steps:

1.  **Extract Information:**
    *   Extract the `ARXIV_ID` from the user's input (e.g., from `https://arxiv.org/abs/2602.11124`, the ID is `2602.11124`).
    *   Determine the `METHOD_NAME` (e.g., "PhyCritic"). If the user doesn't provide one, try to extract it from the paper title or ask the user.
    *   Determine the `TARGET_DIR` where the paper should be saved (e.g., `visual-lang-action/papers/PhyCritic`).
    *   Determine the `TARGET_NOTES_DIR` where the output markdown note should be saved (e.g., `visual-lang-action/notes/PhyCritic.md`).

2.  **Locate Skill Assets:**
    *   Find the path to this skill's directory. It is usually `~/.cursor/skills/download-analyze-paper` (if installed globally).
    *   You will need the script `scripts/download_arxiv.sh` and the prompt file `reviewer_prompt.md` located inside the skill directory.

3.  **Download and Extract Paper Source:**
    *   Execute the generalized bash script provided by this skill to download the paper. Make sure to use the absolute path to the skill folder.
        ```bash
        bash ~/.cursor/skills/download-analyze-paper/scripts/download_arxiv.sh <ARXIV_ID> <TARGET_DIR>
        ```

4.  **Generate Analytical Note:**
    *   Use the `Read` tool to read the contents of `~/.cursor/skills/download-analyze-paper/reviewer_prompt.md`.
    *   Use the `Task` tool (with `subagent_type: generalPurpose`) to analyze the downloaded paper.
    *   In the `Task` prompt, include the full text of `reviewer_prompt.md` so the subagent knows exactly how to analyze the paper.
    *   Instruct the subagent to read the LaTeX source in `<TARGET_DIR>`.
    *   Instruct the subagent to **strictly follow** the criteria in the prompt.
    *   Instruct the subagent to save the output markdown note to `<TARGET_NOTES_DIR>` (e.g., `visual-lang-action/notes/PhyCritic.md`). 
    *   Tell the subagent to use `$$` for block math formulas and `$` for inline math.
    *   **Important**: Markdown does not support `\begin{aligned}`, `\begin{array}`, `\begin{cases}`, or other LaTeX environments. Only use simple inline (`$...$`) or block (`$$...$$`) math syntax.
    *   Tell the subagent to perform web searches for section 4 of the prompt ("与当前领域主流共识及反对观点的关系").

5.  **Confirm Completion:**
    *   Once the subagent finishes, report success to the user and mention the path to the newly created markdown note.

## Example Usage

**User:** "下载论文 https://arxiv.org/abs/2602.11124 到 papers/ 目录下，名字叫 PhyCritic，并生成审稿笔记。"

**Agent Action:**
1.  Run `bash ~/.cursor/skills/download-analyze-paper/scripts/download_arxiv.sh 2602.11124 papers/PhyCritic`.
2.  Read `~/.cursor/skills/download-analyze-paper/reviewer_prompt.md`.
3.  Run `Task` tool prompting the subagent to read `papers/PhyCritic/main.tex`, follow the reviewer prompt, and write the result to `notes/PhyCritic.md`.