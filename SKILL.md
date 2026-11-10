---
name: r-writing
description: Write, refactor, and review R code with six sections, tidyverse and magrittr pipes, snake_case names, and ggplot layers on separate lines. Use familiar functions, straightforward paths, distinct stage objects, and parameters written where they are used. Keep scripts easy to run step by step, set seeds when needed, end with sessionInfo, and add brief usage notes. Use journal-style figures with colors chosen for the content, concise labels, and checks of exported plots. Use for R scripts and code style cleanup.
---

# R-writing

This skill covers R code structure, naming, formatting, dependencies, paths, reproducibility, comments, figure design, and output format. Examples illustrate code style. The current task determines the analysis goals, methods, parameters, and workflow. Preserve the meaning of computations when refactoring for style. By default, write ordinary analysis scripts that the user can run in sections, inspect, and edit in place.

## Thirteen writing rules

### 1. Organize code into six sections

Use these comment headings in order in a complete script, each on its own line:

```r
# 1. Load packages
# 2. Import data
# 3. Clean data
# 4. Run analysis
# 5. Visualize
# 6. Export results
```

- Load packages: load the packages the script needs at the top.
- Import data: read inputs directly, with paths in the read calls and no preliminary parameter block.
- Clean data: group the data preparation steps required by the task.
- Run analysis: write the steps in order using existing package functions, vectorized expressions, and clear conventional loops.
- Visualize: set plot options near their use, define any shared themes and colors, and create named plot objects.
- Export results: save the requested outputs together, then call `sessionInfo()`.

Keep this section order and preserve computational dependencies. When the user asks for a function or plotting fragment, return that fragment with the relevant section heading.

### 2. Follow tidyverse style

- Prefer `%>%` for data frame transformations and retain it as the default pipe. Combine it with familiar base R functions when that makes the code easy to understand and edit.
- Prefer `select()`, `filter()`, `mutate()`, `summarise()`, `arrange()`, `pivot_longer()`, and `*_join()` with explicit join keys.
- Keep intermediate objects when they will be reused, hold a stage result, or make the code easier to read. Combine consecutive one-time transformations in a pipeline.
- Keep clear vectorized expressions where they fit. Avoid repeated data structure conversions just to add a pipe.
- Split code into stages that are easy to run, inspect, and modify. Avoid overly long pipelines.
- Use `<-` for assignment, two-space indentation, and full `TRUE`/`FALSE` values. Put arguments on separate lines in long calls.

### 3. Use meaningful snake_case names

- Use English snake_case for variables and new columns, such as `raw_data`, `clean_data`, `summary_result`, and `trend_plot`. When the user requests custom functions, apply the same rule to their names and parameters.
- Do not use single letters, pinyin, or vague names such as `p`, `M`, `x`, `shuju`, or `data1`. Avoid names such as `data` and `exp` that shadow common functions.
- Keep fixed R API parameter names, as in `aes(x = ..., y = ...)`. The naming rule applies to names you create.
- Rename imported fields explicitly when needed. Preserve the meaning of the data values.

### 4. Put ggplot layers and mappings on separate lines

- Put `ggplot()`, each `geom_*()` and `stat_*()`, scales, coordinates, labels, and themes on separate lines or in separate multiline calls. Place `+` at the end of the preceding line.
- Put each mapping argument inside `aes()` on its own line. Keep multiple layers or mappings off the same line.
- Split calls to `labs()`, `theme()`, and geometric layers across lines when they have several arguments.
- Give plot objects clear names. Set `plot`, dimensions, and units explicitly in `ggsave()`.

This fragment shows how to format layers and mappings. Define `plot_theme` and `plot_colors` in the visualization section to suit the current figures, then reuse them across related plots:

```r
distribution_plot <- plot_data %>%
  ggplot(
    aes(
      x = category,
      y = measured_value,
      fill = category
    )
  ) +
  geom_boxplot(alpha = 0.6) +
  scale_fill_manual(values = plot_colors) +
  labs(
    x = NULL,
    y = "Measured value",
    fill = "Category"
  ) +
  plot_theme
```

### 5. Explain the purpose in comments

Keep the section headings and comment only on important decisions. Explain why a step is written that way; skip line-by-line descriptions of functions and statements. For example, "Fix the category order to keep it consistent across plots" explains the reason for a choice.

### 6. Load packages at the top and write paths where they are used

- Keep all package-loading statements in the opening "Load packages" section. Use `package::function()` later when it helps identify a function's source.
- Load required packages with `library()` at the top. For optional features, store an availability flag from `requireNamespace(..., quietly = TRUE)` there, then call functions through `package::function()`. Omit custom wrappers for package loading.
- Use familiar path handling by default: set the working directory with `setwd("path/to/project")` in "Import data", then write relative paths directly in read and write calls. Use `file.path()` when constructing paths.
- Use the real directory of the user's open project. Write paths where they are used, without a central list of path variables or configuration settings.
- Use tools such as `here` when they help: for example, when the project already uses them or needs automatic location detection when run from different directories or moved. Treat them as optional dependencies and briefly explain their purpose in a comment at first use.

### 7. Keep stage objects separate and reuse shared themes and colors

