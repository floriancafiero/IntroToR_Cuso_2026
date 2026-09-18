# Session 1 — Working with real social-science data in R

**Course:** R for Research in the Humanities and Social Sciences - CUSO 
**Format:** online, 2 hours  
**Dataset:** `carData::SLID` — Survey of Labour and Income Dynamics  

## What we will do

This first session is built around a real social-science dataset rather than a toy example. `SLID` contains observations from the 1994 wave of the Canadian Survey of Labour and Income Dynamics in Ontario. Each row is a respondent; the variables include hourly wages, years of education, age, sex, and language. There are also missing values, especially for wages.

That makes it useful for learning R because the structure resembles many research datasets in the social sciences, psychology, education, and related fields:

> **individuals → measurements → groups → missing data → summaries → figures**

The goal today is not to learn every feature of R. The goal is to understand what R is doing, line by line, and to acquire a small vocabulary that already lets us answer a real empirical question.

Our workflow will be:

> **Question → inspect → clean → transform → summarise → visualise → interpret**

The main empirical question will be:

> **How is education associated with hourly wages in this sample?**

We will also ask whether the pattern looks similar across groups. We will remain descriptive today: association is not causation.

---

# Before the session

You need **R** and **RStudio** installed.

Run this once before class:

```r
# install.packages() downloads packages to your computer.
# You normally do this only once, not every time you use R.

install.packages(c("tidyverse", "carData"))
```

During the course we will load the packages with `library()` or access a dataset directly from a package.

A useful distinction:

```r
# DONE ONCE on a computer:
# install.packages("tidyverse")

# DONE AGAIN after restarting R:
library(tidyverse)
```

Open the `R_Cuso_2026` folder as an RStudio project or working folder. We will avoid hard-coded paths such as:

```r
# Do NOT copy this kind of path into your research scripts:
# setwd("C:/Users/myname/Desktop/project")
```

Relative paths and projects make analyses easier to reproduce on another computer.

---

# 1. R is a calculator, but one that remembers things — 0:00–0:15

We begin in the Console or in an R script.

Anything following `#` is a **comment**. R ignores it. Comments are for humans — including your future self.

```r
# Addition
1 + 1

# Multiplication
3 * 4

# Division
10 / 4

# Power
2^3

# Parentheses work as expected
(2 + 3) * 4
```

When you run one of these lines, R:

1. reads the expression;
2. evaluates it;
3. prints the result.

## Numbers and text are different

```r
# A numeric value
12

# A character string: the quotation marks matter
"12"

# Numeric addition works
12 + 1
```

Now deliberately create an error:

```r
# This does NOT work because "12" is text, not a number.
"12" + 1
```

Errors are normal. An error message tells us where R could not understand or execute an instruction.

## Store a value in an object

```r
# <- is the assignment operator.
# Read this as: "put the value 10 into an object called x".

x <- 10

# Typing the object's name displays its current value.
x

# We can reuse it.
x + 5

# We can overwrite it.
x <- 100
x
```

Object names should tell us what they contain.

```r
# Legal, but not informative
x <- 14

# Much easier to understand six months later
mean_years_education <- 14
mean_years_education
```

## Functions

A function takes input, performs an operation, and returns output.

```r
# sqrt() takes one argument.
sqrt(16)

# round() can take several arguments.
round(x = 3.1415926, digits = 2)
```

The general shape is:

```text
function(argument_1, argument_2, ...)
```

You do not need to memorise every function. You need to learn how to read a function call.

```r
# Open the documentation for a function.
?round
```

---

# 2. Load the tools and the dataset — 0:15–0:25

## Load the tidyverse

```r
# The tidyverse contains packages for data manipulation and visualisation.
library(tidyverse)
```

You may see messages about packages being attached or functions being masked. That is normal.

## Load the SLID dataset

The dataset is distributed with the `carData` R package.

```r
# data() loads a dataset included in an R package.
# We tell R both the dataset name and the package in which it lives.

data("SLID", package = "carData")
```

We now have an object called `SLID`.

