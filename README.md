# R-writing

An agent skill for clinicians who know basic R and want short scripts they can run and edit one section at a time.

The 13 rules cover writing, refactoring, and reviewing R code. Use short `%>%` pipelines for steps that read clearly together. Keep variables the calculation needs and name intermediate results when they help explain a step, even if they are used only once. Write important settings directly in the calls they affect. The plotting rules cover layout, color choices, checks of exported figures, and concise labels.

The skill describes coding and presentation style. The task determines the analysis method and its parameters.

## Install

Place this repository in a skills directory your agent reads. For Codex setups that use `~/.codex/skills`, run the following command.

```bash
git clone https://github.com/SonnyHC/r-writing.git ~/.codex/skills/r-writing
```

## Use

Invoke the skill with a request and provide the data structure or code to work on.

```text
$r-writing Refactor this R script so I can run it section by section.
Keep the analysis unchanged and improve the plots for a journal article.
```

Complete scripts load packages, import data, clean data, run the analysis, visualize, and export results in six sections. When a project is already open at its root, the script uses relative paths directly. The response starts with the code, followed by short notes on input format, parameters to edit, and outputs.

Colors and layouts follow the data and the purpose of each figure. The skill keeps necessary labels and statistical annotations while trimming repeated explanations inside plots.

## Files

- Read the full writing rules in [SKILL.md](SKILL.md).
- Find the display name, description, and default prompt in [agents/openai.yaml](agents/openai.yaml).
