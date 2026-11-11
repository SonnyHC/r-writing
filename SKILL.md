---
name: r-writing
description: Write, refactor, and review R scripts for clinicians who know basic R. Use familiar functions, short tidyverse pipelines, and descriptive variables for calculations and intermediate results. Put parameters where the reader will edit them. Organize full scripts into six sections and design figures for journal articles.
---

# R-writing

Write for clinicians who know basic R and want to run, inspect, and edit their own analyses. Choose the simplest script that does the requested job. Preserve the meaning of computations when refactoring, and keep choices that affect the results visible in the code.

## Thirteen writing rules

### 1. Organize code into six sections

In a complete script, use the following comment headings in order, each on its own line.

```r
# 1. Load packages
# 2. Import data
# 3. Clean data
# 4. Run analysis
# 5. Visualize
# 6. Export results
```

Load the required packages at the top. Read inputs with paths in the read calls, then group the cleaning steps together. Run the analysis in order using familiar functions, vectorized expressions, or clear loops. Set plot options near the plots. Save the requested outputs together and finish with `sessionInfo()`.

Keep this section order and preserve computational dependencies. When the user asks for a function or plotting fragment, return that fragment with the relevant section heading.

### 2. Follow tidyverse style

- Prefer `%>%` for data frame transformations and retain it as the default pipe. Combine it with familiar base R functions when that makes the code easy to understand and edit.
- Prefer `select()`, `filter()`, `mutate()`, `summarise()`, `arrange()`, `pivot_longer()`, and `*_join()` with explicit join keys.
- Combine consecutive transformations in a short pipeline when they read clearly together. Split the pipeline when a named intermediate would help the reader follow a step.
- Keep clear vectorized expressions where they fit. Avoid repeated data structure conversions just to add a pipe.
- Keep each pipeline short enough to follow and run on its own. Preserve steps that affect the result when simplifying the code.
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
- Give plot objects descriptive names. Set `plot`, dimensions, and units explicitly in `ggsave()`.

This boxplot uses columns already present in `clean_data`. The horizontal axis identifies each category, so the legend would repeat those labels. The plot has a name for export.

```r
distribution_plot <- ggplot(
  clean_data,
  aes(
    x = category,
    y = measured_value,
    fill = category
  )
) +
  geom_boxplot(show.legend = FALSE) +
  labs(
    x = NULL,
    y = "Measured value"
  )
```

### 5. Explain the purpose in comments

Keep the section headings and comment only on important decisions. Explain why a step is written that way. For example, "Fix the category order to keep it consistent across plots" explains a choice that the code alone might leave unclear.

### 6. Load packages at the top and write paths where they are used

- Keep all package-loading statements in the opening "Load packages" section. Use `package::function()` later when it helps identify a function's source.
- Load required packages with `library()` at the top. For optional features, store an availability flag from `requireNamespace(..., quietly = TRUE)` there, then call functions through `package::function()`. Omit custom wrappers for package loading.
- When the working directory is already the project root, use relative paths such as `"data/input.csv"` directly in read and write calls. Use `setwd()` in "Import data" only when the working directory needs to change, and use the actual project directory.
- Use `file.path()` when a path needs to be built from parts. A tool such as `here` is useful when the project already relies on it or the script needs to locate the project from different working directories. Briefly explain its purpose on first use.

### 7. Name useful results and share plot settings when needed

- Keep variables the calculation needs, such as a fitted model or a running total in a loop. A named intermediate can make a step easier to follow even when it is used only once. Also keep results the reader wants to inspect, reuse, or export.
- Remove redundant aliases and duplicate copies. Give raw and cleaned data distinct names when both are needed. Use the existing data frame directly when it already contains the columns for the next step or plot.
- Define and reuse `plot_theme` and `plot_colors` near the plots in "Visualize" when several figures share them. Put settings for a single plot directly in its layers. Use named vectors for categorical colors, with names matching the actual categories. Write dimensions, point sizes, opacity, and similar values where they are used.

### 8. Write parameters where they are used

- Put thresholds, groups, test methods, adjustment methods, confidence levels, resampling counts, and export settings directly in the relevant calls or filter expressions. This lets the user inspect results and edit the corresponding code in place.
- Write an important choice explicitly in its call, as in `p.adjust(p_value, method = "BH")` when the analysis uses that adjustment. Write a threshold once in its filter, then reuse the result or flag in subsequent calculations, plots, and exports.
- Omit variables that merely pass a value to a call without making the code easier to follow. Keep parameters in their calls by default, and use a separate parameter block or configuration list when the user requests centralized settings.
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

After the code, give three usage notes in one sentence each.

- State the required input file types, fields, and directory locations.
- Identify the statements or sections to edit, such as input paths, filter expressions, and plotting calls. Mention the seed statement when there are random operations.
- Name the output files and objects and give their save locations.

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

By default, retain concise axis names and units, necessary legends, panel labels, and statistical annotations required by the task. Omit large titles, subtitles, conclusions, method descriptions, parameter lists, and decorative text. Avoid duplicate labels and label only points of interest. Put explanations outside the figure. Add a short title when it helps a figure stand on its own. After export, remove duplicate or irrelevant text. If necessary text is crowded or obscures data, adjust its position, font size, or layout.

## Check before returning

Review all thirteen rules internally. Check that the script uses familiar functions where possible, briefly explains useful less familiar functions, writes parameters where they are used, and omits unnecessary parameter blocks and function wrappers. Check that the figure design suits the content. When running plotting tasks, inspect the exports and confirm that every piece of text helps readers identify the data or understand the figure. Deliver the code and the three brief usage notes.