```r
# Display the first few rows.
head(SLID)
```

For the rest of the session, we will use a lower-case object name. This does not change the data; it simply creates another reference to the same values.

```r
# Store the dataset under a name we will use throughout the course.
slid <- SLID

# Check that it exists.
slid
```

## What does one row mean?

This question should always come before statistical analysis.

```r
# How many observations (rows)?
nrow(slid)

# How many variables (columns)?
ncol(slid)

# Both at once: rows, then columns.
dim(slid)
```

Here, **one row corresponds to one respondent** in this extract of the survey.

That means that when we calculate something such as a mean age, the observational unit is the respondent — not a province, a household, or a year.

---

# 3. Never analyse a dataset you have not inspected — 0:25–0:40

## Look at names

```r
# Column names
names(slid)
```

The variables are:

```text
wages       hourly wage rate
education   years of schooling
age         age in years
sex         recorded sex category
language    language category
```

## Look at the first rows

```r
# First six rows
head(slid)

# First ten rows
head(slid, 10)
```

## Look at structure and data types

```r
# glimpse() gives a compact overview of the dataset.
glimpse(slid)
```

You should see types such as:

```text
<dbl>  numeric value, usually allowing decimals
<int>  integer
<fct>  factor: a categorical variable with defined levels
<chr>  character/text
```

Data types matter. Taking the mean of `age` makes sense. Taking the mean of `language` does not.

Check individual variables:

```r
# What class is each variable?
class(slid$wages)
class(slid$age)
class(slid$sex)
class(slid$language)
```

## A quick summary

```r
# summary() adapts its output to each variable type.
summary(slid)
```

Do not rush past this output. Ask:

- Are values in plausible ranges?
- Are there missing observations?
- Which variables are continuous?
- Which are categorical?
- Does the coding match what we think the variable means?

This habit catches many errors before they become results.

---

# 4. Columns are vectors — and missing values are values too — 0:40–0:52

A data-frame column can be extracted with `$`.

```r
# Extract the age column.
slid$age
```

This is a **vector**: an ordered collection of values.

```r
# Save it as a separate object.
age <- slid$age

# Display the first ten values.
head(age, 10)

# Number of elements in the vector.
length(age)
```

We can apply functions to a vector.

```r
# Mean age in the sample.
mean(age)

# Median age.
median(age)

# Standard deviation.
sd(age)
```

Now try wages:

```r
# This returns NA.
mean(slid$wages)
```

Why?

Because the wage variable contains missing values.

```r
# is.na() returns TRUE when a value is missing.
is.na(slid$wages)

# TRUE is treated as 1 and FALSE as 0 when summed,
# so this counts the number of missing wage observations.
sum(is.na(slid$wages))

# The proportion of missing wage values.
mean(is.na(slid$wages))
```

To compute a mean while excluding missing observations from **that calculation**:

```r
mean(slid$wages, na.rm = TRUE)
```

Read `na.rm = TRUE` as:

> "remove missing values before applying this function."

Other examples:

```r
median(slid$wages, na.rm = TRUE)
sd(slid$wages, na.rm = TRUE)
min(slid$wages, na.rm = TRUE)
max(slid$wages, na.rm = TRUE)
```

Important research point: `na.rm = TRUE` is a computational instruction, **not a missing-data strategy**. In a real study we would also ask why wages are missing and whether missingness is related to other variables.

---

# 5. Manipulate observations with `dplyr` — 0:52–1:07

Most research questions require us to keep, transform, or combine variables and observations.

The `dplyr` package, loaded with the tidyverse, uses verbs that correspond closely to those operations.

## `filter()` keeps rows

```r
# Keep respondents aged 30 or older.
filter(slid, age >= 30)
```

A condition produces TRUE/FALSE values.

```r
# One TRUE/FALSE value per respondent.
slid$age >= 30
```

`filter()` retains the rows for which the condition is TRUE.

Several conditions can be combined.

```r
# Respondents aged 30–40 with an observed wage.
filter(
  slid,
  age >= 30,
  age <= 40,
  !is.na(wages)
)
```

