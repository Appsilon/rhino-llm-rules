# Rhino Framework Instructions for GitHub Copilot

## Project Structure

Rhino enforces a specific structure for robust and maintainable R Shiny applications:

- `app.R`: The entry point recognized by RStudio. Do not modify this file.
- `app/main.R`: The true application entry point where top-level UI and server are defined.
- `app/logic/`: Contains application code independent from Shiny (data processing, external connections, etc.).
- `app/view/`: Contains Shiny modules and UI code that depends on Shiny's reactive capabilities.
- `app/static/`: Contains static assets (CSS, JavaScript, images).
- `tests/testthat/`: Contains unit tests.

### Key Principles
- Place code that can be expressed without Shiny in the `logic` directory.
- Place code that relies on reactivity or UI components in the `view` directory.
- Use Shiny modules to isolate paired UI/Server code and prevent reactivity overlap.

## Importing and Exporting

### Importing

Use only `box::use` for imports. Using `library()` and `::` is forbidden.

`box::use` statements should be located at the top of the file with the following rules:
- Group packages and local scripts into separate `box::use` blocks.
- First block should import only R packages, second block should import only other scripts.
- Imports in each block should be sorted alphabetically.
- Using `[...]` is forbidden.
- All external functions in a script should be explicitly imported, including operators like `%>%`.
- A script should only import functions that it actually uses.

There are two ways to import packages or scripts:

1. **List specific imported functions** (preferred when importing ≤ 8 functions):
```r
box::use(
  dplyr[filter, select, mutate],
)

filter(mtcars, cyl > 4)
```

2. **Import package/module and access functions with `$`** (use when importing > 8 functions):
```r
box::use(
  dplyr,
)

dplyr$filter(mtcars, cyl > 4)
```

### When Moving Functions

When moving a function to a different script:
1. Add imports for all required functions to the destination file.
2. Use the appropriate import style in the new file (direct or using `$`).
3. Remove redundant imports from the original file.
4. Import the moved function in the original file.

### Exporting

- If a function is used only inside a script, it should not be exported.
- If a function is used by other scripts, export it by adding `#' @export` before the function definition.

## Shiny Modules in Rhino

When creating a new module in `app/view`, use this template:
```r
box::use(
  shiny[moduleServer, NS, tagList, div]
)

#' @export
ui <- function(id) {
  ns <- NS(id)
  tagList(
    # UI elements using ns() for namespacing
  )
}

#' @export
server <- function(id) {
  moduleServer(id, function(input, output, session) {
    # Server logic here
  })
}
```

### Module Best Practices

- Always use `ns()` to namespace input and output IDs in UI functions.
- Keep modules focused on a specific functionality.
- When accessing reactivity from parent modules, use proper input/output mechanisms.
- Import only the specific Shiny functions you need.
- Use logic functions from the `logic` directory for complex data processing.

## Unit Tests

All R unit tests are located in `tests/testthat`.

Guidelines:
- Create one test file per script, named `test-{script_name}.R`.
- Test both exported and non-exported functions.

### Testing Exported Functions
```r
box::use(
  app/logic/mymodule,
)

test_that("exported function works", {
  expect_equal(mymodule$exported_function(1), 2)
})
```

### Testing Non-Exported Functions
```r
box::use(
  app/logic/mymodule,
)

impl <- attr(mymodule, "namespace")

test_that("non-exported function works", {
  expect_equal(impl$internal_function(1), 2)
})
```

## Code Style

- Maximum line length is 100 characters.
- Use consistent indentation (2 spaces).
- Prefer explicit returns for readability in complex functions.
- Use snake_case for function and variable names.
- Prefer descriptive variable names over abbreviations.
- Add appropriate documentation for exported functions.

## Main Module Structure

The main module (`app/main.R`) should follow this pattern:
```r
box::use(
  shiny[bootstrapPage, div, moduleServer, NS, tags],
)

box::use(
  app/view/some_module,
)

#' @export
ui <- function(id) {
  ns <- NS(id)
  bootstrapPage(
    some_module$ui(ns("some_module"))
  )
}

#' @export
server <- function(id) {
  moduleServer(id, function(input, output, session) {
    some_module$server("some_module")
  })
}
```
