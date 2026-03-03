---
name: download-analyze-paper
description: Downloads an arXiv paper given its ID/URL, creates a target directory, extracts the LaTeX source, and generates a reviewer-level analytical note. Use when the user wants to download a paper and generate notes for it.
---

# Paper Download and Analysis Workflow

This skill provides a self-contained workflow for downloading arXiv papers and performing a deep reviewer-level analysis.

## Instructions

When the user asks to download an arXiv paper and generate a note, follow these steps.  
By default, you should assume a **`BASE_DIR`** under which you always create two subfolders:

- `BASE_DIR/papers` 用于存放论文 LaTeX 源码（每篇一个子目录）
- `BASE_DIR/notes` 用于存放对应的 Markdown 审稿笔记

约定规则：
- 若用户显式给出了一个目录（例如 `reviewer`），则将该目录视为 `BASE_DIR`。
- 若用户未给出目录，只是说“下载论文并生成笔记”，则将仓库根目录视为 `BASE_DIR`。

1.  **Extract Information:**
    *   Extract the `ARXIV_ID` from the user's input (e.g., from `https://arxiv.org/abs/2602.11124`, the ID is `2602.11124`).
    *   Determine the `METHOD_NAME` (e.g., "PhyCritic"). If the user doesn't provide one, try to extract it from the paper title or ask the user.
    *   Determine the `BASE_DIR`:
        - If the user mentions a directory like `reviewer` ("下载到reviewer并生成笔记"), then `BASE_DIR = reviewer`.
        - Otherwise, use the workspace root as `BASE_DIR`.
    *   Determine the `TARGET_DIR` where the paper should be saved: `TARGET_DIR = <BASE_DIR>/papers/<METHOD_NAME>` (e.g., `reviewer/papers/PhyCritic`).
    *   Determine the `TARGET_NOTES_DIR` where the output markdown note should be saved: `TARGET_NOTES_DIR = <BASE_DIR>/notes/<METHOD_NAME>.md` (e.g., `reviewer/notes/PhyCritic.md`).

2.  **Locate Skill Assets:**
    *   Find the path to this skill's directory. In this project it is typically `.cursor/skills/paper-reading-skill`（如已全局安装则可能为 `~/.cursor/skills/download-analyze-paper`，需按实际路径调整）。
    *   You will need the script `scripts/download_arxiv.sh` and the prompt file `reviewer_prompt.md` located inside the skill directory.

3.  **Download and Extract Paper Source:**
    *   Execute the generalized bash script provided by this skill to download the paper. Make sure to use the absolute path to the skill folder.
        ```bash
        bash <SKILL_DIR>/scripts/download_arxiv.sh <ARXIV_ID> <TARGET_DIR>
        ```

4.  **Generate Analytical Note:**
    *   Use the `Read` tool to read the contents of `<SKILL_DIR>/reviewer_prompt.md`.
    *   Use the `Task` tool (with `subagent_type: generalPurpose`) to analyze the downloaded paper.
    *   In the `Task` prompt, include the full text of `reviewer_prompt.md` so the subagent knows exactly how to analyze the paper.
    *   Instruct the subagent to read the LaTeX source in `<TARGET_DIR>`.
    *   Instruct the subagent to **strictly follow** the criteria in the prompt.
    *   Instruct the subagent to save the output markdown note to `<TARGET_NOTES_DIR>` (e.g., `reviewer/notes/PhyCritic.md`). 
    *   Tell the subagent to use `$$` for block math formulas and `$` for inline math.
    *   **Important**: Markdown does not support `\begin{aligned}`, `\begin{array}`, `\begin{cases}`, or other LaTeX environments. Only use simple inline (`$...$`) or block (`$$...$$`) math syntax.
    *   Tell the subagent to perform web searches for section 4 of the prompt ("与当前领域主流共识及反对观点的关系").

5.  **Confirm Completion:**
    *   Once the subagent finishes, report success to the user and mention the path to the newly created markdown note.

## Example Usage

**User:** "下载论文 https://arxiv.org/abs/2602.11124 到某个目录（例如 `reviewer`），并生成审稿笔记。"

**Agent Action:**
1.  解析得到例如 `ARXIV_ID = 2602.11124`，`METHOD_NAME = <METHOD_NAME>`，`BASE_DIR = <BASE_DIR>`，因此：
    - `TARGET_DIR = <BASE_DIR>/papers/<METHOD_NAME>`
    - `TARGET_NOTES_DIR = <BASE_DIR>/notes/<METHOD_NAME>.md`
2.  Run `bash <SKILL_DIR>/scripts/download_arxiv.sh 2602.11124 <BASE_DIR>/papers/<METHOD_NAME>`.
3.  Read `<SKILL_DIR>/reviewer_prompt.md`.
4.  Run `Task` tool prompting the subagent to read `<BASE_DIR>/papers/<METHOD_NAME>/main.tex`, follow the reviewer prompt, and write the result to `<BASE_DIR>/notes/<METHOD_NAME>.md`.