Useful operators:

```text
==   equal to
!=   not equal to
>    greater than
<    less than
>=   greater than or equal to
<=   less than or equal to
!    logical NOT
&    AND
|    OR
```

For a categorical variable:

```r
# Keep respondents in the French-language category.
filter(slid, language == "French")
```

## `select()` keeps columns

```r
# Keep only the variables required for a simple wage analysis.
select(slid, wages, education, age, language)
```

## `arrange()` sorts rows

```r
# Lowest observed wages first.
arrange(slid, wages)

# Highest observed wages first.
arrange(slid, desc(wages))
```

Notice that none of these commands modifies `slid`. They return a new result.

---

# Short break — 1:07–1:12

Five minutes away from the screen.

---

# 6. The pipe: write code in the order you think — 1:12–1:28

Suppose we want to:

1. start with `slid`;
2. keep respondents with an observed wage;
3. keep only a few variables;
4. sort from highest to lowest wage.

We could nest functions, but it quickly becomes difficult to read.

The native R pipe `|>` means approximately **"then"**.

```r
slid |>
  # THEN keep rows with a known wage
  filter(!is.na(wages)) |>
  # THEN keep these four columns
  select(wages, education, age, language) |>
  # THEN sort by wage, largest first
  arrange(desc(wages))
```

Read the code aloud:

> Take `slid`, then filter, then select, then arrange.

The following two pieces of code are equivalent:

```r
# Function style
filter(slid, age >= 30)
```

```r
# Pipe style
slid |>
  filter(age >= 30)
```

The pipe becomes useful when the analysis has several consecutive steps.

## Save the result

```r
# Create a new dataset containing respondents for whom wage is observed.

slid_complete_wage <- slid |>
  filter(!is.na(wages))

# Compare the number of observations before and after filtering.
nrow(slid)
nrow(slid_complete_wage)
```

We have **not** overwritten the original data. Keeping the raw object unchanged is usually a good habit.

## `mutate()` creates variables

A common task in research is deriving a variable from existing measurements.

For example, create broad age groups:

```r
slid_with_age_group <- slid |>
  mutate(
    # case_when() checks conditions from top to bottom.
    age_group = case_when(
      age < 30 ~ "under 30",
      age < 45 ~ "30–44",
      age < 60 ~ "45–59",
      age >= 60 ~ "60+"
    )
  )

# Look at the newly created variable.
slid_with_age_group |>
  select(age, age_group) |>
  head(20)
```

The original `slid` object still has only the original variables:

```r
names(slid)
```

The modified object contains the new one:

```r
names(slid_with_age_group)
```

### Mini exercise

Create an object called `working_sample` that:

- keeps only respondents with observed wages;
- keeps only `wages`, `education`, `age`, `sex`, and `language`;
- sorts rows from highest to lowest wage.

Try it before opening the solution.

<details>
<summary>Solution</summary>

```r
working_sample <- slid |>
  # Keep observations for which wages are available.
  filter(!is.na(wages)) |>
  # Keep only variables useful for the current question.
  select(wages, education, age, sex, language) |>
  # Put the highest wages first.
  arrange(desc(wages))

working_sample
```

</details>

---

# 7. Go from individual observations to evidence about groups — 1:28–1:42

## Count observations

```r
# Number of respondents in each language category.
slid |>
  count(language)
```

`count()` is useful whenever you need to understand the composition of a dataset.

We can immediately turn counts into percentages of **this sample**:

```r
slid |>
  count(language) |>
  mutate(
    # n is the count created by count().
    # sum(n) is the total sample size.
    percent = 100 * n / sum(n)
  )
```

Be precise in interpretation: these are proportions in the dataset we are analysing. Survey weights and sampling design would matter before treating them as population estimates.

## One summary for the entire dataset

```r
slid |>
  summarise(
    mean_age = mean(age, na.rm = TRUE),
    mean_wage = mean(wages, na.rm = TRUE)
  )
```

`summarise()` reduces many rows to one or more summary values.

## Summaries by group

Now ask:

> What do observed wages look like in each language category?

