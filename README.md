# R-writing

An agent skill for readable R scripts and clear figures for journal articles.

[SKILL.md](SKILL.md) defines 13 rules for writing, refactoring, and reviewing R code. They cover script structure, `%>%` pipelines, descriptive names, comments, package loading, paths, and code that is easy to run and edit one step at a time. Plotting rules cover layout, color choices, checks of exported figures, and concise labels.

The skill describes coding and presentation style. The task determines the analysis method and its parameters.

## Install

Place this repository in a skills directory your agent reads. For a Codex setup that uses `~/.codex/skills`:

```bash
git clone https://github.com/SonnyHC/r-writing.git ~/.codex/skills/r-writing
```

## Use

Invoke the skill with a request and provide the data structure or code to work on:

```text
$r-writing Refactor this R script so I can run it section by section.
Keep the analysis unchanged and improve the plots for a journal article.
```

Complete scripts use six sections: load packages, import data, clean data, analyze, visualize, and export results. The response starts with the code, followed by short notes on input format, parameters to edit, and outputs.

Colors and layouts follow the data and the purpose of each figure. The skill keeps necessary labels and statistical annotations while trimming repeated explanations inside plots.

## Files

- [SKILL.md](SKILL.md): the full writing rules.
- [agents/openai.yaml](agents/openai.yaml): the display name, description, and default prompt.