- Distinguish objects such as `raw_data`, `clean_data`, `analysis_result`, `plot_data`, and `result_plot`. Preserve the original input and avoid repeatedly overwriting one variable across stages.
- Keep a clearly named object for each useful stage. Use pipes for one-time transformations within a stage and avoid unnecessary copies.
- Define and reuse `plot_theme` and `plot_colors` near the plots in "Visualize" when several figures share them. Put settings for a single plot directly in its layers. Use named vectors for categorical colors, with names matching the actual categories. Write dimensions, point sizes, opacity, and similar values where they are used.

### 8. Write parameters where they are used

- Put thresholds, groups, test methods, adjustment methods, confidence levels, resampling counts, and export settings directly in the relevant calls or filter expressions. This lets the user inspect results and edit the corresponding code in place.
- By default, omit parameter blocks at the top or in section 2, `params <- list(...)`, and configuration objects. Also omit scalar variables such as `min_fpkm`, `fdr_cutoff`, and `random_seed` that only pass a value onward. Use centralized configuration when the user explicitly requests it.
- Keep useful stage objects such as expression matrices, group tables, models, and filtered results. Write a filter's threshold once at the filtering step, then use that result or flag in subsequent calculations, plots, and exports.
- For example, write a low-expression filter as `rowSums(expression_fpkm >= 1) >= 3`, a differential filter as `mutate(is_differential = fdr <= 0.05 & abs(log_fc) >= 1)`, and an adjustment method as `p.adjust(p_value, method = "BH")`. These values illustrate parameter placement; choose the actual values for the current analysis.
- Set a seed only when there is a random operation, directly before that operation, such as `set.seed(123)`. Omit seed settings for deterministic calculations.
- Make `sessionInfo()` the last executable statement in a complete script to report the runtime environment.

### 9. Prefer familiar functions and write the steps directly

- Prefer familiar, straightforward functions that the user can run and adjust while inspecting results. Start with functions the user already uses and the field's usual APIs. Retain specialized analysis package calls needed for the task.
- Use less familiar functions when they offer a practical benefit, such as implementing the needed operation directly, substantially reducing code or manual work, or improving efficiency. At first use, add one comment explaining the function's purpose and why it was chosen.
- When functionality and effort are similar, choose the approach the user knows better and can read more easily. Avoid adding packages or abstractions just for code style.
- Keep data handling and formatting consistent throughout. Combine familiar base R and tidyverse functions as appropriate. Do not use `attach()`.
- Prefer existing vectorized functions and suitable data transformations. Use clear conventional `for` loops for simple repeated operations, with meaningful snake_case loop variables.
- By default, omit predefined custom functions, general wrappers, and anonymous functions inside `map_*()`. Write analysis, plotting, and export steps directly in their sections. Wrap steps in functions when the user explicitly requests functions or reusable interfaces.
- Keep each section's inputs, processing statements, and results easy to inspect. Avoid extra abstraction just to make every call look alike.

### 10. Add brief usage notes

After the code, include only these three items, each in one sentence:

- **Input format:** required file types, fields, and directory locations.
- **Parameters to change:** point to the statements or sections to edit, such as input paths, filter expressions, and plotting calls. Mention the seed statement when there are random operations.
- **Outputs:** generated files, objects, and save locations.

For a code fragment, describe only its input objects, parameters, and outputs. Keep the notes consistent with the actual code.

### 11. Lead with code and omit chatter

- By default, return one complete `r` code block followed by the brief notes in rule 10. Omit introductions, tutorials, line-by-line explanations, and repeated summaries.
- When the user requests a file, deliver an `.R` file and provide its link with the three usage notes.
- When the user requests an explanation or review, give a concise response suited to that request. Keep ordinary coding tasks focused on code.
- Ask a brief question only when missing information affects the implementation and the context does not provide it. Use real paths and field names when known. Label demonstration data as such.

### 12. Use journal-style figures and inspect the exports

Use the visual style of journal figures by default. Choose colors for the data type and the point the figure needs to convey. Balance appearance, distinguishability, and readability, and keep each category's color consistent across figures. Coordinate fonts, line widths, spacing, and legends. Adjust the layout for the plot type, then inspect exported figures and fix overlaps, clipping, and unbalanced proportions.

Choose color values, palettes, base themes, and label angles for the current content. Keep related figures visually consistent without mechanically applying a fixed template. When running a plotting task, inspect the actual exported figures before making adjustments. When delivering code, leave the plot settings easy to change.

### 13. Keep text inside figures concise

By default, retain concise axis names and units, necessary legends, panel labels, and statistical annotations required by the task. Omit large titles, subtitles, conclusions, method descriptions, parameter lists, and decorative text. Avoid duplicate labels and label only points of interest. Put explanations outside the figure; add a short title when it helps a figure stand on its own. After export, remove duplicate or irrelevant text. If necessary text is crowded or obscures data, adjust its position, font size, or layout.

## Check before returning

Review all thirteen rules internally. Check that the script uses familiar functions where possible, briefly explains useful less familiar functions, writes parameters where they are used, and omits unnecessary parameter blocks and function wrappers. Check that the figure design suits the content. When running plotting tasks, inspect the exports and confirm that every piece of text helps readers identify the data or understand the figure. Deliver the code and the three brief usage notes.