```r
slid |>
  # Define the groups first.
  group_by(language) |>
  # Then perform the same calculations independently in each group.
  summarise(
    # Total number of rows in the group
    n = n(),
    # Number with an observed wage
    n_wage = sum(!is.na(wages)),
    # Mean among observed wages
    mean_wage = mean(wages, na.rm = TRUE),
    # Median is often informative for skewed quantities such as income/wages
    median_wage = median(wages, na.rm = TRUE),
    # Standard deviation describes dispersion
    sd_wage = sd(wages, na.rm = TRUE)
  )
```

The logic is important:

```text
group_by(language)
        ↓
English observations  → summarise()
French observations   → summarise()
Other observations    → summarise()
        ↓
one row per group
```

Now look at education:

```r
slid |>
  group_by(language) |>
  summarise(
    n = n(),
    mean_education = mean(education, na.rm = TRUE),
    sd_education = sd(education, na.rm = TRUE)
  )
```

A crucial lesson: a difference between group means is a **description**. It does not by itself tell us why the groups differ.

---

# 8. Make the observations visible with `ggplot2` — 1:42–1:58

`ggplot2` builds figures from layers. Instead of memorising complete plotting commands, we will construct a figure step by step.

The three pieces to remember are:

```text
data        which data frame?
aesthetics  which variables become x, y, colour, etc.?
geometry    points, bars, lines, boxes, ...?
```

## Step 1 — declare the data

```r
# This creates an empty plotting object.
ggplot(data = slid)
```

Nothing is visible yet because we have not said what to plot.

## Step 2 — map a variable to an axis

```r
# aes() means aesthetic mapping.
# We map wages to the x-axis.
ggplot(data = slid, aes(x = wages))
```

Still no marks: we have defined an axis but not a geometry.

## Step 3 — add a geometry

```r
# A histogram shows the distribution of one numerical variable.
ggplot(data = slid, aes(x = wages)) +
  geom_histogram()
```

You may see a warning about removed rows containing missing values. That warning is useful: `ggplot2` is telling us that some wage observations could not be drawn.

We can make the analytic choice explicit:

```r
slid |>
  filter(!is.na(wages)) |>
  ggplot(aes(x = wages)) +
  geom_histogram()
```

## Relationship between two numerical variables

Our main question concerns education and wages.

```r
# One point represents one respondent with both values available.
ggplot(slid, aes(x = education, y = wages)) +
  geom_point()
```

With thousands of observations, points overlap. We can make them partly transparent.

```r
ggplot(slid, aes(x = education, y = wages)) +
  geom_point(alpha = 0.25)
```

`alpha = 0.25` is **not** mapped to a variable. It is a fixed visual setting, so it is written outside `aes()`.

Compare:

```r
# FIXED property: every point gets the same transparency.
ggplot(slid, aes(x = education, y = wages)) +
  geom_point(alpha = 0.25)
```

with:

```r
# MAPPED property: colour now represents the variable language.
ggplot(slid, aes(x = education, y = wages, colour = language)) +
  geom_point(alpha = 0.25)
```

This distinction — **inside vs outside `aes()`** — is one of the most important things to understand in `ggplot2`.

## Add a descriptive trend

```r
ggplot(slid, aes(x = education, y = wages)) +
  geom_point(alpha = 0.20) +
  # method = "lm" adds a straight fitted line.
  # We will study the statistical model behind this in Session 2.
  geom_smooth(method = "lm")
```

At this stage the line is a visual summary of association. Do not interpret it as evidence that education itself *causes* a particular wage increase.

## Compare groups without putting everything on one plot

Facets create one panel per category.

```r
ggplot(slid, aes(x = education, y = wages)) +
  geom_point(alpha = 0.20) +
  geom_smooth(method = "lm") +
  facet_wrap(~ language)
```

The formula:

```r
~ language
```

means: create panels according to values of `language`.

## Add human-readable labels

```r
wage_plot <- slid |>
  filter(!is.na(wages)) |>
  ggplot(aes(x = education, y = wages)) +
  geom_point(alpha = 0.20) +
  geom_smooth(method = "lm") +
  facet_wrap(~ language) +
  labs(
    title = "Education and hourly wages in the SLID sample",
    subtitle = "Ontario, 1994; panels show recorded language category",
    x = "Years of education",
    y = "Hourly wage",
    caption = "Source: Survey of Labour and Income Dynamics (SLID)"
  ) +
  theme_minimal()

wage_plot
```

Notice that a plot can itself be stored in an object:

```r
class(wage_plot)
```

## Save a publication-quality image

Research code should be able to reproduce output files as well as results on screen.

```r
# Create a folder if it does not already exist.
dir.create("figures", showWarnings = FALSE)

# Save the plot object rather than taking a screenshot.
ggsave(
  filename = "figures/education_and_wages.png",
  plot = wage_plot,
  width = 8,
  height = 5,
  dpi = 300
)
```

This is much more reproducible than manually exporting the figure from RStudio.

---

# 9. Final challenge — 1:58–2:00 and optional continuation

Choose one question. If time is short, begin it now and finish after class.

### A — Education and wages

> How does hourly wage vary with years of education in this sample?

### B — Age and wages

> What does the relationship between age and observed wages look like?

### C — Group composition

> How are respondents distributed across language categories, and how do their observed wage distributions differ?

For your chosen question, produce:

1. one filtered or transformed dataset if necessary;
2. one summary table;
3. one figure;
4. one sentence describing what the data show.

A possible skeleton:

```r
# STEP 1 — choose the observations you need
analysis_data <- slid |>
  filter(!is.na(wages))

# STEP 2 — produce a numerical summary
summary_table <- analysis_data |>
  group_by(language) |>
  summarise(
    n = n(),
    mean_wage = mean(wages),
    median_wage = median(wages)
  )

summary_table

# STEP 3 — produce a figure
analysis_plot <- analysis_data |>
  ggplot(aes(x = education, y = wages)) +
  geom_point(alpha = 0.20) +
  geom_smooth(method = "lm") +
  theme_minimal()

analysis_plot
```

A careful descriptive sentence would look like:

> "In this sample, respondents with more years of education tend to report higher observed hourly wages."

That is different from:

> "Additional education causes higher wages."

The second statement requires a causal research design that we have not established.

---

# 10. What you now know

The amount of syntax introduced today is deliberately small.

```r
# Inspect
head()
glimpse()
summary()
nrow()
names()

# Missingness
is.na()

# Manipulate
filter()
select()
arrange()
mutate()

# Group and summarise
count()
group_by()
summarise()

# Visualise
ggplot()
geom_histogram()
geom_point()
geom_smooth()
facet_wrap()
labs()

# Save output
ggsave()
```

The more important idea is the workflow:

```text
What is one row?
      ↓
What are the variables and their types?
      ↓
What is missing?
      ↓
Which observations do I need?
      ↓
What numerical summary answers my question?
      ↓
What figure lets me see the observations?
      ↓
What can I legitimately conclude?
```

In Session 2 we will move from a visible association to **statistical evidence**: estimates, uncertainty, confidence intervals, and regression models.

---

# Optional: how this translates to your own CSV file

Today we used a dataset bundled with an R package so that everybody could run exactly the same code. Your own research data will often arrive as CSV or Excel files.

For a CSV file, the workflow changes only at the import step:

```r
# Example only — replace the path with your own file.
my_data <- read_csv("data/my_data.csv")

# Immediately inspect it.
glimpse(my_data)
summary(my_data)
```

For an Excel file:

```r
# Install readxl once if necessary:
# install.packages("readxl")

library(readxl)

# Example only:
my_data <- read_excel("data/my_data.xlsx")
```

Everything that follows — `filter()`, `mutate()`, `group_by()`, `ggplot()` — works with the same logic.





## Dataset source

`SLID` is distributed in the `carData` package and is described as data from the 1994 wave of the Canadian Survey of Labour and Income Dynamics for Ontario, prepared from the public-use dataset made available by Statistics Canada.
