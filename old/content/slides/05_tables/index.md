---
title: "APA Tables"
subtitle: "https://osf.io/r9k5u/"
author: "Emorie D Beck"
date: "April 23, 2020"
slides:
  df_print: paged
  highlight_style: dracula
  theme: black
  widescreen: yes
institution: Washington University in St. Louis
---

# 

Download the tables.zip file from this link:  
https://osf.io/r9k5u/  
and unzip it on your Desktop.  
 
### APA Tables  
In psychology, we must work within the confines of APA style. Although these guidelines have been updated, the style guide remains quite similar to earlier guidelines with respect to tables.  

But psychology research is heterogeneous and expectations for modern tables require combining multiple models in creative ways.  

Small tweaks to data or model arguments can spell disaster for creating a table. It's easy to make mistakes in copying values or matching different models to their respective rows and columns.

Thankfully, the increasing popularity of `R` has been coupled with more methods for creating a reproducible workflow that includes tables.  

---

### Outline  
In this tutorial, we will directly cover 3 different use cases, while a few others will be included in supplementary materials.  

Personally, I favor non-automated tools, so we will cover the following packages:  
- `kable` + `kableExtra` (<a href="http://haozhu233.github.io/kableExtra/awesome_table_in_html.html">html</a> and <a href="https://haozhu233.github.io/kableExtra/awesome_table_in_pdf.pdf">LaTeX</a>)  
- <a href ="https://github.com/crsh/papaja">`papaja`</a>  

Using these packages will build on earlier tutorials using `tidyr`, `dplyr`, workflow, and `purrr` and round out our discuss on data presentation using `ggplot2`.  

For less flexible but more accessible tables see:  
- <a href="https://cran.r-project.org/web/packages/apaTables/vignettes/apaTables.html">`apaTable`</a>  
- <a href="http://www.strengejacke.de/sjPlot/">`sjPlot`</a>  
- <a href="https://cran.r-project.org/web/packages/corrplot/vignettes/corrplot-intro.html">`corrplot`</a>  

---

### Important Tools  
Although it doesn't cover all models, the `broom` and `broom.mixed` family of packages will provide easy to work with estimates of nearly all types of models and will also provide the model terms that are ideal for most APA tables, including estimates, standard errors, and confidence intervals.  

`lavaan` models are slightly more complicated, but it's actually relatively easy to deal with them (and how to extract their terms), assuming that you understand the models you are running.  

---

### Data  
The data we're going to use are from the teaching sample from the German Socioeconomic Panel Study. These data have been pre-cleaned (see earlier workshop on workflow and creating guidelines for tips).  

The data we'll use fall into three categories:  
1. **Personality trait composites:** Negative Affect, Positive Affect, Self-Esteem, CESD Depression, and Optimism. These were cleaned, reversed coded, and composited prior to being included in this final data set.  
2. **Outcomes:** Moving in with a partner, marriage, divorce, and child birth. These were cleaned, coded as 1 (occurred) or 0 (did not occur) according to whether an outcome occurred for each individual or not *after* each possible measured personality year. Moreover, people who experienced these outcomes prior to the target personality year are excluded.  
3. **Covariates:** Age, gender (0 = male, 1 = female, education (0 = high school or below, 1 = college, 2 = higher than college), gross wages, self-rated health, smoking (0 = never smoked 1 = ever smoked), exercise, BMI, religion, parental education, and parental occupational prestige (ISEI). Each of these were composited for all available data up to the measured personality years.  
---

### Data  

```{.r .code-style}
data_source <- "https://github.com/emoriebeck/R-tutorials/raw/master/99_archive/wustl_r_workshops/tables.zip"
data_dest <- "~/Desktop/tables.zip"
download.file(data_source, data_dest)
```


```{.r .code-style}
wd <- "~/Desktop/tables"

(gsoep <- sprintf("%s/data/gsoep.csv", wd) %>% read_csv())
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
  </script>
</div>


### Basic Lessons: One DV/Outcome, Multiple Model Terms  
We'll start with a basic case, predicting who has a child from personality, both with and without control variables. 

Becauce outcome variables are binary, we'll use logistic regression.  

The basic form of the model is: $\log\Big(\frac{p_i}{1-p_i}\Big) = b_0 + b_1X_1 + b_2X_2 ... b_pXp$

In other words, we're predicting the log odds of having a child from a linear combination of predictor variables.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Set up the data frame  
First, we'll use some of what we learned in the `purrr` workshop to set ourselves up to be able to create these tables easily, using `group_by()` and `nest()` to create nested data frames for our target personality + outcome combinations. To do this, we'll also use what you learned about `filter()` and `mutate()`.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Set up the data frame  
First, we'll use some of what we learned in the `purrr` workshop to set ourselves up to be able to create these tables easily, using `group_by()` and `nest()` to create nested data frames for our target personality + outcome combinations. To do this, we'll also use what you learned about `filter()` and `mutate()`.  


```{.r .code-style}
gsoep_nested1 <- gsoep %>%
  filter(Outcome == "chldbrth") %>%
  group_by(Trait, Outcome) %>%
  nest() %>%
  ungroup()
gsoep_nested1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["data"],"name":[3],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"<tibble>"},{"1":"chldbrth","2":"PA","3":"<tibble>"},{"1":"chldbrth","2":"SE","3":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

First, let's pause and see what we have. We now have a data frame with 3 columns (Outcome, Trait, and data) and 4 rows. The data column is of class list, meaning it's a "list column" that contains a `tibble` in each cell. This means that we can use `purrr` functions to run operations on each of these data frames individually but without having to copy and paste the same operation multiple times for each model we want to run.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Run Models  
To run the models, I like to write short functions that are easier to read than including a local function within a call to `purrr::map()`. Here, we're just going to write a simple function to predict child birth from personality.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Run Models  
To run the models, I like to write short functions that are easier to read than including a local function within a call to `purrr::map()`. Here, we're just going to write a simple function to predict child birth from personality.  


```{.r .code-style}
mod1_fun <- function(d){
  d$o_value <- factor(d$o_value)
  glm(o_value ~ p_value, data = d, family = binomial(link = "logit"))
}

gsoep_nested1 <- gsoep_nested1 %>%
  mutate(m = map(data, mod1_fun))
gsoep_nested1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["data"],"name":[3],"type":["list"],"align":["right"]},{"label":["m"],"name":[4],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"<tibble>","4":"<S3: glm>"},{"1":"chldbrth","2":"DEP","3":"<tibble>","4":"<S3: glm>"},{"1":"chldbrth","2":"NegAff","3":"<tibble>","4":"<S3: glm>"},{"1":"chldbrth","2":"PA","3":"<tibble>","4":"<S3: glm>"},{"1":"chldbrth","2":"SE","3":"<tibble>","4":"<S3: glm>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Now, when we look at the nested frame, we see an additional column, which is also a list, but this column contains `<glm>` objects rather than `tibbles`.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Get Key Terms  
Now that we have the models, we want to get our key terms. I'm a big fan of using the function `tidy` from the `broom` package to do this. Bonus because it plays nicely with `purrr`. Double bonus because it will give us confidence intervals, which I generally prefer over p-values and standard erorrs because I find them more informative.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Get Key Terms  
Now that we have the models, we want to get our key terms. I'm a big fan of using the function `tidy` from the `broom` package to do this. Bonus because it plays nicely with `purrr`. Double bonus because it will give us confidence intervals, which I generally prefer over p-values and standard erorrs because I find them more informative.  

```{.r .code-style}
gsoep_nested1 <- gsoep_nested1 %>%
  mutate(tidy = map(m, ~tidy(., conf.int = T)))
```

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Get Key Terms  

```{.r .code-style}
gsoep_nested1 <- gsoep_nested1 %>%
  mutate(tidy = map(m, ~tidy(., conf.int = T)))
gsoep_nested1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["data"],"name":[3],"type":["list"],"align":["right"]},{"label":["m"],"name":[4],"type":["list"],"align":["right"]},{"label":["tidy"],"name":[5],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"PA","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"SE","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Note that what I've used here is a local function, meaning that I've used the notation `~`function(., arguments). The tilda tells `R` we want a local function, and the `.` tells R to use the mapped `m` column as the function input.  

Now we have a fifth column, which is a list column called `tidy` that contains a `tibble`, just like the `data` column.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  
Now we are finally ready to create a table! I'm going to use `kable` + `kableExtra` to do this in steps.  

First, we'll unnest the `tidy` column from our data frame. Before doing so, we will drop the `data` and `m` columns because they've done their work for now.  

```{.r .code-style}
tidy1 <- gsoep_nested1 %>%
  select(-data, -m) %>%
  unnest(tidy)
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[9],"type":["dbl"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"(Intercept)","4":"-4.30140097","5":"0.18588797","6":"-23.139749","7":"1.843845e-118","8":"-4.67170199","9":"-3.9428698"},{"1":"chldbrth","2":"OP","3":"p_value","4":"0.51772803","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"0.40460371","9":"0.6326562"},{"1":"chldbrth","2":"DEP","3":"(Intercept)","4":"-3.47523815","5":"0.14605559","6":"-23.793941","7":"3.858771e-125","8":"-3.76458852","9":"-3.1919380"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"0.14250618","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"0.07164275","9":"0.2143148"},{"1":"chldbrth","2":"NegAff","3":"(Intercept)","4":"-3.58156233","5":"0.13150068","6":"-27.236074","7":"2.429835e-163","8":"-3.84123426","9":"-3.3257168"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"0.10688426","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"0.01415774","9":"0.1988123"},{"1":"chldbrth","2":"PA","3":"(Intercept)","4":"-5.79331962","5":"0.21877964","6":"-26.480159","7":"1.640624e-154","8":"-6.22838107","9":"-5.3707054"},{"1":"chldbrth","2":"PA","3":"p_value","4":"0.67677729","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"0.56895170","9":"0.7863265"},{"1":"chldbrth","2":"SE","3":"(Intercept)","4":"-4.10885973","5":"0.34183754","6":"-12.019920","7":"2.792429e-33","8":"-4.80260759","9":"-3.4616551"},{"1":"chldbrth","2":"SE","3":"p_value","4":"0.08533618","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"-0.02642707","9":"0.2023474"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

As you can see, we now have lots of information about our model terms, which are already nicely indexed by Outcome and Trait combinations. 

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  
But before we're ready to create a table, we have to make a few considerations: 
- What is our target term? In this case "p_value" which is the change in log odds associated with a 1 unit increase/decrease in p_value.  
- How will we denote significance? In this case, we'll use confidence intervals whose signs match. We'll then bold these terms for our table. 
- What is the desired final structure for the table? I'd like columns for Trait, estimate (b), and confidence intervals (CI) formatted to two decimal places and bolded if significant. I'd also like a span header denoting that the outcome measure is child birth.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  
But before we're ready to create a table, we have to make a few considerations: 
- What is our target term? In this case "p_value" which is the change in log odds associated with a 1 unit increase/decrease in p_value.  

```{.r .code-style}
tidy1 <- tidy1 %>% filter(term == "p_value")
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[9],"type":["dbl"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"0.51772803","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"0.40460371","9":"0.6326562"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"0.14250618","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"0.07164275","9":"0.2143148"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"0.10688426","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"0.01415774","9":"0.1988123"},{"1":"chldbrth","2":"PA","3":"p_value","4":"0.67677729","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"0.56895170","9":"0.7863265"},{"1":"chldbrth","2":"SE","3":"p_value","4":"0.08533618","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"-0.02642707","9":"0.2023474"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  
- How will we denote significance? In this case, we'll use confidence intervals whose signs match. We'll then bold these terms for our table. 


```{.r .code-style}
tidy1 <- tidy1 %>% mutate(sig = ifelse(sign(conf.low) == sign(conf.high), "sig", "ns"))
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[9],"type":["dbl"],"align":["right"]},{"label":["sig"],"name":[10],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"0.51772803","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"0.40460371","9":"0.6326562","10":"sig"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"0.14250618","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"0.07164275","9":"0.2143148","10":"sig"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"0.10688426","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"0.01415774","9":"0.1988123","10":"sig"},{"1":"chldbrth","2":"PA","3":"p_value","4":"0.67677729","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"0.56895170","9":"0.7863265","10":"sig"},{"1":"chldbrth","2":"SE","3":"p_value","4":"0.08533618","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"-0.02642707","9":"0.2023474","10":"ns"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  
- What is the desired final structure for the table? I'd like columns for Trait, estimate (b), and confidence intervals (CI) formatted to two decimal places and bolded if significant. I'd also like a span header denoting that the outcome measure is child birth.  

Before we do this, though, we need to convert our log odds to odds ratios, using the `exp()` function.  


```{.r .code-style}
tidy1 <- tidy1 %>%
  mutate_at(vars(estimate, conf.low, conf.high), exp) 
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[9],"type":["dbl"],"align":["right"]},{"label":["sig"],"name":[10],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"1.678210","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"1.4987085","9":"1.882605","10":"sig"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"1.153160","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"1.0742715","9":"1.239013","10":"sig"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"1.112805","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"1.0142584","9":"1.219953","10":"sig"},{"1":"chldbrth","2":"PA","3":"p_value","4":"1.967527","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"1.7664143","9":"2.195317","10":"sig"},{"1":"chldbrth","2":"SE","3":"p_value","4":"1.089083","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"0.9739191","9":"1.224273","10":"ns"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  

Now, we can format them.  

```{.r .code-style}
tidy1 <- tidy1 %>%
  mutate_at(vars(estimate, conf.low, conf.high), ~sprintf("%.2f", .)) 
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["chr"],"align":["left"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["chr"],"align":["left"]},{"label":["conf.high"],"name":[9],"type":["chr"],"align":["left"]},{"label":["sig"],"name":[10],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"1.68","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"1.50","9":"1.88","10":"sig"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"1.15","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"1.07","9":"1.24","10":"sig"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"1.11","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"1.01","9":"1.22","10":"sig"},{"1":"chldbrth","2":"PA","3":"p_value","4":"1.97","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"1.77","9":"2.20","10":"sig"},{"1":"chldbrth","2":"SE","3":"p_value","4":"1.09","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"0.97","9":"1.22","10":"ns"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

`sprintf()` is my favorite base `R` formatting function. "%.2f" means I'm asking it to take a floating point number and include 2 digits after the "." and 0 before. We can now see that the `estimate`, `conf.low`, and `conf.high` columns are of class `<chr>` instead of `<dbl>`. 

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  

But now we need to create our confidence intervals.  


```{.r .code-style}
tidy1 <- tidy1 %>%
  mutate(CI = sprintf("[%s, %s]", conf.low, conf.high))
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["chr"],"align":["left"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["chr"],"align":["left"]},{"label":["conf.high"],"name":[9],"type":["chr"],"align":["left"]},{"label":["sig"],"name":[10],"type":["chr"],"align":["left"]},{"label":["CI"],"name":[11],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"1.68","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"1.50","9":"1.88","10":"sig","11":"[1.50, 1.88]"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"1.15","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"1.07","9":"1.24","10":"sig","11":"[1.07, 1.24]"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"1.11","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"1.01","9":"1.22","10":"sig","11":"[1.01, 1.22]"},{"1":"chldbrth","2":"PA","3":"p_value","4":"1.97","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"1.77","9":"2.20","10":"sig","11":"[1.77, 2.20]"},{"1":"chldbrth","2":"SE","3":"p_value","4":"1.09","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"0.97","9":"1.22","10":"ns","11":"[0.97, 1.22]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  

And bold the significant confidence intervals and estimates.  


```{.r .code-style}
tidy1 <- tidy1 %>%
  mutate_at(vars(estimate, CI), ~ifelse(sig == "sig", sprintf("<strong>%s</strong>", .), .))
tidy1
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["chr"],"align":["left"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["chr"],"align":["left"]},{"label":["conf.high"],"name":[9],"type":["chr"],"align":["left"]},{"label":["sig"],"name":[10],"type":["chr"],"align":["left"]},{"label":["CI"],"name":[11],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"p_value","4":"<strong>1.68<\/strong>","5":"0.05816468","6":"8.901073","7":"5.530926e-19","8":"1.50","9":"1.88","10":"sig","11":"<strong>[1.50, 1.88]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"<strong>1.15<\/strong>","5":"0.03638957","6":"3.916127","7":"8.998282e-05","8":"1.07","9":"1.24","10":"sig","11":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"<strong>1.11<\/strong>","5":"0.04710173","6":"2.269222","7":"2.325486e-02","8":"1.01","9":"1.22","10":"sig","11":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"PA","3":"p_value","4":"<strong>1.97<\/strong>","5":"0.05544973","6":"12.205240","7":"2.914484e-34","8":"1.77","9":"2.20","10":"sig","11":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth","2":"SE","3":"p_value","4":"1.09","5":"0.05832303","6":"1.463164","7":"1.434224e-01","8":"0.97","9":"1.22","10":"ns","11":"[0.97, 1.22]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

This reads as "for both the estimate and the CI columns, if the sig column is equal to "sig", then let's format it as bold using html. Otherwise, let's leave it alone." And indeed, we can see that the final result formats 3/4 rows.

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Creating a Table  

Thankfully, these can be achieved without considerable reshaping of the data, which is why we've started here, so we're almost done. We just need to get rid of some unnecessary columnns.  

```{.r .code-style}
tidy1 <- tidy1 %>%
  select(Trait, OR = estimate, CI)
```

Because we just have one target term and one outcome, we don't need those columns, so we're just keeping Trait, OR, which I renamed as such within in the select call, and CI. 

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
Now let's `kable`.  You've likely used the `kable()` function from the `knitr` before. It's a very useful and simple function in most occasions.  


```{.r .code-style}
kable(tidy1)
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> Trait </th>
   <th style="text-align:left;"> OR </th>
   <th style="text-align:left;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> OP </td>
   <td style="text-align:left;"> &lt;strong&gt;1.68&lt;/strong&gt; </td>
   <td style="text-align:left;"> &lt;strong&gt;[1.50, 1.88]&lt;/strong&gt; </td>
  </tr>
  <tr>
   <td style="text-align:left;"> DEP </td>
   <td style="text-align:left;"> &lt;strong&gt;1.15&lt;/strong&gt; </td>
   <td style="text-align:left;"> &lt;strong&gt;[1.07, 1.24]&lt;/strong&gt; </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NegAff </td>
   <td style="text-align:left;"> &lt;strong&gt;1.11&lt;/strong&gt; </td>
   <td style="text-align:left;"> &lt;strong&gt;[1.01, 1.22]&lt;/strong&gt; </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PA </td>
   <td style="text-align:left;"> &lt;strong&gt;1.97&lt;/strong&gt; </td>
   <td style="text-align:left;"> &lt;strong&gt;[1.77, 2.20]&lt;/strong&gt; </td>
  </tr>
  <tr>
   <td style="text-align:left;"> SE </td>
   <td style="text-align:left;"> 1.09 </td>
   <td style="text-align:left;"> [0.97, 1.22] </td>
  </tr>
</tbody>
</table>

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
It will automatically generate the html code needed to create a table. But if we look closely at the code, it gives us some gobbledigook where we inputted html, so we need a way around that. I'm also going to throw in `kable_styling(full_width = F)` from the `kableExtra` package to help out here. It's not doing much, but it will make the formatted table print in your Viewer.  


```{.r .code-style}
kable(tidy1, escape = F) %>%
  kable_styling(full_width = F)
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Trait </th>
   <th style="text-align:left;"> OR </th>
   <th style="text-align:left;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> OP </td>
   <td style="text-align:left;"> <strong>1.68</strong> </td>
   <td style="text-align:left;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> DEP </td>
   <td style="text-align:left;"> <strong>1.15</strong> </td>
   <td style="text-align:left;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NegAff </td>
   <td style="text-align:left;"> <strong>1.11</strong> </td>
   <td style="text-align:left;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PA </td>
   <td style="text-align:left;"> <strong>1.97</strong> </td>
   <td style="text-align:left;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> SE </td>
   <td style="text-align:left;"> 1.09 </td>
   <td style="text-align:left;"> [0.97, 1.22] </td>
  </tr>
</tbody>
</table>

Much better. But this still doesn't look like an APA table, so let's keep going. 

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
1. APA tables usually write out long names for our predictors, so let's change those first. I'm going to create a reference tibble and use `mapvalues()` from the `plyr` function for this.  

```{.r .code-style}
p_names <- tibble(
  old = c("NegAff", "PA", "SE", "OP", "DEP"),
  new = c("Negative Affect", "Positive Affect", "Self-Esteem", "Optimism", "Depression")
)

tidy1 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F) %>%
  kable_styling(full_width = F)
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Trait </th>
   <th style="text-align:left;"> OR </th>
   <th style="text-align:left;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Negative Affect </td>
   <td style="text-align:left;"> <strong>1.11</strong> </td>
   <td style="text-align:left;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Positive Affect </td>
   <td style="text-align:left;"> <strong>1.97</strong> </td>
   <td style="text-align:left;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Self-Esteem </td>
   <td style="text-align:left;"> 1.09 </td>
   <td style="text-align:left;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Optimism </td>
   <td style="text-align:left;"> <strong>1.68</strong> </td>
   <td style="text-align:left;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Depression </td>
   <td style="text-align:left;"> <strong>1.15</strong> </td>
   <td style="text-align:left;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
</table>

The combinatin of factor plus arrange here is super helpful for ordering your table.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
2. The alignment of the columns isn't quite right. Let's fix that. We'll change the trait to right justified and b and CI to centered.  

```{.r .code-style}
tidy1 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F,
        align = c("r", "c", "c")) %>%
  kable_styling(full_width = F)
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
</table>


### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
3. But we're still missing our span header. There's a great function in the `kableExtra` package for this `add_header_above`. This function takes a named vector as argument, where the elements of the vector refer to the number of columns the named element should span.  


```{.r .code-style}
tidy1 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F,
        align = c("r", "c", "c")) %>%
  kable_styling(full_width = F) %>%
  add_header_above(c(" " = 1, "Birth of a Child" = 2))
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
</table>

Note that what the `" " = 1` does is skip the Trait column. This is very useful because it let's us not have a span header over every column.  

### Basic Lessons: One DV/Outcome, Multiple Model Terms  
#### Kabling a Table  
4. APA style requires we note how we denote significance and have a title, so let's add a title and a note.  

```{.r .code-style}
tidy1 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F,
        align = c("r", "c", "c"),
        caption = "<strong>Table 1</strong><br><em>Estimated Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  add_header_above(c(" " = 1, "Birth of a Child" = 2)) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 1</strong><br><em>Estimated Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>

We did it!  

### A Quick Note: HTML v. LaTeX  
When creating tables, I prefer using HTML when I need the resulting tables to be in HTML and LaTeX when I can place the tables in a PDF. The syntax using `kable` and `kableExtra` is the same with the following exceptions: 

1. The `format` argument in `kable()` would need to be set as `format = "latex"`.  
2. The chunk option for a table to render would need to be set as `{r, results = 'asis'}`.  
3. Bolding would need to be done as `\\textbf{}`, rather than the `html` `<strong></strong>` tag.  
4. When using `collapse_rows()`, which we'll get to later, you'd want to set the `latex_hline` argument to `latex_hline = "none"`.  

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  

Often, our models are not quite so simple. So what happens when we mix multiple outcomes/DVs and multiple predictors / IVs? Thankfully, not much is different!

Below, we'll go through the steps. I'll skip over explaining ones that were explained in detail in the first example and focus on the new pieces.  

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Set Up Data  

```{.r .code-style}
gsoep_nested2 <- gsoep %>%
  group_by(Trait, Outcome) %>%
  nest() %>%
  ungroup()
gsoep_nested2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["data"],"name":[3],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"<tibble>"},{"1":"divorced","2":"OP","3":"<tibble>"},{"1":"married","2":"OP","3":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"<tibble>"},{"1":"chldbrth","2":"PA","3":"<tibble>"},{"1":"chldbrth","2":"SE","3":"<tibble>"},{"1":"divorced","2":"DEP","3":"<tibble>"},{"1":"divorced","2":"NegAff","3":"<tibble>"},{"1":"divorced","2":"PA","3":"<tibble>"},{"1":"divorced","2":"SE","3":"<tibble>"},{"1":"married","2":"DEP","3":"<tibble>"},{"1":"married","2":"NegAff","3":"<tibble>"},{"1":"married","2":"PA","3":"<tibble>"},{"1":"married","2":"SE","3":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

#### Run the models  

```{.r .code-style}
mod1_fun <- function(d){
  d$o_value <- factor(d$o_value)
  glm(o_value ~ p_value, data = d, family = binomial(link = "logit"))
}

gsoep_nested2 <- gsoep_nested2 %>%
  mutate(m = map(data, mod1_fun),
         tidy = map(m, ~tidy(., conf.int = T)))
gsoep_nested2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["data"],"name":[3],"type":["list"],"align":["right"]},{"label":["m"],"name":[4],"type":["list"],"align":["right"]},{"label":["tidy"],"name":[5],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"divorced","2":"OP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"married","2":"OP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"PA","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"chldbrth","2":"SE","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"divorced","2":"DEP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"divorced","2":"NegAff","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"divorced","2":"PA","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"divorced","2":"SE","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"married","2":"DEP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"married","2":"NegAff","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"married","2":"PA","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"married","2":"SE","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"<tibble>","4":"<S3: glm>","5":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

#### Create the Table  

```{.r .code-style}
tidy2 <- gsoep_nested2 %>%
  select(Outcome, Trait, tidy) %>%
  unnest(tidy)
tidy2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["term"],"name":[3],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[9],"type":["dbl"],"align":["right"]}],"data":[{"1":"chldbrth","2":"OP","3":"(Intercept)","4":"-4.301400973","5":"0.18588797","6":"-23.13974874","7":"1.843845e-118","8":"-4.67170199","9":"-3.94286980"},{"1":"chldbrth","2":"OP","3":"p_value","4":"0.517728030","5":"0.05816468","6":"8.90107321","7":"5.530926e-19","8":"0.40460371","9":"0.63265622"},{"1":"divorced","2":"OP","3":"(Intercept)","4":"-4.108157312","5":"0.25522514","6":"-16.09620967","7":"2.712184e-58","8":"-4.62049360","9":"-3.61947996"},{"1":"divorced","2":"OP","3":"p_value","4":"0.098117536","5":"0.08498366","6":"1.15454587","7":"2.482764e-01","8":"-0.06722175","9":"0.26608673"},{"1":"married","2":"OP","3":"(Intercept)","4":"-3.424477120","5":"0.15988828","6":"-21.41793657","7":"9.092844e-102","8":"-3.74223714","9":"-3.11534442"},{"1":"married","2":"OP","3":"p_value","4":"0.274753038","5":"0.05202187","6":"5.28149064","7":"1.281370e-07","8":"0.17333957","9":"0.37730577"},{"1":"mvInPrtnr","2":"OP","3":"(Intercept)","4":"-4.302485133","5":"0.19801816","6":"-21.72773059","7":"1.122124e-104","8":"-4.69738157","9":"-3.92096896"},{"1":"mvInPrtnr","2":"OP","3":"p_value","4":"0.458717666","5":"0.06251252","6":"7.33801228","7":"2.167893e-13","8":"0.33714457","9":"0.58225058"},{"1":"chldbrth","2":"DEP","3":"(Intercept)","4":"-3.475238148","5":"0.14605559","6":"-23.79394070","7":"3.858771e-125","8":"-3.76458852","9":"-3.19193801"},{"1":"chldbrth","2":"DEP","3":"p_value","4":"0.142506185","5":"0.03638957","6":"3.91612710","7":"8.998282e-05","8":"0.07164275","9":"0.21431481"},{"1":"chldbrth","2":"NegAff","3":"(Intercept)","4":"-3.581562333","5":"0.13150068","6":"-27.23607422","7":"2.429835e-163","8":"-3.84123426","9":"-3.32571677"},{"1":"chldbrth","2":"NegAff","3":"p_value","4":"0.106884261","5":"0.04710173","6":"2.26922155","7":"2.325486e-02","8":"0.01415774","9":"0.19881230"},{"1":"chldbrth","2":"PA","3":"(Intercept)","4":"-5.793319616","5":"0.21877964","6":"-26.48015935","7":"1.640624e-154","8":"-6.22838107","9":"-5.37070540"},{"1":"chldbrth","2":"PA","3":"p_value","4":"0.676777291","5":"0.05544973","6":"12.20523984","7":"2.914484e-34","8":"0.56895170","9":"0.78632650"},{"1":"chldbrth","2":"SE","3":"(Intercept)","4":"-4.108859728","5":"0.34183754","6":"-12.01991965","7":"2.792429e-33","8":"-4.80260759","9":"-3.46165510"},{"1":"chldbrth","2":"SE","3":"p_value","4":"0.085336183","5":"0.05832303","6":"1.46316441","7":"1.434224e-01","8":"-0.02642707","9":"0.20234736"},{"1":"divorced","2":"DEP","3":"(Intercept)","4":"-2.781660205","5":"0.18969716","6":"-14.66368916","7":"1.101234e-48","8":"-3.15908437","9":"-2.41531496"},{"1":"divorced","2":"DEP","3":"p_value","4":"-0.301788759","5":"0.05008814","6":"-6.02515416","7":"1.689485e-09","8":"-0.39925158","9":"-0.20286681"},{"1":"divorced","2":"NegAff","3":"(Intercept)","4":"-5.515456081","5":"0.20781359","6":"-26.54040169","7":"3.314658e-155","8":"-5.92777503","9":"-5.11301293"},{"1":"divorced","2":"NegAff","3":"p_value","4":"0.454944653","5":"0.06780118","6":"6.70998092","7":"1.946499e-11","8":"0.32136309","9":"0.58719994"},{"1":"divorced","2":"PA","3":"(Intercept)","4":"-3.691424074","5":"0.23992131","6":"-15.38597858","7":"2.032798e-53","8":"-4.17308336","9":"-3.23233858"},{"1":"divorced","2":"PA","3":"p_value","4":"-0.166866279","5":"0.06886530","6":"-2.42308213","7":"1.538945e-02","8":"-0.30045359","9":"-0.03044111"},{"1":"divorced","2":"SE","3":"(Intercept)","4":"-4.342776084","5":"0.47531517","6":"-9.13662429","7":"6.443216e-20","8":"-5.32519371","9":"-3.45909795"},{"1":"divorced","2":"SE","3":"p_value","4":"-0.056506891","5":"0.08389470","6":"-0.67354540","7":"5.006004e-01","8":"-0.21588797","9":"0.11344136"},{"1":"married","2":"DEP","3":"(Intercept)","4":"-2.731280949","5":"0.12750718","6":"-21.42060447","7":"8.586741e-102","8":"-2.98345944","9":"-2.48358807"},{"1":"married","2":"DEP","3":"p_value","4":"-0.006836054","5":"0.03234210","6":"-0.21136704","7":"8.326009e-01","8":"-0.06989988","9":"0.05689134"},{"1":"married","2":"NegAff","3":"(Intercept)","4":"-3.681409575","5":"0.11622222","6":"-31.67560896","7":"3.367890e-220","8":"-3.91077286","9":"-3.45515331"},{"1":"married","2":"NegAff","3":"p_value","4":"0.256448781","5":"0.04013494","6":"6.38966441","7":"1.662502e-10","8":"0.17757219","9":"0.33491493"},{"1":"married","2":"PA","3":"(Intercept)","4":"-4.755813059","5":"0.17710575","6":"-26.85295628","7":"7.790072e-159","8":"-5.10737886","9":"-4.41311434"},{"1":"married","2":"PA","3":"p_value","4":"0.486623335","5":"0.04614902","6":"10.54460905","7":"5.379613e-26","8":"0.39679649","9":"0.57770268"},{"1":"married","2":"SE","3":"(Intercept)","4":"-2.418268120","5":"0.23622121","6":"-10.23730286","7":"1.349459e-24","8":"-2.89306758","9":"-1.96651726"},{"1":"married","2":"SE","3":"p_value","4":"-0.150229939","5":"0.04265156","6":"-3.52226136","7":"4.278821e-04","8":"-0.23271571","9":"-0.06543186"},{"1":"mvInPrtnr","2":"DEP","3":"(Intercept)","4":"-2.678550743","5":"0.14022602","6":"-19.10166646","7":"2.445556e-81","8":"-2.95626192","9":"-2.40648549"},{"1":"mvInPrtnr","2":"DEP","3":"p_value","4":"-0.092832330","5":"0.03591710","6":"-2.58462768","7":"9.748420e-03","8":"-0.16283710","9":"-0.02201763"},{"1":"mvInPrtnr","2":"NegAff","3":"(Intercept)","4":"-4.278844009","5":"0.14878268","6":"-28.75901936","7":"6.986232e-182","8":"-4.57302536","9":"-3.98974075"},{"1":"mvInPrtnr","2":"NegAff","3":"p_value","4":"0.286719473","5":"0.05092592","6":"5.63012834","7":"1.800756e-08","8":"0.18650039","9":"0.38615471"},{"1":"mvInPrtnr","2":"PA","3":"(Intercept)","4":"-4.594037677","5":"0.21279911","6":"-21.58861364","7":"2.298033e-103","8":"-5.01819755","9":"-4.18397253"},{"1":"mvInPrtnr","2":"PA","3":"p_value","4":"0.304167971","5":"0.05699409","6":"5.33683322","7":"9.458397e-08","8":"0.19341659","9":"0.41684231"},{"1":"mvInPrtnr","2":"SE","3":"(Intercept)","4":"-3.896976194","5":"0.36679113","6":"-10.62451043","7":"2.292117e-26","8":"-4.64457503","9":"-3.20537299"},{"1":"mvInPrtnr","2":"SE","3":"p_value","4":"-0.002132562","5":"0.06378408","6":"-0.03343408","7":"9.733284e-01","8":"-0.12419908","9":"0.12605518"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
The basic steps from here are similar: filter target terms, index significance, exponentiate, format values, create CI's, bold significance, select needed columns.  

```{.r .code-style}
tidy2 <- tidy2 %>%
  filter(term == "p_value") %>%
  mutate(sig = ifelse(sign(conf.low) == sign(conf.high), "sig", "ns")) %>%
  mutate_at(vars(estimate, conf.low, conf.high), exp) %>%
  mutate_at(vars(estimate, conf.low, conf.high), ~sprintf("%.2f", .)) %>%
  mutate(CI = sprintf("[%s, %s]", conf.low, conf.high)) %>%
  mutate_at(vars(estimate, CI), ~ifelse(sig == "sig", sprintf("<strong>%s</strong>", .), .)) %>%
  select(Outcome, Trait, OR = estimate, CI)
tidy2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["OR"],"name":[3],"type":["chr"],"align":["left"]},{"label":["CI"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"<strong>1.68<\/strong>","4":"<strong>[1.50, 1.88]<\/strong>"},{"1":"divorced","2":"OP","3":"1.10","4":"[0.93, 1.30]"},{"1":"married","2":"OP","3":"<strong>1.32<\/strong>","4":"<strong>[1.19, 1.46]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"<strong>1.15<\/strong>","4":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"<strong>1.11<\/strong>","4":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"PA","3":"<strong>1.97<\/strong>","4":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth","2":"SE","3":"1.09","4":"[0.97, 1.22]"},{"1":"divorced","2":"DEP","3":"<strong>0.74<\/strong>","4":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced","2":"NegAff","3":"<strong>1.58<\/strong>","4":"<strong>[1.38, 1.80]<\/strong>"},{"1":"divorced","2":"PA","3":"<strong>0.85<\/strong>","4":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced","2":"SE","3":"0.95","4":"[0.81, 1.12]"},{"1":"married","2":"DEP","3":"0.99","4":"[0.93, 1.06]"},{"1":"married","2":"NegAff","3":"<strong>1.29<\/strong>","4":"<strong>[1.19, 1.40]<\/strong>"},{"1":"married","2":"PA","3":"<strong>1.63<\/strong>","4":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married","2":"SE","3":"<strong>0.86<\/strong>","4":"<strong>[0.79, 0.94]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"<strong>1.33<\/strong>","4":"<strong>[1.21, 1.47]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"<strong>1.36<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr","2":"SE","3":"1.00","4":"[0.88, 1.13]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Great, we're all set right? Not quite. If we want to do a span header, we need our data in shape for that. But  right now, our outcomes are rows, not columns. To get them as columns, we will need to: (1) `pivot_longer()` the OR's and CI's, (2) `unite()` the outcomes and type of estimate, (3) `pivot_wider()` these united terms, (4) reorder these columns as we want.  

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["OR"],"name":[3],"type":["chr"],"align":["left"]},{"label":["CI"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"<strong>1.68<\/strong>","4":"<strong>[1.50, 1.88]<\/strong>"},{"1":"divorced","2":"OP","3":"1.10","4":"[0.93, 1.30]"},{"1":"married","2":"OP","3":"<strong>1.32<\/strong>","4":"<strong>[1.19, 1.46]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"<strong>1.15<\/strong>","4":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"<strong>1.11<\/strong>","4":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"PA","3":"<strong>1.97<\/strong>","4":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth","2":"SE","3":"1.09","4":"[0.97, 1.22]"},{"1":"divorced","2":"DEP","3":"<strong>0.74<\/strong>","4":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced","2":"NegAff","3":"<strong>1.58<\/strong>","4":"<strong>[1.38, 1.80]<\/strong>"},{"1":"divorced","2":"PA","3":"<strong>0.85<\/strong>","4":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced","2":"SE","3":"0.95","4":"[0.81, 1.12]"},{"1":"married","2":"DEP","3":"0.99","4":"[0.93, 1.06]"},{"1":"married","2":"NegAff","3":"<strong>1.29<\/strong>","4":"<strong>[1.19, 1.40]<\/strong>"},{"1":"married","2":"PA","3":"<strong>1.63<\/strong>","4":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married","2":"SE","3":"<strong>0.86<\/strong>","4":"<strong>[0.79, 0.94]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"<strong>1.33<\/strong>","4":"<strong>[1.21, 1.47]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"<strong>1.36<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr","2":"SE","3":"1.00","4":"[0.88, 1.13]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Let's do each in turn:  
(1) Long format  

```{.r .code-style}
tidy2 <- tidy2 %>%
  pivot_longer(cols = c(OR, CI), names_to = "est", values_to = "value")
tidy2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["est"],"name":[3],"type":["chr"],"align":["left"]},{"label":["value"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"OP","3":"OR","4":"<strong>1.68<\/strong>"},{"1":"chldbrth","2":"OP","3":"CI","4":"<strong>[1.50, 1.88]<\/strong>"},{"1":"divorced","2":"OP","3":"OR","4":"1.10"},{"1":"divorced","2":"OP","3":"CI","4":"[0.93, 1.30]"},{"1":"married","2":"OP","3":"OR","4":"<strong>1.32<\/strong>"},{"1":"married","2":"OP","3":"CI","4":"<strong>[1.19, 1.46]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"OR","4":"<strong>1.58<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"CI","4":"<strong>[1.40, 1.79]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"OR","4":"<strong>1.15<\/strong>"},{"1":"chldbrth","2":"DEP","3":"CI","4":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"OR","4":"<strong>1.11<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"CI","4":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"PA","3":"OR","4":"<strong>1.97<\/strong>"},{"1":"chldbrth","2":"PA","3":"CI","4":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth","2":"SE","3":"OR","4":"1.09"},{"1":"chldbrth","2":"SE","3":"CI","4":"[0.97, 1.22]"},{"1":"divorced","2":"DEP","3":"OR","4":"<strong>0.74<\/strong>"},{"1":"divorced","2":"DEP","3":"CI","4":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced","2":"NegAff","3":"OR","4":"<strong>1.58<\/strong>"},{"1":"divorced","2":"NegAff","3":"CI","4":"<strong>[1.38, 1.80]<\/strong>"},{"1":"divorced","2":"PA","3":"OR","4":"<strong>0.85<\/strong>"},{"1":"divorced","2":"PA","3":"CI","4":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced","2":"SE","3":"OR","4":"0.95"},{"1":"divorced","2":"SE","3":"CI","4":"[0.81, 1.12]"},{"1":"married","2":"DEP","3":"OR","4":"0.99"},{"1":"married","2":"DEP","3":"CI","4":"[0.93, 1.06]"},{"1":"married","2":"NegAff","3":"OR","4":"<strong>1.29<\/strong>"},{"1":"married","2":"NegAff","3":"CI","4":"<strong>[1.19, 1.40]<\/strong>"},{"1":"married","2":"PA","3":"OR","4":"<strong>1.63<\/strong>"},{"1":"married","2":"PA","3":"CI","4":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married","2":"SE","3":"OR","4":"<strong>0.86<\/strong>"},{"1":"married","2":"SE","3":"CI","4":"<strong>[0.79, 0.94]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"OR","4":"<strong>0.91<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"CI","4":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"OR","4":"<strong>1.33<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"CI","4":"<strong>[1.21, 1.47]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"OR","4":"<strong>1.36<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"CI","4":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr","2":"SE","3":"OR","4":"1.00"},{"1":"mvInPrtnr","2":"SE","3":"CI","4":"[0.88, 1.13]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Let's do each in turn:  
(2) Unite!  

```{.r .code-style}
tidy2 <- tidy2 %>%
  unite(tmp, Outcome, est, sep = "_")
tidy2
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["tmp"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["value"],"name":[3],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth_OR","2":"OP","3":"<strong>1.68<\/strong>"},{"1":"chldbrth_CI","2":"OP","3":"<strong>[1.50, 1.88]<\/strong>"},{"1":"divorced_OR","2":"OP","3":"1.10"},{"1":"divorced_CI","2":"OP","3":"[0.93, 1.30]"},{"1":"married_OR","2":"OP","3":"<strong>1.32<\/strong>"},{"1":"married_CI","2":"OP","3":"<strong>[1.19, 1.46]<\/strong>"},{"1":"mvInPrtnr_OR","2":"OP","3":"<strong>1.58<\/strong>"},{"1":"mvInPrtnr_CI","2":"OP","3":"<strong>[1.40, 1.79]<\/strong>"},{"1":"chldbrth_OR","2":"DEP","3":"<strong>1.15<\/strong>"},{"1":"chldbrth_CI","2":"DEP","3":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth_OR","2":"NegAff","3":"<strong>1.11<\/strong>"},{"1":"chldbrth_CI","2":"NegAff","3":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth_OR","2":"PA","3":"<strong>1.97<\/strong>"},{"1":"chldbrth_CI","2":"PA","3":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth_OR","2":"SE","3":"1.09"},{"1":"chldbrth_CI","2":"SE","3":"[0.97, 1.22]"},{"1":"divorced_OR","2":"DEP","3":"<strong>0.74<\/strong>"},{"1":"divorced_CI","2":"DEP","3":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced_OR","2":"NegAff","3":"<strong>1.58<\/strong>"},{"1":"divorced_CI","2":"NegAff","3":"<strong>[1.38, 1.80]<\/strong>"},{"1":"divorced_OR","2":"PA","3":"<strong>0.85<\/strong>"},{"1":"divorced_CI","2":"PA","3":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced_OR","2":"SE","3":"0.95"},{"1":"divorced_CI","2":"SE","3":"[0.81, 1.12]"},{"1":"married_OR","2":"DEP","3":"0.99"},{"1":"married_CI","2":"DEP","3":"[0.93, 1.06]"},{"1":"married_OR","2":"NegAff","3":"<strong>1.29<\/strong>"},{"1":"married_CI","2":"NegAff","3":"<strong>[1.19, 1.40]<\/strong>"},{"1":"married_OR","2":"PA","3":"<strong>1.63<\/strong>"},{"1":"married_CI","2":"PA","3":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married_OR","2":"SE","3":"<strong>0.86<\/strong>"},{"1":"married_CI","2":"SE","3":"<strong>[0.79, 0.94]<\/strong>"},{"1":"mvInPrtnr_OR","2":"DEP","3":"<strong>0.91<\/strong>"},{"1":"mvInPrtnr_CI","2":"DEP","3":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr_OR","2":"NegAff","3":"<strong>1.33<\/strong>"},{"1":"mvInPrtnr_CI","2":"NegAff","3":"<strong>[1.21, 1.47]<\/strong>"},{"1":"mvInPrtnr_OR","2":"PA","3":"<strong>1.36<\/strong>"},{"1":"mvInPrtnr_CI","2":"PA","3":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr_OR","2":"SE","3":"1.00"},{"1":"mvInPrtnr_CI","2":"SE","3":"[0.88, 1.13]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Let's do each in turn:  
(3) Pivot wider  

```{.r .code-style}
tidy2 <- tidy2 %>%
  pivot_wider(names_from = "tmp", values_from = "value")
```

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Let's do each in turn:  
(4) Create the order of columns

```{.r .code-style}
O_names <- tibble(
  old = c("mvInPrtnr", "married", "divorced", "chldbrth"),
  new = c("Move in with Partner", "Married", "Divorced", "Birth of a Child")
)

levs <- paste(rep(O_names$old, each = 2), rep(c("OR","CI"), times = 4), sep = "_")
tidy2 <- tidy2 %>%
  select(Trait, all_of(levs))
```

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Now we're ready to `kable()`! This will proceed almost exactly as before. The only difference from the previous example is that we have multiple different columns we want to span. Thankfully, we know what these are because we carefully ordered them when we factored them.  

For our named vector, we'll take advantage of our `O_names` object to create the vector in advance:


```{.r .code-style}
heads <- c(1, rep(2, 4))
heads
```

```
## [1] 1 2 2 2 2
```

```{.r .code-style}
names(heads) <- c(" ", O_names$new)
heads
```

```
##                      Move in with Partner              Married 
##                    1                    2                    2 
##             Divorced     Birth of a Child 
##                    2                    2
```

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Starting where we left off in the previous example:  

```{.r .code-style}
tidy2 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F,
        align = c("r", rep("c", 8)),
        caption = "<strong>Table 2</strong><br><em>Estimated Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  add_header_above(heads) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 2</strong><br><em>Estimated Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Move in with Partner</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Married</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Divorced</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:center;"> mvInPrtnr_OR </th>
   <th style="text-align:center;"> mvInPrtnr_CI </th>
   <th style="text-align:center;"> married_OR </th>
   <th style="text-align:center;"> married_CI </th>
   <th style="text-align:center;"> divorced_OR </th>
   <th style="text-align:center;"> divorced_CI </th>
   <th style="text-align:center;"> chldbrth_OR </th>
   <th style="text-align:center;"> chldbrth_CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.29</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.40]</strong> </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.38, 1.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:center;"> <strong>1.36</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>

Ew, but those column names are terrible. Let's fix them using the `col.names` argument in `kable()`:  

### Intermediate Lessons: Multiple DVs/Outcomes, Multiple Model Terms  
#### Create the Table  
Ew, but those column names are terrible. Let's fix them using the `col.names` argument in `kable()`:  


```{.r .code-style}
tidy2 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new)) %>%
  arrange(Trait) %>%
  kable(., escape = F,
        align = c("r", rep("c", 8)),
        col.names = c("Trait", rep(c("OR", "CI"), times = 4)),
        caption = "<strong>Table 2</strong><br><em>Estimated Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  add_header_above(heads) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 2</strong><br><em>Estimated Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Move in with Partner</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Married</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Divorced</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.29</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.40]</strong> </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.38, 1.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:center;"> <strong>1.36</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>

Much better.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting  

Often, we want to do things like model comparison tests in which we add additional covariates and test if they significantly improve the model or see if they change the direction and magnitude of key terms. Generally, the resulting terms would then be placed in a table. 

Below, I'll cover two cases -- covariates and moderators. For the case of moderators, I'll follow up with simple effects (although these may be added at a later date). 

Specifically, we'll test this for age, gender, parental education, and self-rated health.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting  
To be able to do this, we'll need to set our data up just a little bit differently. First, this is because we need to center values for testing moderators for variables that aren't factors or don't have natural 0 points. Second, the covariates / moderators are currently in wide format, so we'll need to make them long for our purposes.  

Let's do that first. 


```{.r .code-style}
gsoep_long <- gsoep %>%
  select(SID, Outcome, o_value, Trait, p_value, age, gender, parEdu, SRhealth) %>%
  mutate_at(vars(age, SRhealth), ~as.numeric(scale(., center = T, scale = F))) %>%
  pivot_longer(
    cols = c(age, gender, parEdu, SRhealth), 
    values_to = "c_value", names_to = "Covariate"
    )
```

We'll need to worry about changing gender and parental education into factor variables later. I'm going to show you my favorite trick.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Set up Data  

This time, we need to add a new grouping variable -- namely, the target covariates.  

Only trick is we also need the combination with no covariate. There's lots of ways to add this on. I'll show you one way.  


```{.r .code-style}
gsoep_nested3 <- gsoep_long %>%
  full_join(gsoep %>% select(SID, Outcome, o_value, Trait, p_value) %>%
              mutate(Covariate = "none")) %>% 
  group_by(Trait, Outcome, Covariate) %>%
  nest() %>%
  ungroup() %>%
  arrange(Outcome, Trait, Covariate)
gsoep_nested3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["data"],"name":[4],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"none","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"none","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"none","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"none","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"none","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"age","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"gender","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"none","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"age","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"none","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"OP","3":"age","4":"<tibble>"},{"1":"divorced","2":"OP","3":"gender","4":"<tibble>"},{"1":"divorced","2":"OP","3":"none","4":"<tibble>"},{"1":"divorced","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"PA","3":"age","4":"<tibble>"},{"1":"divorced","2":"PA","3":"gender","4":"<tibble>"},{"1":"divorced","2":"PA","3":"none","4":"<tibble>"},{"1":"divorced","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"SE","3":"age","4":"<tibble>"},{"1":"divorced","2":"SE","3":"gender","4":"<tibble>"},{"1":"divorced","2":"SE","3":"none","4":"<tibble>"},{"1":"divorced","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"DEP","3":"age","4":"<tibble>"},{"1":"married","2":"DEP","3":"gender","4":"<tibble>"},{"1":"married","2":"DEP","3":"none","4":"<tibble>"},{"1":"married","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"married","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"NegAff","3":"age","4":"<tibble>"},{"1":"married","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"married","2":"NegAff","3":"none","4":"<tibble>"},{"1":"married","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"married","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"OP","3":"age","4":"<tibble>"},{"1":"married","2":"OP","3":"gender","4":"<tibble>"},{"1":"married","2":"OP","3":"none","4":"<tibble>"},{"1":"married","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"married","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"PA","3":"age","4":"<tibble>"},{"1":"married","2":"PA","3":"gender","4":"<tibble>"},{"1":"married","2":"PA","3":"none","4":"<tibble>"},{"1":"married","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"married","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"SE","3":"age","4":"<tibble>"},{"1":"married","2":"SE","3":"gender","4":"<tibble>"},{"1":"married","2":"SE","3":"none","4":"<tibble>"},{"1":"married","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"married","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"none","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"none","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"none","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"none","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"none","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Run the models  

```{.r .code-style}
factor_fun <- function(x){if(is.numeric(x)){diff(range(x, na.rm = T)) %in% 1:2 & length(unique(x)) <= 4} else{F}}

mod3_fun <- function(d, cov){
  d$o_value <- factor(d$o_value)
  d <- d %>% mutate_if(factor_fun, factor)
  if(cov == "none"){f <- formula(o_value ~ p_value)} else{f <- formula(o_value ~ p_value + c_value)}
  glm(f, data = d, family = binomial(link = "logit"))
}

gsoep_nested3 <- gsoep_nested3 %>%
  mutate(m = map2(data, Covariate, mod3_fun),
         tidy = map(m, ~tidy(., conf.int = T)))
gsoep_nested3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["data"],"name":[4],"type":["list"],"align":["right"]},{"label":["m"],"name":[5],"type":["list"],"align":["right"]},{"label":["tidy"],"name":[6],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"none","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Looking specifically at the `tidy` column, notice that there are different numbers of rows. This is good! We should see 3 rows for age, gender, and SRhealth because we have one new term -- a continuous covariate of two level binary covariate. We should see 4 rows for parEdu because we have two new terms -- for a three level categorical covariate. Finally, when there is no covariate, we should just have 2 rows like in the previous example. 


### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Create the Table  

```{.r .code-style}
tidy3 <- gsoep_nested3 %>%
  select(Outcome, Trait, Covariate, tidy) %>%
  unnest(tidy)
tidy3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["term"],"name":[4],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[9],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[10],"type":["dbl"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"(Intercept)","5":"-4.055454730","6":"0.161333235","7":"-25.13713140","8":"1.954094e-139","9":"-4.374922563","10":"-3.742437288"},{"1":"chldbrth","2":"DEP","3":"age","4":"p_value","5":"0.146342373","6":"0.039675882","7":"3.68844659","8":"2.256274e-04","9":"0.069049012","10":"0.224594739"},{"1":"chldbrth","2":"DEP","3":"age","4":"c_value","5":"-0.066315784","6":"0.001778656","7":"-37.28421947","8":"2.957304e-304","9":"-0.069827550","10":"-0.062854610"},{"1":"chldbrth","2":"DEP","3":"gender","4":"(Intercept)","5":"-3.567227860","6":"0.153172949","7":"-23.28888930","8":"5.745551e-120","9":"-3.870436358","10":"-3.269884573"},{"1":"chldbrth","2":"DEP","3":"gender","4":"p_value","5":"0.150779338","6":"0.036662397","7":"4.11264269","8":"3.911555e-05","9":"0.079380968","10":"0.223123009"},{"1":"chldbrth","2":"DEP","3":"gender","4":"c_value1","5":"0.113176758","6":"0.054135041","7":"2.09063772","8":"3.656055e-02","9":"0.007185869","10":"0.219445599"},{"1":"chldbrth","2":"DEP","3":"none","4":"(Intercept)","5":"-3.475238148","6":"0.146055594","7":"-23.79394070","8":"3.858771e-125","9":"-3.764588515","10":"-3.191938009"},{"1":"chldbrth","2":"DEP","3":"none","4":"p_value","5":"0.142506185","6":"0.036389571","7":"3.91612710","8":"8.998282e-05","9":"0.071642752","10":"0.214314809"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-3.486026507","6":"0.150064667","7":"-23.23016188","8":"2.257658e-119","9":"-3.783374985","10":"-3.194991717"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"p_value","5":"0.117451939","6":"0.037402031","7":"3.14025563","8":"1.688005e-03","9":"0.044613422","10":"0.191258813"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"c_value1","5":"0.579249968","6":"0.067017917","7":"8.64321051","8":"5.465509e-18","9":"0.446615254","10":"0.709414336"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"c_value2","5":"0.864477764","6":"0.105138017","7":"8.22231373","8":"1.996142e-16","9":"0.653710837","10":"1.066229585"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-2.098995232","6":"0.156910804","7":"-13.37699623","8":"8.241105e-41","9":"-2.409178996","10":"-1.794019600"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"p_value","5":"-0.267782656","6":"0.040764871","7":"-6.56895630","8":"5.066915e-11","9":"-0.347365349","10":"-0.187551364"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"c_value","5":"1.014550647","6":"0.043120411","7":"23.52831556","8":"2.093360e-122","9":"0.930418238","10":"1.099457053"},{"1":"chldbrth","2":"NegAff","3":"age","4":"(Intercept)","5":"-3.856362973","6":"0.143950363","7":"-26.78953279","8":"4.279003e-158","9":"-4.140952202","10":"-3.576594195"},{"1":"chldbrth","2":"NegAff","3":"age","4":"p_value","5":"-0.011764384","6":"0.049688167","7":"-0.23676429","8":"8.128397e-01","9":"-0.109606441","10":"0.085196782"},{"1":"chldbrth","2":"NegAff","3":"age","4":"c_value","5":"-0.063185870","6":"0.002623346","7":"-24.08598438","8":"3.505847e-128","9":"-0.068383073","10":"-0.058096926"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"(Intercept)","5":"-3.597032414","6":"0.133866224","7":"-26.87035096","8":"4.879069e-159","9":"-3.861398543","10":"-3.336600633"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"p_value","5":"0.102358231","6":"0.047743813","7":"2.14390564","8":"3.204045e-02","9":"0.008392494","10":"0.195562369"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"c_value1","5":"0.052101923","6":"0.084365394","7":"0.61757458","8":"5.368558e-01","9":"-0.113005375","10":"0.217859528"},{"1":"chldbrth","2":"NegAff","3":"none","4":"(Intercept)","5":"-3.581562333","6":"0.131500682","7":"-27.23607422","8":"2.429835e-163","9":"-3.841234261","10":"-3.325716773"},{"1":"chldbrth","2":"NegAff","3":"none","4":"p_value","5":"0.106884261","6":"0.047101730","7":"2.26922155","8":"2.325486e-02","9":"0.014157741","10":"0.198812299"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-3.712284189","6":"0.136510000","7":"-27.19422898","8":"7.600252e-163","9":"-3.981909980","10":"-3.446750175"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"p_value","5":"0.109228560","6":"0.047891778","7":"2.28073723","8":"2.256400e-02","9":"0.014949744","10":"0.202703646"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"c_value1","5":"0.635593180","6":"0.098927474","7":"6.42483989","8":"1.320084e-10","9":"0.438935253","10":"0.827012161"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"c_value2","5":"0.727463411","6":"0.169304017","7":"4.29678766","8":"1.732910e-05","9":"0.381650945","10":"1.047000795"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.394429244","6":"0.146998698","7":"-29.89434129","8":"2.330962e-196","9":"-4.684957836","10":"-4.108670803"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"p_value","5":"0.326841902","6":"0.049642406","7":"6.58392544","8":"4.581870e-11","9":"0.229235081","10":"0.423858754"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"c_value","5":"1.103872391","6":"0.066440104","7":"16.61454943","8":"5.468415e-62","9":"0.974637880","10":"1.235104551"},{"1":"chldbrth","2":"OP","3":"age","4":"(Intercept)","5":"-4.236292541","6":"0.195668282","7":"-21.65037935","8":"6.028211e-104","9":"-4.626219670","10":"-3.858957649"},{"1":"chldbrth","2":"OP","3":"age","4":"p_value","5":"0.285816854","6":"0.061505237","7":"4.64703280","8":"3.367436e-06","9":"0.166064246","10":"0.407238366"},{"1":"chldbrth","2":"OP","3":"age","4":"c_value","5":"-0.063012834","6":"0.003028372","7":"-20.80749650","8":"3.701775e-96","9":"-0.069023510","10":"-0.057148031"},{"1":"chldbrth","2":"OP","3":"gender","4":"(Intercept)","5":"-4.368900098","6":"0.193446995","7":"-22.58448160","8":"6.157714e-113","9":"-4.753981534","10":"-3.995528893"},{"1":"chldbrth","2":"OP","3":"gender","4":"p_value","5":"0.519266167","6":"0.058224688","7":"8.91831600","8":"4.734319e-19","9":"0.406024743","10":"0.634312427"},{"1":"chldbrth","2":"OP","3":"gender","4":"c_value1","5":"0.118867729","6":"0.089268836","7":"1.33157028","8":"1.830014e-01","9":"-0.055778548","10":"0.294331835"},{"1":"chldbrth","2":"OP","3":"none","4":"(Intercept)","5":"-4.301400973","6":"0.185887972","7":"-23.13974874","8":"1.843845e-118","9":"-4.671701991","10":"-3.942869800"},{"1":"chldbrth","2":"OP","3":"none","4":"p_value","5":"0.517728030","6":"0.058164675","7":"8.90107321","8":"5.530926e-19","9":"0.404603708","10":"0.632656221"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.320286817","6":"0.191668485","7":"-22.54041298","8":"1.667561e-112","9":"-4.702203647","10":"-3.950704305"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"p_value","5":"0.486007099","6":"0.060211205","7":"8.07170519","8":"6.932310e-16","9":"0.368868266","10":"0.604946476"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"c_value1","5":"0.501083912","6":"0.111701274","7":"4.48592836","8":"7.259713e-06","9":"0.278676515","10":"0.716911772"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"c_value2","5":"0.876036990","6":"0.161099004","7":"5.43787960","8":"5.391839e-08","9":"0.550497443","10":"1.183216266"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.768486703","6":"0.189736613","7":"-19.86167378","8":"8.735046e-88","9":"-4.146169557","10":"-3.402236614"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"p_value","5":"0.267766744","6":"0.061709494","7":"4.33914987","8":"1.430350e-05","9":"0.147561344","10":"0.389509397"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"c_value","5":"0.879854835","6":"0.071680488","7":"12.27467696","8":"1.238853e-34","9":"0.740585088","10":"1.021602883"},{"1":"chldbrth","2":"PA","3":"age","4":"(Intercept)","5":"-5.595834211","6":"0.222877067","7":"-25.10726783","8":"4.142710e-139","9":"-6.039382798","10":"-5.165654605"},{"1":"chldbrth","2":"PA","3":"age","4":"p_value","5":"0.473716721","6":"0.056888241","7":"8.32714654","8":"8.281421e-17","9":"0.363127220","10":"0.586140206"},{"1":"chldbrth","2":"PA","3":"age","4":"c_value","5":"-0.059561860","6":"0.002647587","7":"-22.49665529","8":"4.475672e-112","9":"-0.064807304","10":"-0.054426241"},{"1":"chldbrth","2":"PA","3":"gender","4":"(Intercept)","5":"-5.826936681","6":"0.222317149","7":"-26.21001898","8":"2.042998e-151","9":"-6.268740467","10":"-5.397187747"},{"1":"chldbrth","2":"PA","3":"gender","4":"p_value","5":"0.675278661","6":"0.055426678","7":"12.18327854","8":"3.816265e-34","9":"0.567502383","10":"0.784786985"},{"1":"chldbrth","2":"PA","3":"gender","4":"c_value1","5":"0.074281298","6":"0.083821313","7":"0.88618629","8":"3.755172e-01","9":"-0.089736015","10":"0.239010323"},{"1":"chldbrth","2":"PA","3":"none","4":"(Intercept)","5":"-5.793319616","6":"0.218779636","7":"-26.48015935","8":"1.640624e-154","9":"-6.228381073","10":"-5.370705397"},{"1":"chldbrth","2":"PA","3":"none","4":"p_value","5":"0.676777291","6":"0.055449733","7":"12.20523984","8":"2.914484e-34","9":"0.568951699","10":"0.786326495"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"(Intercept)","5":"-5.713115281","6":"0.220732571","7":"-25.88252044","8":"1.047839e-147","9":"-6.152191740","10":"-5.286867053"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"p_value","5":"0.627851991","6":"0.056194019","7":"11.17293272","8":"5.532353e-29","9":"0.518591716","10":"0.738881801"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"c_value1","5":"0.530610429","6":"0.099812464","7":"5.31607383","8":"1.060301e-07","9":"0.332241234","10":"0.723787636"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"c_value2","5":"0.680887390","6":"0.170443846","7":"3.99479011","8":"6.475159e-05","9":"0.332998750","10":"1.002818702"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-5.252531900","6":"0.224356372","7":"-23.41155656","8":"3.259157e-121","9":"-5.698517433","10":"-4.819047423"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"p_value","5":"0.478851292","6":"0.058272701","7":"8.21742052","8":"2.079270e-16","9":"0.365440836","10":"0.593866315"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"c_value","5":"0.851663549","6":"0.066952695","7":"12.72037749","8":"4.556536e-37","9":"0.721390761","10":"0.983857691"},{"1":"chldbrth","2":"SE","3":"age","4":"(Intercept)","5":"-5.187611675","6":"0.371960252","7":"-13.94668286","8":"3.295813e-44","9":"-5.940434718","10":"-4.481371537"},{"1":"chldbrth","2":"SE","3":"age","4":"p_value","5":"0.165195084","6":"0.061043342","7":"2.70619329","8":"6.805942e-03","9":"0.048146419","10":"0.287605945"},{"1":"chldbrth","2":"SE","3":"age","4":"c_value","5":"-0.064882405","6":"0.004397430","7":"-14.75461945","8":"2.873074e-49","9":"-0.073669987","10":"-0.056417164"},{"1":"chldbrth","2":"SE","3":"gender","4":"(Intercept)","5":"-4.253805004","6":"0.354754759","7":"-11.99083280","8":"3.968946e-33","9":"-4.971562141","10":"-3.579982377"},{"1":"chldbrth","2":"SE","3":"gender","4":"p_value","5":"0.089430255","6":"0.058293584","7":"1.53413548","8":"1.249963e-01","9":"-0.022299055","10":"0.206362610"},{"1":"chldbrth","2":"SE","3":"gender","4":"c_value1","5":"0.223188529","6":"0.142839444","7":"1.56251329","8":"1.181671e-01","9":"-0.055395239","10":"0.505386441"},{"1":"chldbrth","2":"SE","3":"none","4":"(Intercept)","5":"-4.108859728","6":"0.341837537","7":"-12.01991965","8":"2.792429e-33","9":"-4.802607591","10":"-3.461655105"},{"1":"chldbrth","2":"SE","3":"none","4":"p_value","5":"0.085336183","6":"0.058323031","7":"1.46316441","8":"1.434224e-01","9":"-0.026427068","10":"0.202347362"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.242730922","6":"0.347848957","7":"-12.19704941","8":"3.222921e-34","9":"-4.948423442","10":"-3.583845246"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"p_value","5":"0.078124988","6":"0.058711125","7":"1.33066752","8":"1.832984e-01","9":"-0.034366399","10":"0.195940614"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"c_value1","5":"0.860196614","6":"0.157479551","7":"5.46227500","8":"4.700711e-08","9":"0.545753109","10":"1.164216228"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"c_value2","5":"0.492632504","6":"0.333914665","7":"1.47532455","8":"1.401253e-01","9":"-0.226453661","10":"1.096374252"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-3.715844255","6":"0.347877326","7":"-10.68147871","8":"1.242761e-26","9":"-4.421436485","10":"-3.056749330"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"p_value","5":"-0.022132974","6":"0.060442698","7":"-0.36618110","8":"7.142299e-01","9":"-0.138191416","10":"0.098904332"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"c_value","5":"1.009939204","6":"0.111480325","7":"9.05934926","8":"1.312311e-19","9":"0.794097005","10":"1.231185624"},{"1":"divorced","2":"DEP","3":"age","4":"(Intercept)","5":"-2.813256479","6":"0.194755786","7":"-14.44504697","8":"2.693736e-47","9":"-3.200690078","10":"-2.437086693"},{"1":"divorced","2":"DEP","3":"age","4":"p_value","5":"-0.320272679","6":"0.051423708","7":"-6.22811326","8":"4.720858e-10","9":"-0.420372880","10":"-0.218749420"},{"1":"divorced","2":"DEP","3":"age","4":"c_value","5":"-0.023429303","6":"0.002303587","7":"-10.17079286","8":"2.677205e-24","9":"-0.027966718","10":"-0.018935129"},{"1":"divorced","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.762829856","6":"0.201079975","7":"-13.73995527","8":"5.851193e-43","9":"-3.162393379","10":"-2.374022921"},{"1":"divorced","2":"DEP","3":"gender","4":"p_value","5":"-0.303847828","6":"0.050465166","7":"-6.02094181","8":"1.734051e-09","9":"-0.402047126","10":"-0.204185501"},{"1":"divorced","2":"DEP","3":"gender","4":"c_value1","5":"-0.020118962","6":"0.082732124","7":"-0.24318198","8":"8.078644e-01","9":"-0.182106310","10":"0.142358901"},{"1":"divorced","2":"DEP","3":"none","4":"(Intercept)","5":"-2.781660205","6":"0.189697161","7":"-14.66368916","8":"1.101234e-48","9":"-3.159084369","10":"-2.415314965"},{"1":"divorced","2":"DEP","3":"none","4":"p_value","5":"-0.301788759","6":"0.050088139","7":"-6.02515416","8":"1.689485e-09","9":"-0.399251577","10":"-0.202866815"},{"1":"divorced","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.888470122","6":"0.199707069","7":"-14.46353467","8":"2.059443e-47","9":"-3.286041880","10":"-2.503011915"},{"1":"divorced","2":"DEP","3":"parEdu","4":"p_value","5":"-0.277057133","6":"0.052540519","7":"-5.27320891","8":"1.340588e-07","9":"-0.379259521","10":"-0.173256249"},{"1":"divorced","2":"DEP","3":"parEdu","4":"c_value1","5":"0.089670095","6":"0.118186838","7":"0.75871473","8":"4.480232e-01","9":"-0.147947315","10":"0.315905675"},{"1":"divorced","2":"DEP","3":"parEdu","4":"c_value2","5":"0.338045615","6":"0.180481072","7":"1.87302531","8":"6.106489e-02","9":"-0.033734574","10":"0.675951352"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-2.197110205","6":"0.210169394","7":"-10.45399695","8":"1.404773e-25","9":"-2.613987375","10":"-1.789994733"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"p_value","5":"-0.463572853","6":"0.056598766","7":"-8.19051162","8":"2.601181e-16","9":"-0.574004153","10":"-0.352107299"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"c_value","5":"0.369332098","6":"0.059809191","7":"6.17517295","8":"6.609108e-10","9":"0.252673458","10":"0.487132523"},{"1":"divorced","2":"NegAff","3":"age","4":"(Intercept)","5":"-5.503333859","6":"0.209882807","7":"-26.22098464","8":"1.531934e-151","9":"-5.919760544","10":"-5.096872260"},{"1":"divorced","2":"NegAff","3":"age","4":"p_value","5":"0.414658749","6":"0.068719292","7":"6.03409521","8":"1.598558e-09","9":"0.279237764","10":"0.548675282"},{"1":"divorced","2":"NegAff","3":"age","4":"c_value","5":"-0.019898711","6":"0.003453306","7":"-5.76222107","8":"8.301416e-09","9":"-0.026712904","10":"-0.013170036"},{"1":"divorced","2":"NegAff","3":"gender","4":"(Intercept)","5":"-5.492492768","6":"0.210345412","7":"-26.11177835","8":"2.679528e-150","9":"-5.909890811","10":"-5.085180144"},{"1":"divorced","2":"NegAff","3":"gender","4":"p_value","5":"0.463408820","6":"0.068840861","7":"6.73159539","8":"1.678128e-11","9":"0.327798681","10":"0.597704067"},{"1":"divorced","2":"NegAff","3":"gender","4":"c_value1","5":"-0.086275812","6":"0.126405425","7":"-0.68253251","8":"4.949023e-01","9":"-0.333647114","10":"0.162412770"},{"1":"divorced","2":"NegAff","3":"none","4":"(Intercept)","5":"-5.515456081","6":"0.207813587","7":"-26.54040169","8":"3.314658e-155","9":"-5.927775034","10":"-5.113012928"},{"1":"divorced","2":"NegAff","3":"none","4":"p_value","5":"0.454944653","6":"0.067801184","7":"6.70998092","8":"1.946499e-11","9":"0.321363088","10":"0.587199940"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-5.566335515","6":"0.215819520","7":"-25.79162221","8":"1.100975e-146","9":"-5.994683503","10":"-5.148523441"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"p_value","5":"0.473860015","6":"0.069382095","7":"6.82971613","8":"8.508282e-12","9":"0.337183635","10":"0.609225067"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"c_value1","5":"-0.011988202","6":"0.176638659","7":"-0.06786851","8":"9.458903e-01","9":"-0.372168018","10":"0.322172510"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"c_value2","5":"0.309329640","6":"0.263134785","7":"1.17555586","8":"2.397724e-01","9":"-0.246002753","10":"0.792328211"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-5.741583193","6":"0.218351415","7":"-26.29514992","8":"2.178992e-152","9":"-6.174815205","10":"-5.318767553"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"p_value","5":"0.529216901","6":"0.070627078","7":"7.49311618","8":"6.725731e-14","9":"0.390175719","10":"0.667084796"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"c_value","5":"0.337283904","6":"0.088451471","7":"3.81320853","8":"1.371743e-04","9":"0.165566237","10":"0.512338890"},{"1":"divorced","2":"OP","3":"age","4":"(Intercept)","5":"-3.965328117","6":"0.256772002","7":"-15.44299256","8":"8.410565e-54","9":"-4.480697130","10":"-3.473682413"},{"1":"divorced","2":"OP","3":"age","4":"p_value","5":"0.003004414","6":"0.087049395","7":"0.03451390","8":"9.724674e-01","9":"-0.166454902","10":"0.174925747"},{"1":"divorced","2":"OP","3":"age","4":"c_value","5":"-0.022459404","6":"0.003983067","7":"-5.63872082","8":"1.713180e-08","9":"-0.030337356","10":"-0.014716237"},{"1":"divorced","2":"OP","3":"gender","4":"(Intercept)","5":"-4.120005705","6":"0.266125546","7":"-15.48143639","8":"4.630181e-54","9":"-4.653707314","10":"-3.610007346"},{"1":"divorced","2":"OP","3":"gender","4":"p_value","5":"0.097933903","6":"0.085007021","7":"1.15206840","8":"2.492930e-01","9":"-0.067450056","10":"0.265950269"},{"1":"divorced","2":"OP","3":"gender","4":"c_value1","5":"0.024317770","6":"0.137777564","7":"0.17650022","8":"8.599010e-01","9":"-0.245470909","10":"0.295404821"},{"1":"divorced","2":"OP","3":"none","4":"(Intercept)","5":"-4.108157312","6":"0.255225137","7":"-16.09620967","8":"2.712184e-58","9":"-4.620493602","10":"-3.619479961"},{"1":"divorced","2":"OP","3":"none","4":"p_value","5":"0.098117536","6":"0.084983662","7":"1.15454587","8":"2.482764e-01","9":"-0.067221751","10":"0.266086729"},{"1":"divorced","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.038070774","6":"0.264989291","7":"-15.23861876","8":"1.959695e-52","9":"-4.570437355","10":"-3.531095904"},{"1":"divorced","2":"OP","3":"parEdu","4":"p_value","5":"0.066305428","6":"0.089288493","7":"0.74259768","8":"4.577253e-01","9":"-0.107477311","10":"0.242724877"},{"1":"divorced","2":"OP","3":"parEdu","4":"c_value1","5":"-0.029639696","6":"0.205250691","7":"-0.14440729","8":"8.851789e-01","9":"-0.450934785","10":"0.356640269"},{"1":"divorced","2":"OP","3":"parEdu","4":"c_value2","5":"0.269471387","6":"0.293625509","7":"0.91773834","8":"3.587559e-01","9":"-0.354739278","10":"0.805661331"},{"1":"divorced","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.957209634","6":"0.266334271","7":"-14.85805643","8":"6.168903e-50","9":"-4.490873279","10":"-3.446365897"},{"1":"divorced","2":"OP","3":"SRhealth","4":"p_value","5":"0.039265261","6":"0.090540892","7":"0.43367433","8":"6.645249e-01","9":"-0.137179375","10":"0.217886389"},{"1":"divorced","2":"OP","3":"SRhealth","4":"c_value","5":"0.187546923","6":"0.099309659","7":"1.88850637","8":"5.895800e-02","9":"-0.005046339","10":"0.384344754"},{"1":"divorced","2":"PA","3":"age","4":"(Intercept)","5":"-3.506027868","6":"0.239538097","7":"-14.63661906","8":"1.640241e-48","9":"-3.986871901","10":"-3.047595741"},{"1":"divorced","2":"PA","3":"age","4":"p_value","5":"-0.255364649","6":"0.069783605","7":"-3.65937884","8":"2.528273e-04","9":"-0.390801984","10":"-0.117179638"},{"1":"divorced","2":"PA","3":"age","4":"c_value","5":"-0.024015804","6":"0.003476141","7":"-6.90875450","8":"4.889272e-12","9":"-0.030875850","10":"-0.017243781"},{"1":"divorced","2":"PA","3":"gender","4":"(Intercept)","5":"-3.718675586","6":"0.248896989","7":"-14.94062103","8":"1.792965e-50","9":"-4.217359221","10":"-3.241445483"},{"1":"divorced","2":"PA","3":"gender","4":"p_value","5":"-0.167335866","6":"0.068836406","7":"-2.43092102","8":"1.506050e-02","9":"-0.300869478","10":"-0.030969268"},{"1":"divorced","2":"PA","3":"gender","4":"c_value1","5":"0.055097509","6":"0.124328469","7":"0.44316084","8":"6.576494e-01","9":"-0.188141992","10":"0.299798886"},{"1":"divorced","2":"PA","3":"none","4":"(Intercept)","5":"-3.691424074","6":"0.239921306","7":"-15.38597858","8":"2.032798e-53","9":"-4.173083358","10":"-3.232338577"},{"1":"divorced","2":"PA","3":"none","4":"p_value","5":"-0.166866279","6":"0.068865301","7":"-2.42308213","8":"1.538945e-02","9":"-0.300453593","10":"-0.030441105"},{"1":"divorced","2":"PA","3":"parEdu","4":"(Intercept)","5":"-3.772606808","6":"0.249234278","7":"-15.13678953","8":"9.262638e-52","9":"-4.273123650","10":"-3.295857523"},{"1":"divorced","2":"PA","3":"parEdu","4":"p_value","5":"-0.143476151","6":"0.071200388","7":"-2.01510349","8":"4.389381e-02","9":"-0.281545427","10":"-0.002378320"},{"1":"divorced","2":"PA","3":"parEdu","4":"c_value1","5":"0.011135875","6":"0.177063172","7":"0.06289210","8":"9.498524e-01","9":"-0.349804423","10":"0.346196225"},{"1":"divorced","2":"PA","3":"parEdu","4":"c_value2","5":"0.368782151","6":"0.262612955","7":"1.40428012","8":"1.602355e-01","9":"-0.185644312","10":"0.850655245"},{"1":"divorced","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-3.461287030","6":"0.250072488","7":"-13.84113487","8":"1.439164e-43","9":"-3.962684017","10":"-2.982197503"},{"1":"divorced","2":"PA","3":"SRhealth","4":"p_value","5":"-0.238949824","6":"0.072877988","7":"-3.27876535","8":"1.042623e-03","9":"-0.380472035","10":"-0.094749550"},{"1":"divorced","2":"PA","3":"SRhealth","4":"c_value","5":"0.266402895","6":"0.090118868","7":"2.95612783","8":"3.115279e-03","9":"0.091350806","10":"0.444644813"},{"1":"divorced","2":"SE","3":"age","4":"(Intercept)","5":"-4.573369661","6":"0.486848572","7":"-9.39382372","8":"5.786070e-21","9":"-5.578347472","10":"-3.667006955"},{"1":"divorced","2":"SE","3":"age","4":"p_value","5":"-0.032295303","6":"0.084890199","7":"-0.38043619","8":"7.036217e-01","9":"-0.193589137","10":"0.139660515"},{"1":"divorced","2":"SE","3":"age","4":"c_value","5":"-0.020587114","6":"0.005933660","7":"-3.46954702","8":"5.213368e-04","9":"-0.032353741","10":"-0.009057889"},{"1":"divorced","2":"SE","3":"gender","4":"(Intercept)","5":"-4.337338999","6":"0.496821714","7":"-8.73017197","8":"2.542857e-18","9":"-5.359129857","10":"-3.408917604"},{"1":"divorced","2":"SE","3":"gender","4":"p_value","5":"-0.056701964","6":"0.084059335","7":"-0.67454690","8":"4.999637e-01","9":"-0.216441756","10":"0.113528951"},{"1":"divorced","2":"SE","3":"gender","4":"c_value1","5":"-0.007879168","6":"0.219948613","7":"-0.03582277","8":"9.714237e-01","9":"-0.439160820","10":"0.426263595"},{"1":"divorced","2":"SE","3":"none","4":"(Intercept)","5":"-4.342776084","6":"0.475315165","7":"-9.13662429","8":"6.443216e-20","9":"-5.325193709","10":"-3.459097955"},{"1":"divorced","2":"SE","3":"none","4":"p_value","5":"-0.056506891","6":"0.083894703","7":"-0.67354540","8":"5.006004e-01","9":"-0.215887970","10":"0.113441359"},{"1":"divorced","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.449638146","6":"0.496749534","7":"-8.95750845","8":"3.320973e-19","9":"-5.476965212","10":"-3.526651047"},{"1":"divorced","2":"SE","3":"parEdu","4":"p_value","5":"-0.037062920","6":"0.086579515","7":"-0.42807955","8":"6.685932e-01","9":"-0.201324285","10":"0.138554210"},{"1":"divorced","2":"SE","3":"parEdu","4":"c_value1","5":"0.135883424","6":"0.288845711","7":"0.47043601","8":"6.380435e-01","9":"-0.466876354","10":"0.673757764"},{"1":"divorced","2":"SE","3":"parEdu","4":"c_value2","5":"-0.135826754","6":"0.593434900","7":"-0.22888232","8":"8.189604e-01","9":"-1.548423997","10":"0.861266082"},{"1":"divorced","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-4.193588831","6":"0.482763180","7":"-8.68663768","8":"3.733244e-18","9":"-5.189668320","10":"-3.294414945"},{"1":"divorced","2":"SE","3":"SRhealth","4":"p_value","5":"-0.087343225","6":"0.086020897","7":"-1.01537217","8":"3.099285e-01","9":"-0.251194755","10":"0.086462101"},{"1":"divorced","2":"SE","3":"SRhealth","4":"c_value","5":"0.276211388","6":"0.158204322","7":"1.74591556","8":"8.082563e-02","9":"-0.028335312","10":"0.592014765"},{"1":"married","2":"DEP","3":"age","4":"(Intercept)","5":"-3.140256857","6":"0.138521222","7":"-22.66986114","8":"8.887399e-114","9":"-3.414120300","10":"-2.871067552"},{"1":"married","2":"DEP","3":"age","4":"p_value","5":"-0.012774999","6":"0.034798730","7":"-0.36711107","8":"7.135362e-01","9":"-0.080666378","10":"0.055758121"},{"1":"married","2":"DEP","3":"age","4":"c_value","5":"-0.055873323","6":"0.001586824","7":"-35.21078265","8":"1.367527e-271","9":"-0.059002856","10":"-0.052781946"},{"1":"married","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.731072005","6":"0.133818491","7":"-20.40877899","8":"1.397319e-92","9":"-2.995547536","10":"-2.470938380"},{"1":"married","2":"DEP","3":"gender","4":"p_value","5":"-0.007143735","6":"0.032571007","7":"-0.21932803","8":"8.263945e-01","9":"-0.070656470","10":"0.057031917"},{"1":"married","2":"DEP","3":"gender","4":"c_value1","5":"0.002738526","6":"0.049790735","7":"0.05500072","8":"9.561379e-01","9":"-0.094815585","10":"0.100393816"},{"1":"married","2":"DEP","3":"none","4":"(Intercept)","5":"-2.731280949","6":"0.127507184","7":"-21.42060447","8":"8.586741e-102","9":"-2.983459436","10":"-2.483588073"},{"1":"married","2":"DEP","3":"none","4":"p_value","5":"-0.006836054","6":"0.032342101","7":"-0.21136704","8":"8.326009e-01","9":"-0.069899883","10":"0.056891340"},{"1":"married","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.707523502","6":"0.130159399","7":"-20.80159810","8":"4.186262e-96","9":"-2.964952446","10":"-2.454679888"},{"1":"married","2":"DEP","3":"parEdu","4":"p_value","5":"-0.026107743","6":"0.033039631","7":"-0.79019474","8":"4.294140e-01","9":"-0.090539502","10":"0.038987276"},{"1":"married","2":"DEP","3":"parEdu","4":"c_value1","5":"0.526383625","6":"0.061840351","7":"8.51197661","8":"1.709931e-17","9":"0.404078482","10":"0.646555744"},{"1":"married","2":"DEP","3":"parEdu","4":"c_value2","5":"0.399178554","6":"0.110060519","7":"3.62690054","8":"2.868437e-04","9":"0.177681514","10":"0.609576265"},{"1":"married","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-1.711759358","6":"0.138244802","7":"-12.38208842","8":"3.267286e-35","9":"-1.984632779","10":"-1.442666671"},{"1":"married","2":"DEP","3":"SRhealth","4":"p_value","5":"-0.299149455","6":"0.036250324","7":"-8.25232497","8":"1.553423e-16","9":"-0.369987880","10":"-0.227876318"},{"1":"married","2":"DEP","3":"SRhealth","4":"c_value","5":"0.704396229","6":"0.038116251","7":"18.48020733","8":"2.980339e-76","9":"0.629979710","10":"0.779398570"},{"1":"married","2":"NegAff","3":"age","4":"(Intercept)","5":"-3.914115809","6":"0.125799978","7":"-31.11380350","8":"1.567031e-212","9":"-4.162648464","10":"-3.669460202"},{"1":"married","2":"NegAff","3":"age","4":"p_value","5":"0.155458848","6":"0.042214928","7":"3.68255623","8":"2.309069e-04","9":"0.072500189","10":"0.238001788"},{"1":"married","2":"NegAff","3":"age","4":"c_value","5":"-0.056497637","6":"0.002260112","7":"-24.99771663","8":"6.473097e-138","9":"-0.060967144","10":"-0.052106007"},{"1":"married","2":"NegAff","3":"gender","4":"(Intercept)","5":"-3.656024235","6":"0.117667710","7":"-31.07075196","8":"5.984271e-212","9":"-3.888232201","10":"-3.426943204"},{"1":"married","2":"NegAff","3":"gender","4":"p_value","5":"0.266162230","6":"0.040749053","7":"6.53174024","8":"6.500984e-11","9":"0.186093490","10":"0.345842898"},{"1":"married","2":"NegAff","3":"gender","4":"c_value1","5":"-0.097990500","6":"0.072643509","7":"-1.34892301","8":"1.773617e-01","9":"-0.240348662","10":"0.044504914"},{"1":"married","2":"NegAff","3":"none","4":"(Intercept)","5":"-3.681409575","6":"0.116222219","7":"-31.67560896","8":"3.367890e-220","9":"-3.910772858","10":"-3.455153309"},{"1":"married","2":"NegAff","3":"none","4":"p_value","5":"0.256448781","6":"0.040134937","7":"6.38966441","8":"1.662502e-10","9":"0.177572191","10":"0.334914928"},{"1":"married","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-3.848988044","6":"0.121303889","7":"-31.73012907","8":"5.969787e-221","9":"-4.088474822","10":"-3.612928548"},{"1":"married","2":"NegAff","3":"parEdu","4":"p_value","5":"0.274296160","6":"0.040930716","7":"6.70147474","8":"2.063265e-11","9":"0.193873848","10":"0.354338184"},{"1":"married","2":"NegAff","3":"parEdu","4":"c_value1","5":"0.716403963","6":"0.083675323","7":"8.56171133","8":"1.112044e-17","9":"0.550662165","10":"0.878826051"},{"1":"married","2":"NegAff","3":"parEdu","4":"c_value2","5":"0.321372248","6":"0.165341391","7":"1.94368903","8":"5.193296e-02","9":"-0.016695564","10":"0.633043604"},{"1":"married","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.245563986","6":"0.126657802","7":"-33.51995617","8":"2.467935e-246","9":"-4.495692128","10":"-3.999153787"},{"1":"married","2":"NegAff","3":"SRhealth","4":"p_value","5":"0.422405505","6":"0.042242266","7":"9.99959390","8":"1.530233e-23","9":"0.339481271","10":"0.505090596"},{"1":"married","2":"NegAff","3":"SRhealth","4":"c_value","5":"0.796512064","6":"0.054941707","7":"14.49740293","8":"1.258210e-47","9":"0.689538062","10":"0.904923165"},{"1":"married","2":"OP","3":"age","4":"(Intercept)","5":"-3.291188984","6":"0.165587253","7":"-19.87585950","8":"6.584905e-88","9":"-3.620228257","10":"-2.970986082"},{"1":"married","2":"OP","3":"age","4":"p_value","5":"0.074932718","6":"0.054621729","7":"1.37184816","8":"1.701107e-01","9":"-0.031675206","10":"0.182488720"},{"1":"married","2":"OP","3":"age","4":"c_value","5":"-0.051070474","6":"0.002676707","7":"-19.07959134","8":"3.731651e-81","9":"-0.056370492","10":"-0.045875529"},{"1":"married","2":"OP","3":"gender","4":"(Intercept)","5":"-3.418961350","6":"0.166027506","7":"-20.59274046","8":"3.188162e-94","9":"-3.748744616","10":"-3.097792398"},{"1":"married","2":"OP","3":"gender","4":"p_value","5":"0.274339450","6":"0.052026675","7":"5.27305372","8":"1.341723e-07","9":"0.172917508","10":"0.376902533"},{"1":"married","2":"OP","3":"gender","4":"c_value1","5":"-0.007528753","6":"0.082549100","7":"-0.09120334","8":"9.273310e-01","9":"-0.169274151","10":"0.154459306"},{"1":"married","2":"OP","3":"none","4":"(Intercept)","5":"-3.424477120","6":"0.159888284","7":"-21.41793657","8":"9.092844e-102","9":"-3.742237136","10":"-3.115344422"},{"1":"married","2":"OP","3":"none","4":"p_value","5":"0.274753038","6":"0.052021873","7":"5.28149064","8":"1.281370e-07","9":"0.173339573","10":"0.377305770"},{"1":"married","2":"OP","3":"parEdu","4":"(Intercept)","5":"-3.485275767","6":"0.165660694","7":"-21.03864037","8":"2.905909e-98","9":"-3.814609025","10":"-3.165080490"},{"1":"married","2":"OP","3":"parEdu","4":"p_value","5":"0.278965505","6":"0.053976204","7":"5.16830534","8":"2.362262e-07","9":"0.173736262","10":"0.385366258"},{"1":"married","2":"OP","3":"parEdu","4":"c_value1","5":"0.389320030","6":"0.106090410","7":"3.66970050","8":"2.428348e-04","9":"0.177994116","10":"0.594192210"},{"1":"married","2":"OP","3":"parEdu","4":"c_value2","5":"0.358867821","6":"0.173948384","7":"2.06307074","8":"3.910591e-02","9":"0.004007768","10":"0.687626685"},{"1":"married","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.025206010","6":"0.164671813","7":"-18.37112230","8":"2.237372e-75","9":"-3.352148834","10":"-2.706515880"},{"1":"married","2":"OP","3":"SRhealth","4":"p_value","5":"0.103870733","6":"0.055395540","7":"1.87507395","8":"6.078255e-02","9":"-0.004261429","10":"0.212924102"},{"1":"married","2":"OP","3":"SRhealth","4":"c_value","5":"0.573153040","6":"0.063319239","7":"9.05179930","8":"1.406316e-19","9":"0.449979457","10":"0.698213312"},{"1":"married","2":"PA","3":"age","4":"(Intercept)","5":"-4.564845026","6":"0.180071560","7":"-25.35017211","8":"8.947223e-142","9":"-4.922416818","10":"-4.216496983"},{"1":"married","2":"PA","3":"age","4":"p_value","5":"0.300407610","6":"0.047379603","7":"6.34044166","8":"2.291074e-10","9":"0.208167038","10":"0.393905179"},{"1":"married","2":"PA","3":"age","4":"c_value","5":"-0.054672374","6":"0.002268543","7":"-24.10021297","8":"2.486890e-128","9":"-0.059158963","10":"-0.050264848"},{"1":"married","2":"PA","3":"gender","4":"(Intercept)","5":"-4.739679135","6":"0.180211215","7":"-26.30068910","8":"1.883221e-152","9":"-5.097173881","10":"-4.390731484"},{"1":"married","2":"PA","3":"gender","4":"p_value","5":"0.486803662","6":"0.046183853","7":"10.54055987","8":"5.616354e-26","9":"0.396911584","10":"0.577954492"},{"1":"married","2":"PA","3":"gender","4":"c_value1","5":"-0.031725519","6":"0.071744947","7":"-0.44219866","8":"6.583455e-01","9":"-0.172300248","10":"0.109033632"},{"1":"married","2":"PA","3":"none","4":"(Intercept)","5":"-4.755813059","6":"0.177105754","7":"-26.85295628","8":"7.790072e-159","9":"-5.107378863","10":"-4.413114335"},{"1":"married","2":"PA","3":"none","4":"p_value","5":"0.486623335","6":"0.046149016","7":"10.54460905","8":"5.379613e-26","9":"0.396796493","10":"0.577702678"},{"1":"married","2":"PA","3":"parEdu","4":"(Intercept)","5":"-4.682021080","6":"0.178854530","7":"-26.17781660","8":"4.754780e-151","9":"-5.037154323","10":"-4.336027019"},{"1":"married","2":"PA","3":"parEdu","4":"p_value","5":"0.438317736","6":"0.046788599","7":"9.36804571","8":"7.388790e-21","9":"0.347252756","10":"0.530667190"},{"1":"married","2":"PA","3":"parEdu","4":"c_value1","5":"0.633536504","6":"0.084103858","7":"7.53278762","8":"4.966836e-14","9":"0.466952125","10":"0.796792734"},{"1":"married","2":"PA","3":"parEdu","4":"c_value2","5":"0.315981382","6":"0.165557376","7":"1.90859139","8":"5.631482e-02","9":"-0.022472836","10":"0.628113525"},{"1":"married","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-4.348663208","6":"0.181670212","7":"-23.93712853","8":"1.258223e-126","9":"-4.709104511","10":"-3.996947917"},{"1":"married","2":"PA","3":"SRhealth","4":"p_value","5":"0.350097658","6":"0.048357571","7":"7.23976924","8":"4.494488e-13","9":"0.255886941","10":"0.445449460"},{"1":"married","2":"PA","3":"SRhealth","4":"c_value","5":"0.556629845","6":"0.055213116","7":"10.08147858","8":"6.671454e-24","9":"0.449076236","10":"0.665520438"},{"1":"married","2":"SE","3":"age","4":"(Intercept)","5":"-3.307986653","6":"0.261371160","7":"-12.65628025","8":"1.032781e-36","9":"-3.831978188","10":"-2.806909880"},{"1":"married","2":"SE","3":"age","4":"p_value","5":"-0.100707093","6":"0.045082998","7":"-2.23381537","8":"2.549522e-02","9":"-0.188054665","10":"-0.011222927"},{"1":"married","2":"SE","3":"age","4":"c_value","5":"-0.062640293","6":"0.003752030","7":"-16.69504187","8":"1.424192e-62","9":"-0.070115775","10":"-0.055400198"},{"1":"married","2":"SE","3":"gender","4":"(Intercept)","5":"-2.425178993","6":"0.247871779","7":"-9.78400607","8":"1.318847e-22","9":"-2.922152513","10":"-1.949970906"},{"1":"married","2":"SE","3":"gender","4":"p_value","5":"-0.149984514","6":"0.042716153","7":"-3.51118965","8":"4.461060e-04","9":"-0.232605382","10":"-0.065068778"},{"1":"married","2":"SE","3":"gender","4":"c_value1","5":"0.011117783","6":"0.117585707","7":"0.09455046","8":"9.246719e-01","9":"-0.219130056","10":"0.242249853"},{"1":"married","2":"SE","3":"none","4":"(Intercept)","5":"-2.418268120","6":"0.236221215","7":"-10.23730286","8":"1.349459e-24","9":"-2.893067583","10":"-1.966517264"},{"1":"married","2":"SE","3":"none","4":"p_value","5":"-0.150229939","6":"0.042651559","7":"-3.52226136","8":"4.278821e-04","9":"-0.232715712","10":"-0.065431859"},{"1":"married","2":"SE","3":"parEdu","4":"(Intercept)","5":"-2.572541676","6":"0.242707686","7":"-10.59934162","8":"3.000853e-26","9":"-3.060141710","10":"-2.108160952"},{"1":"married","2":"SE","3":"parEdu","4":"p_value","5":"-0.155525023","6":"0.043166718","7":"-3.60289201","8":"3.146963e-04","9":"-0.239027444","10":"-0.069716845"},{"1":"married","2":"SE","3":"parEdu","4":"c_value1","5":"0.905262036","6":"0.130296763","7":"6.94769398","8":"3.713053e-12","9":"0.646370114","10":"1.157733466"},{"1":"married","2":"SE","3":"parEdu","4":"c_value2","5":"0.357952626","6":"0.283390803","7":"1.26310601","8":"2.065511e-01","9":"-0.242401967","10":"0.876876235"},{"1":"married","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-2.072115973","6":"0.241058889","7":"-8.59589118","8":"8.262107e-18","9":"-2.555975014","10":"-1.610375173"},{"1":"married","2":"SE","3":"SRhealth","4":"p_value","5":"-0.230618266","6":"0.044345328","7":"-5.20050872","8":"1.987438e-07","9":"-0.316580862","10":"-0.142641311"},{"1":"married","2":"SE","3":"SRhealth","4":"c_value","5":"0.697245649","6":"0.089019724","7":"7.83248491","8":"4.783206e-15","9":"0.524546174","10":"0.873559377"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"(Intercept)","5":"-3.121072788","6":"0.153940089","7":"-20.27459390","8":"2.155638e-91","9":"-3.425705157","10":"-2.822197477"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"p_value","5":"-0.132419255","6":"0.038964861","7":"-3.39842752","8":"6.777442e-04","9":"-0.208437017","10":"-0.055676156"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"c_value","5":"-0.067147940","6":"0.001907397","7":"-35.20396671","8":"1.738754e-271","9":"-0.070915071","10":"-0.063437407"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.683681386","6":"0.147614285","7":"-18.18036367","8":"7.384247e-74","9":"-2.975781454","10":"-2.397043279"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"p_value","5":"-0.092672692","6":"0.036172075","7":"-2.56199549","8":"1.040727e-02","9":"-0.163177322","10":"-0.021358496"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"c_value1","5":"0.009477625","6":"0.056738396","7":"0.16704077","8":"8.673380e-01","9":"-0.101668275","10":"0.120815388"},{"1":"mvInPrtnr","2":"DEP","3":"none","4":"(Intercept)","5":"-2.678550743","6":"0.140226024","7":"-19.10166646","8":"2.445556e-81","9":"-2.956261918","10":"-2.406485488"},{"1":"mvInPrtnr","2":"DEP","3":"none","4":"p_value","5":"-0.092832330","6":"0.035917100","7":"-2.58462768","8":"9.748420e-03","9":"-0.162837103","10":"-0.022017630"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.722627375","6":"0.143910305","7":"-18.91891882","8":"7.967210e-80","9":"-3.007627846","10":"-2.443437283"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"p_value","5":"-0.118550419","6":"0.036863950","7":"-3.21589031","8":"1.300405e-03","9":"-0.190406928","10":"-0.045884997"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"c_value1","5":"0.819183825","6":"0.067502188","7":"12.13566328","8":"6.835388e-34","9":"0.685764514","10":"0.950453190"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"c_value2","5":"0.843535476","6":"0.109013673","7":"7.73788695","8":"1.010827e-14","9":"0.624690702","10":"1.052446538"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-1.437663703","6":"0.152764311","7":"-9.41099199","8":"4.914763e-21","9":"-1.739467859","10":"-1.140557043"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"p_value","5":"-0.454131164","6":"0.040709652","7":"-11.15536823","8":"6.741314e-29","9":"-0.533679992","10":"-0.374081169"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"c_value","5":"0.860167804","6":"0.044282935","7":"19.42436302","8":"4.803073e-84","9":"0.773752975","10":"0.947348502"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"(Intercept)","5":"-4.598726037","6":"0.161614787","7":"-28.45485930","8":"4.243109e-178","9":"-4.918726121","10":"-4.285089053"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"p_value","5":"0.182994234","6":"0.053141733","7":"3.44351274","8":"5.742095e-04","9":"0.078436050","10":"0.286788428"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"c_value","5":"-0.062808582","6":"0.002949864","7":"-21.29202923","8":"1.345731e-100","9":"-0.068657417","10":"-0.057090535"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"(Intercept)","5":"-4.269598441","6":"0.150991273","7":"-28.27712071","8":"6.606853e-176","9":"-4.568168537","10":"-3.976217144"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"p_value","5":"0.290271745","6":"0.051661197","7":"5.61875767","8":"1.923354e-08","9":"0.188625203","10":"0.391159553"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"c_value1","5":"-0.034564116","6":"0.093042389","7":"-0.37148785","8":"7.102742e-01","9":"-0.216712771","10":"0.148224886"},{"1":"mvInPrtnr","2":"NegAff","3":"none","4":"(Intercept)","5":"-4.278844009","6":"0.148782681","7":"-28.75901936","8":"6.986232e-182","9":"-4.573025360","10":"-3.989740748"},{"1":"mvInPrtnr","2":"NegAff","3":"none","4":"p_value","5":"0.286719473","6":"0.050925921","7":"5.63012834","8":"1.800756e-08","9":"0.186500389","10":"0.386154715"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-4.517081235","6":"0.156205544","7":"-28.91754751","8":"7.184657e-184","9":"-4.826073709","10":"-4.213667899"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"p_value","5":"0.299769242","6":"0.052074771","7":"5.75651583","8":"8.586773e-09","9":"0.197308218","10":"0.401473124"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"c_value1","5":"0.891695590","6":"0.105804822","7":"8.42774056","8":"3.524033e-17","9":"0.681682629","10":"1.096778688"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"c_value2","5":"0.867727878","6":"0.173343643","7":"5.00582462","8":"5.562338e-07","9":"0.513988538","10":"1.195255016"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.853082561","6":"0.161456687","7":"-30.05810813","8":"1.710676e-198","9":"-5.172439672","10":"-4.539468406"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"p_value","5":"0.455819860","6":"0.053309932","7":"8.55037401","8":"1.226903e-17","9":"0.351009519","10":"0.560016343"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"c_value","5":"0.816053817","6":"0.070325959","7":"11.60387752","8":"3.938212e-31","9":"0.679277088","10":"0.954973578"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"(Intercept)","5":"-4.361798412","6":"0.211894203","7":"-20.58479353","8":"3.756344e-94","9":"-4.784549857","10":"-3.953698844"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"p_value","5":"0.209523594","6":"0.066629536","7":"3.14460533","8":"1.663110e-03","9":"0.079790375","10":"0.341053508"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"c_value","5":"-0.071443189","6":"0.003473521","7":"-20.56794239","8":"5.317465e-94","9":"-0.078350991","10":"-0.064730783"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"(Intercept)","5":"-4.360046324","6":"0.206067956","7":"-21.15829369","8":"2.314476e-99","9":"-4.770704671","10":"-3.962750119"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"p_value","5":"0.459819444","6":"0.062575072","7":"7.34828471","8":"2.007665e-13","9":"0.338125206","10":"0.583476362"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"c_value1","5":"0.102710365","6":"0.096327238","7":"1.06626502","8":"2.863039e-01","9":"-0.085738942","10":"0.292095582"},{"1":"mvInPrtnr","2":"OP","3":"none","4":"(Intercept)","5":"-4.302485133","6":"0.198018155","7":"-21.72773059","8":"1.122124e-104","9":"-4.697381569","10":"-3.920968955"},{"1":"mvInPrtnr","2":"OP","3":"none","4":"p_value","5":"0.458717666","6":"0.062512524","7":"7.33801228","8":"2.167893e-13","9":"0.337144572","10":"0.582250578"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.282298285","6":"0.202819803","7":"-21.11380758","8":"5.938994e-99","9":"-4.686837228","10":"-3.891595427"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"p_value","5":"0.404030502","6":"0.064408260","7":"6.27296096","8":"3.542457e-10","9":"0.278713343","10":"0.531253805"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"c_value1","5":"0.778410162","6":"0.114081897","7":"6.82325753","8":"8.899890e-12","9":"0.551714433","10":"0.999295366"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"c_value2","5":"0.625575597","6":"0.187454112","7":"3.33721992","8":"8.462096e-04","9":"0.242459933","10":"0.979407546"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.890114661","6":"0.203395417","7":"-19.12587173","8":"1.537810e-81","9":"-4.295339007","10":"-3.497848016"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"p_value","5":"0.282144783","6":"0.066418881","7":"4.24796051","8":"2.157254e-05","9":"0.152771693","10":"0.413184654"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"c_value","5":"0.592767362","6":"0.074342704","7":"7.97344366","8":"1.543130e-15","9":"0.448284646","10":"0.739734651"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"(Intercept)","5":"-4.463638481","6":"0.216333668","7":"-20.63311978","8":"1.384250e-94","9":"-4.894830000","10":"-4.046712492"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"p_value","5":"0.101737448","6":"0.058338830","7":"1.74390623","8":"8.117544e-02","9":"-0.011695315","10":"0.217017459"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"c_value","5":"-0.062326605","6":"0.002968587","7":"-20.99537732","8":"7.228671e-98","9":"-0.068212969","10":"-0.056572850"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"(Intercept)","5":"-4.620627305","6":"0.217868704","7":"-21.20831129","8":"8.003507e-100","9":"-5.054458823","10":"-4.200353283"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"p_value","5":"0.303440378","6":"0.056974908","7":"5.32585995","8":"1.004765e-07","9":"0.192727510","10":"0.416078377"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"c_value1","5":"0.055757398","6":"0.091931858","7":"0.60650790","8":"5.441775e-01","9":"-0.124157412","10":"0.236437605"},{"1":"mvInPrtnr","2":"PA","3":"none","4":"(Intercept)","5":"-4.594037677","6":"0.212799106","7":"-21.58861364","8":"2.298033e-103","9":"-5.018197553","10":"-4.183972532"},{"1":"mvInPrtnr","2":"PA","3":"none","4":"p_value","5":"0.304167971","6":"0.056994093","7":"5.33683322","8":"9.458397e-08","9":"0.193416591","10":"0.416842313"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"(Intercept)","5":"-4.672662858","6":"0.217757112","7":"-21.45814118","8":"3.833248e-102","9":"-5.106846988","10":"-4.253158270"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"p_value","5":"0.271061894","6":"0.058266837","7":"4.65207838","8":"3.286061e-06","9":"0.157854589","10":"0.386274928"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"c_value1","5":"0.840529677","6":"0.106248343","7":"7.91099094","8":"2.553481e-15","9":"0.629662543","10":"1.046494834"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"c_value2","5":"0.884280773","6":"0.173211488","7":"5.10520858","8":"3.304299e-07","9":"0.530792163","10":"1.211546506"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-4.138438089","6":"0.217845941","7":"-18.99708607","8":"1.802796e-80","9":"-4.572503145","10":"-3.718427130"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"p_value","5":"0.149897790","6":"0.059758857","7":"2.50837781","8":"1.212869e-02","9":"0.033669022","10":"0.267947316"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"c_value","5":"0.618083863","6":"0.070942416","7":"8.71247270","8":"2.973169e-18","9":"0.480032206","10":"0.758169717"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"(Intercept)","5":"-4.806692399","6":"0.395577777","7":"-12.15106783","8":"5.662119e-34","9":"-5.610123591","10":"-4.058140488"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"p_value","5":"0.068102143","6":"0.066145448","7":"1.02958170","8":"3.032064e-01","9":"-0.058614985","10":"0.200912218"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"c_value","5":"-0.058175554","6":"0.004933985","7":"-11.79078413","8":"4.354675e-32","9":"-0.068033860","10":"-0.048672343"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"(Intercept)","5":"-3.843473335","6":"0.379699884","7":"-10.12239798","8":"4.395132e-24","9":"-4.615109258","10":"-3.125343365"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"p_value","5":"-0.003754989","6":"0.063867018","7":"-0.05879386","8":"9.531163e-01","9":"-0.125993504","10":"0.124585642"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"c_value1","5":"-0.086428090","6":"0.161898560","7":"-0.53384100","8":"5.934515e-01","9":"-0.404395587","10":"0.231533501"},{"1":"mvInPrtnr","2":"SE","3":"none","4":"(Intercept)","5":"-3.896976194","6":"0.366791131","7":"-10.62451043","8":"2.292117e-26","9":"-4.644575029","10":"-3.205372994"},{"1":"mvInPrtnr","2":"SE","3":"none","4":"p_value","5":"-0.002132562","6":"0.063784078","7":"-0.03343408","8":"9.733284e-01","9":"-0.124199083","10":"0.126055179"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.028242766","6":"0.375065603","7":"-10.74010183","8":"6.597158e-27","9":"-4.792315053","10":"-3.320745773"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"p_value","5":"-0.012036760","6":"0.064515743","7":"-0.18657089","8":"8.519971e-01","9":"-0.135500086","10":"0.117617419"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"c_value1","5":"0.754347399","6":"0.190423109","7":"3.96142782","8":"7.450291e-05","9":"0.370828043","10":"1.119409011"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"c_value2","5":"1.103987451","6":"0.283548356","7":"3.89347153","8":"9.881980e-05","9":"0.509540199","10":"1.628888223"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-3.612393890","6":"0.371971555","7":"-9.71147885","8":"2.693996e-22","9":"-4.369867291","10":"-2.910417488"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"p_value","5":"-0.064698695","6":"0.065575016","7":"-0.98663635","8":"3.238209e-01","9":"-0.190417337","10":"0.066843901"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"c_value","5":"0.571645292","6":"0.120998710","7":"4.72439163","8":"2.308053e-06","9":"0.337594647","10":"0.811973834"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Create the Table  
The basic steps from here are similar: filter target terms, index significance, exponentiate, format values, create CI's, bold significance, select needed columns.  



```{.r .code-style}
tidy3 <- tidy3 %>%
  filter(term == "p_value") %>%
  mutate(sig = ifelse(sign(conf.low) == sign(conf.high), "sig", "ns")) %>%
  mutate_at(vars(estimate, conf.low, conf.high), exp) %>%
  mutate_at(vars(estimate, conf.low, conf.high), ~sprintf("%.2f", .)) %>%
  mutate(CI = sprintf("[%s, %s]", conf.low, conf.high)) %>%
  mutate_at(vars(estimate, CI), ~ifelse(sig == "sig", sprintf("<strong>%s</strong>", .), .)) %>%
  select(Outcome, Trait, Covariate, OR = estimate, CI)
tidy3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["OR"],"name":[4],"type":["chr"],"align":["left"]},{"label":["CI"],"name":[5],"type":["chr"],"align":["left"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"<strong>1.16<\/strong>","5":"<strong>[1.07, 1.25]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"gender","4":"<strong>1.16<\/strong>","5":"<strong>[1.08, 1.25]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"none","4":"<strong>1.15<\/strong>","5":"<strong>[1.07, 1.24]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"<strong>1.12<\/strong>","5":"<strong>[1.05, 1.21]<\/strong>"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"<strong>0.77<\/strong>","5":"<strong>[0.71, 0.83]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"age","4":"0.99","5":"[0.90, 1.09]"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"<strong>1.11<\/strong>","5":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"none","4":"<strong>1.11<\/strong>","5":"<strong>[1.01, 1.22]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"<strong>1.12<\/strong>","5":"<strong>[1.02, 1.22]<\/strong>"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"<strong>1.39<\/strong>","5":"<strong>[1.26, 1.53]<\/strong>"},{"1":"chldbrth","2":"OP","3":"age","4":"<strong>1.33<\/strong>","5":"<strong>[1.18, 1.50]<\/strong>"},{"1":"chldbrth","2":"OP","3":"gender","4":"<strong>1.68<\/strong>","5":"<strong>[1.50, 1.89]<\/strong>"},{"1":"chldbrth","2":"OP","3":"none","4":"<strong>1.68<\/strong>","5":"<strong>[1.50, 1.88]<\/strong>"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"<strong>1.63<\/strong>","5":"<strong>[1.45, 1.83]<\/strong>"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"<strong>1.31<\/strong>","5":"<strong>[1.16, 1.48]<\/strong>"},{"1":"chldbrth","2":"PA","3":"age","4":"<strong>1.61<\/strong>","5":"<strong>[1.44, 1.80]<\/strong>"},{"1":"chldbrth","2":"PA","3":"gender","4":"<strong>1.96<\/strong>","5":"<strong>[1.76, 2.19]<\/strong>"},{"1":"chldbrth","2":"PA","3":"none","4":"<strong>1.97<\/strong>","5":"<strong>[1.77, 2.20]<\/strong>"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"<strong>1.87<\/strong>","5":"<strong>[1.68, 2.09]<\/strong>"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"<strong>1.61<\/strong>","5":"<strong>[1.44, 1.81]<\/strong>"},{"1":"chldbrth","2":"SE","3":"age","4":"<strong>1.18<\/strong>","5":"<strong>[1.05, 1.33]<\/strong>"},{"1":"chldbrth","2":"SE","3":"gender","4":"1.09","5":"[0.98, 1.23]"},{"1":"chldbrth","2":"SE","3":"none","4":"1.09","5":"[0.97, 1.22]"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"1.08","5":"[0.97, 1.22]"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"0.98","5":"[0.87, 1.10]"},{"1":"divorced","2":"DEP","3":"age","4":"<strong>0.73<\/strong>","5":"<strong>[0.66, 0.80]<\/strong>"},{"1":"divorced","2":"DEP","3":"gender","4":"<strong>0.74<\/strong>","5":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced","2":"DEP","3":"none","4":"<strong>0.74<\/strong>","5":"<strong>[0.67, 0.82]<\/strong>"},{"1":"divorced","2":"DEP","3":"parEdu","4":"<strong>0.76<\/strong>","5":"<strong>[0.68, 0.84]<\/strong>"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"<strong>0.63<\/strong>","5":"<strong>[0.56, 0.70]<\/strong>"},{"1":"divorced","2":"NegAff","3":"age","4":"<strong>1.51<\/strong>","5":"<strong>[1.32, 1.73]<\/strong>"},{"1":"divorced","2":"NegAff","3":"gender","4":"<strong>1.59<\/strong>","5":"<strong>[1.39, 1.82]<\/strong>"},{"1":"divorced","2":"NegAff","3":"none","4":"<strong>1.58<\/strong>","5":"<strong>[1.38, 1.80]<\/strong>"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"<strong>1.61<\/strong>","5":"<strong>[1.40, 1.84]<\/strong>"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"<strong>1.70<\/strong>","5":"<strong>[1.48, 1.95]<\/strong>"},{"1":"divorced","2":"OP","3":"age","4":"1.00","5":"[0.85, 1.19]"},{"1":"divorced","2":"OP","3":"gender","4":"1.10","5":"[0.93, 1.30]"},{"1":"divorced","2":"OP","3":"none","4":"1.10","5":"[0.93, 1.30]"},{"1":"divorced","2":"OP","3":"parEdu","4":"1.07","5":"[0.90, 1.27]"},{"1":"divorced","2":"OP","3":"SRhealth","4":"1.04","5":"[0.87, 1.24]"},{"1":"divorced","2":"PA","3":"age","4":"<strong>0.77<\/strong>","5":"<strong>[0.68, 0.89]<\/strong>"},{"1":"divorced","2":"PA","3":"gender","4":"<strong>0.85<\/strong>","5":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced","2":"PA","3":"none","4":"<strong>0.85<\/strong>","5":"<strong>[0.74, 0.97]<\/strong>"},{"1":"divorced","2":"PA","3":"parEdu","4":"<strong>0.87<\/strong>","5":"<strong>[0.75, 1.00]<\/strong>"},{"1":"divorced","2":"PA","3":"SRhealth","4":"<strong>0.79<\/strong>","5":"<strong>[0.68, 0.91]<\/strong>"},{"1":"divorced","2":"SE","3":"age","4":"0.97","5":"[0.82, 1.15]"},{"1":"divorced","2":"SE","3":"gender","4":"0.94","5":"[0.81, 1.12]"},{"1":"divorced","2":"SE","3":"none","4":"0.95","5":"[0.81, 1.12]"},{"1":"divorced","2":"SE","3":"parEdu","4":"0.96","5":"[0.82, 1.15]"},{"1":"divorced","2":"SE","3":"SRhealth","4":"0.92","5":"[0.78, 1.09]"},{"1":"married","2":"DEP","3":"age","4":"0.99","5":"[0.92, 1.06]"},{"1":"married","2":"DEP","3":"gender","4":"0.99","5":"[0.93, 1.06]"},{"1":"married","2":"DEP","3":"none","4":"0.99","5":"[0.93, 1.06]"},{"1":"married","2":"DEP","3":"parEdu","4":"0.97","5":"[0.91, 1.04]"},{"1":"married","2":"DEP","3":"SRhealth","4":"<strong>0.74<\/strong>","5":"<strong>[0.69, 0.80]<\/strong>"},{"1":"married","2":"NegAff","3":"age","4":"<strong>1.17<\/strong>","5":"<strong>[1.08, 1.27]<\/strong>"},{"1":"married","2":"NegAff","3":"gender","4":"<strong>1.30<\/strong>","5":"<strong>[1.20, 1.41]<\/strong>"},{"1":"married","2":"NegAff","3":"none","4":"<strong>1.29<\/strong>","5":"<strong>[1.19, 1.40]<\/strong>"},{"1":"married","2":"NegAff","3":"parEdu","4":"<strong>1.32<\/strong>","5":"<strong>[1.21, 1.43]<\/strong>"},{"1":"married","2":"NegAff","3":"SRhealth","4":"<strong>1.53<\/strong>","5":"<strong>[1.40, 1.66]<\/strong>"},{"1":"married","2":"OP","3":"age","4":"1.08","5":"[0.97, 1.20]"},{"1":"married","2":"OP","3":"gender","4":"<strong>1.32<\/strong>","5":"<strong>[1.19, 1.46]<\/strong>"},{"1":"married","2":"OP","3":"none","4":"<strong>1.32<\/strong>","5":"<strong>[1.19, 1.46]<\/strong>"},{"1":"married","2":"OP","3":"parEdu","4":"<strong>1.32<\/strong>","5":"<strong>[1.19, 1.47]<\/strong>"},{"1":"married","2":"OP","3":"SRhealth","4":"1.11","5":"[1.00, 1.24]"},{"1":"married","2":"PA","3":"age","4":"<strong>1.35<\/strong>","5":"<strong>[1.23, 1.48]<\/strong>"},{"1":"married","2":"PA","3":"gender","4":"<strong>1.63<\/strong>","5":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married","2":"PA","3":"none","4":"<strong>1.63<\/strong>","5":"<strong>[1.49, 1.78]<\/strong>"},{"1":"married","2":"PA","3":"parEdu","4":"<strong>1.55<\/strong>","5":"<strong>[1.42, 1.70]<\/strong>"},{"1":"married","2":"PA","3":"SRhealth","4":"<strong>1.42<\/strong>","5":"<strong>[1.29, 1.56]<\/strong>"},{"1":"married","2":"SE","3":"age","4":"<strong>0.90<\/strong>","5":"<strong>[0.83, 0.99]<\/strong>"},{"1":"married","2":"SE","3":"gender","4":"<strong>0.86<\/strong>","5":"<strong>[0.79, 0.94]<\/strong>"},{"1":"married","2":"SE","3":"none","4":"<strong>0.86<\/strong>","5":"<strong>[0.79, 0.94]<\/strong>"},{"1":"married","2":"SE","3":"parEdu","4":"<strong>0.86<\/strong>","5":"<strong>[0.79, 0.93]<\/strong>"},{"1":"married","2":"SE","3":"SRhealth","4":"<strong>0.79<\/strong>","5":"<strong>[0.73, 0.87]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"<strong>0.88<\/strong>","5":"<strong>[0.81, 0.95]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"<strong>0.91<\/strong>","5":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"none","4":"<strong>0.91<\/strong>","5":"<strong>[0.85, 0.98]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"<strong>0.89<\/strong>","5":"<strong>[0.83, 0.96]<\/strong>"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"<strong>0.63<\/strong>","5":"<strong>[0.59, 0.69]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"<strong>1.20<\/strong>","5":"<strong>[1.08, 1.33]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"<strong>1.34<\/strong>","5":"<strong>[1.21, 1.48]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"none","4":"<strong>1.33<\/strong>","5":"<strong>[1.21, 1.47]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"<strong>1.35<\/strong>","5":"<strong>[1.22, 1.49]<\/strong>"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"<strong>1.58<\/strong>","5":"<strong>[1.42, 1.75]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"<strong>1.23<\/strong>","5":"<strong>[1.08, 1.41]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"<strong>1.58<\/strong>","5":"<strong>[1.40, 1.79]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"none","4":"<strong>1.58<\/strong>","5":"<strong>[1.40, 1.79]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"<strong>1.50<\/strong>","5":"<strong>[1.32, 1.70]<\/strong>"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"<strong>1.33<\/strong>","5":"<strong>[1.17, 1.51]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"1.11","5":"[0.99, 1.24]"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"<strong>1.35<\/strong>","5":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"none","4":"<strong>1.36<\/strong>","5":"<strong>[1.21, 1.52]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"<strong>1.31<\/strong>","5":"<strong>[1.17, 1.47]<\/strong>"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"<strong>1.16<\/strong>","5":"<strong>[1.03, 1.31]<\/strong>"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"1.07","5":"[0.94, 1.22]"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"1.00","5":"[0.88, 1.13]"},{"1":"mvInPrtnr","2":"SE","3":"none","4":"1.00","5":"[0.88, 1.13]"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"0.99","5":"[0.87, 1.12]"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"0.94","5":"[0.83, 1.07]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Create the Table  
Now we're ready for the `pivot_longer()`, `unite()`, `pivot_wider()`, order columns (using `select()`) combo from before. The only difference is that I'm going to use `pivot_wider()` to do the uniting for me this time!  


```{.r .code-style}
O_names <- tibble(
  old = c("mvInPrtnr", "married", "divorced", "chldbrth"),
  new = c("Move in with Partner", "Married", "Divorced", "Birth of a Child")
)
levs <- paste(rep(O_names$old, each = 2), rep(c("OR","CI"), times = 4), sep = "_")

tidy3 <- tidy3 %>%
  pivot_longer(cols = c(OR, CI), names_to = "est", values_to = "value") %>%
  pivot_wider(names_from = c("Outcome", "est"), values_from = "value", names_sep = "_") %>%
  select(Trait, Covariate, all_of(levs))
tidy3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Trait"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[2],"type":["chr"],"align":["left"]},{"label":["mvInPrtnr_OR"],"name":[3],"type":["chr"],"align":["left"]},{"label":["mvInPrtnr_CI"],"name":[4],"type":["chr"],"align":["left"]},{"label":["married_OR"],"name":[5],"type":["chr"],"align":["left"]},{"label":["married_CI"],"name":[6],"type":["chr"],"align":["left"]},{"label":["divorced_OR"],"name":[7],"type":["chr"],"align":["left"]},{"label":["divorced_CI"],"name":[8],"type":["chr"],"align":["left"]},{"label":["chldbrth_OR"],"name":[9],"type":["chr"],"align":["left"]},{"label":["chldbrth_CI"],"name":[10],"type":["chr"],"align":["left"]}],"data":[{"1":"DEP","2":"age","3":"<strong>0.88<\/strong>","4":"<strong>[0.81, 0.95]<\/strong>","5":"0.99","6":"[0.92, 1.06]","7":"<strong>0.73<\/strong>","8":"<strong>[0.66, 0.80]<\/strong>","9":"<strong>1.16<\/strong>","10":"<strong>[1.07, 1.25]<\/strong>"},{"1":"DEP","2":"gender","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>","5":"0.99","6":"[0.93, 1.06]","7":"<strong>0.74<\/strong>","8":"<strong>[0.67, 0.82]<\/strong>","9":"<strong>1.16<\/strong>","10":"<strong>[1.08, 1.25]<\/strong>"},{"1":"DEP","2":"none","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>","5":"0.99","6":"[0.93, 1.06]","7":"<strong>0.74<\/strong>","8":"<strong>[0.67, 0.82]<\/strong>","9":"<strong>1.15<\/strong>","10":"<strong>[1.07, 1.24]<\/strong>"},{"1":"DEP","2":"parEdu","3":"<strong>0.89<\/strong>","4":"<strong>[0.83, 0.96]<\/strong>","5":"0.97","6":"[0.91, 1.04]","7":"<strong>0.76<\/strong>","8":"<strong>[0.68, 0.84]<\/strong>","9":"<strong>1.12<\/strong>","10":"<strong>[1.05, 1.21]<\/strong>"},{"1":"DEP","2":"SRhealth","3":"<strong>0.63<\/strong>","4":"<strong>[0.59, 0.69]<\/strong>","5":"<strong>0.74<\/strong>","6":"<strong>[0.69, 0.80]<\/strong>","7":"<strong>0.63<\/strong>","8":"<strong>[0.56, 0.70]<\/strong>","9":"<strong>0.77<\/strong>","10":"<strong>[0.71, 0.83]<\/strong>"},{"1":"NegAff","2":"age","3":"<strong>1.20<\/strong>","4":"<strong>[1.08, 1.33]<\/strong>","5":"<strong>1.17<\/strong>","6":"<strong>[1.08, 1.27]<\/strong>","7":"<strong>1.51<\/strong>","8":"<strong>[1.32, 1.73]<\/strong>","9":"0.99","10":"[0.90, 1.09]"},{"1":"NegAff","2":"gender","3":"<strong>1.34<\/strong>","4":"<strong>[1.21, 1.48]<\/strong>","5":"<strong>1.30<\/strong>","6":"<strong>[1.20, 1.41]<\/strong>","7":"<strong>1.59<\/strong>","8":"<strong>[1.39, 1.82]<\/strong>","9":"<strong>1.11<\/strong>","10":"<strong>[1.01, 1.22]<\/strong>"},{"1":"NegAff","2":"none","3":"<strong>1.33<\/strong>","4":"<strong>[1.21, 1.47]<\/strong>","5":"<strong>1.29<\/strong>","6":"<strong>[1.19, 1.40]<\/strong>","7":"<strong>1.58<\/strong>","8":"<strong>[1.38, 1.80]<\/strong>","9":"<strong>1.11<\/strong>","10":"<strong>[1.01, 1.22]<\/strong>"},{"1":"NegAff","2":"parEdu","3":"<strong>1.35<\/strong>","4":"<strong>[1.22, 1.49]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.21, 1.43]<\/strong>","7":"<strong>1.61<\/strong>","8":"<strong>[1.40, 1.84]<\/strong>","9":"<strong>1.12<\/strong>","10":"<strong>[1.02, 1.22]<\/strong>"},{"1":"NegAff","2":"SRhealth","3":"<strong>1.58<\/strong>","4":"<strong>[1.42, 1.75]<\/strong>","5":"<strong>1.53<\/strong>","6":"<strong>[1.40, 1.66]<\/strong>","7":"<strong>1.70<\/strong>","8":"<strong>[1.48, 1.95]<\/strong>","9":"<strong>1.39<\/strong>","10":"<strong>[1.26, 1.53]<\/strong>"},{"1":"OP","2":"age","3":"<strong>1.23<\/strong>","4":"<strong>[1.08, 1.41]<\/strong>","5":"1.08","6":"[0.97, 1.20]","7":"1.00","8":"[0.85, 1.19]","9":"<strong>1.33<\/strong>","10":"<strong>[1.18, 1.50]<\/strong>"},{"1":"OP","2":"gender","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.46]<\/strong>","7":"1.10","8":"[0.93, 1.30]","9":"<strong>1.68<\/strong>","10":"<strong>[1.50, 1.89]<\/strong>"},{"1":"OP","2":"none","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.46]<\/strong>","7":"1.10","8":"[0.93, 1.30]","9":"<strong>1.68<\/strong>","10":"<strong>[1.50, 1.88]<\/strong>"},{"1":"OP","2":"parEdu","3":"<strong>1.50<\/strong>","4":"<strong>[1.32, 1.70]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.47]<\/strong>","7":"1.07","8":"[0.90, 1.27]","9":"<strong>1.63<\/strong>","10":"<strong>[1.45, 1.83]<\/strong>"},{"1":"OP","2":"SRhealth","3":"<strong>1.33<\/strong>","4":"<strong>[1.17, 1.51]<\/strong>","5":"1.11","6":"[1.00, 1.24]","7":"1.04","8":"[0.87, 1.24]","9":"<strong>1.31<\/strong>","10":"<strong>[1.16, 1.48]<\/strong>"},{"1":"PA","2":"age","3":"1.11","4":"[0.99, 1.24]","5":"<strong>1.35<\/strong>","6":"<strong>[1.23, 1.48]<\/strong>","7":"<strong>0.77<\/strong>","8":"<strong>[0.68, 0.89]<\/strong>","9":"<strong>1.61<\/strong>","10":"<strong>[1.44, 1.80]<\/strong>"},{"1":"PA","2":"gender","3":"<strong>1.35<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>","5":"<strong>1.63<\/strong>","6":"<strong>[1.49, 1.78]<\/strong>","7":"<strong>0.85<\/strong>","8":"<strong>[0.74, 0.97]<\/strong>","9":"<strong>1.96<\/strong>","10":"<strong>[1.76, 2.19]<\/strong>"},{"1":"PA","2":"none","3":"<strong>1.36<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>","5":"<strong>1.63<\/strong>","6":"<strong>[1.49, 1.78]<\/strong>","7":"<strong>0.85<\/strong>","8":"<strong>[0.74, 0.97]<\/strong>","9":"<strong>1.97<\/strong>","10":"<strong>[1.77, 2.20]<\/strong>"},{"1":"PA","2":"parEdu","3":"<strong>1.31<\/strong>","4":"<strong>[1.17, 1.47]<\/strong>","5":"<strong>1.55<\/strong>","6":"<strong>[1.42, 1.70]<\/strong>","7":"<strong>0.87<\/strong>","8":"<strong>[0.75, 1.00]<\/strong>","9":"<strong>1.87<\/strong>","10":"<strong>[1.68, 2.09]<\/strong>"},{"1":"PA","2":"SRhealth","3":"<strong>1.16<\/strong>","4":"<strong>[1.03, 1.31]<\/strong>","5":"<strong>1.42<\/strong>","6":"<strong>[1.29, 1.56]<\/strong>","7":"<strong>0.79<\/strong>","8":"<strong>[0.68, 0.91]<\/strong>","9":"<strong>1.61<\/strong>","10":"<strong>[1.44, 1.81]<\/strong>"},{"1":"SE","2":"age","3":"1.07","4":"[0.94, 1.22]","5":"<strong>0.90<\/strong>","6":"<strong>[0.83, 0.99]<\/strong>","7":"0.97","8":"[0.82, 1.15]","9":"<strong>1.18<\/strong>","10":"<strong>[1.05, 1.33]<\/strong>"},{"1":"SE","2":"gender","3":"1.00","4":"[0.88, 1.13]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.94]<\/strong>","7":"0.94","8":"[0.81, 1.12]","9":"1.09","10":"[0.98, 1.23]"},{"1":"SE","2":"none","3":"1.00","4":"[0.88, 1.13]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.94]<\/strong>","7":"0.95","8":"[0.81, 1.12]","9":"1.09","10":"[0.97, 1.22]"},{"1":"SE","2":"parEdu","3":"0.99","4":"[0.87, 1.12]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.93]<\/strong>","7":"0.96","8":"[0.82, 1.15]","9":"1.08","10":"[0.97, 1.22]"},{"1":"SE","2":"SRhealth","3":"0.94","4":"[0.83, 1.07]","5":"<strong>0.79<\/strong>","6":"<strong>[0.73, 0.87]<\/strong>","7":"0.92","8":"[0.78, 1.09]","9":"0.98","10":"[0.87, 1.10]"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Kabling the Table  
All right, time to use `kable()`! This will proceed almost exactly as before. The only difference from the previous example is that we have multiple different models with different combinations of p_value terms depending on what we controlled for. So we'll introduce a new function `collapse_rows()` from `kableExtra`.  

Given that we have multiple covariates, we'll also want to order them. So we'll make the Covariate column a factor as well. But we'll do so after we've given our covariates nicer names.  While we're at it, we'll go ahead and do the same things for our Trait column.  


```{.r .code-style}
c_names <- tibble(
  old = c("none", "age", "SRhealth", "gender", "parEdu"),
  new = c("None", "Age", "Self-Rated Health", "Gender", "Parental Education")
)

tidy3 <- tidy3 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new),
         Covariate = mapvalues(Covariate, from = c_names$old, to = c_names$new),
         Covariate = factor(Covariate, levels = c_names$new)) %>%
  arrange(Trait, Covariate)
tidy3
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Trait"],"name":[1],"type":["fct"],"align":["left"]},{"label":["Covariate"],"name":[2],"type":["fct"],"align":["left"]},{"label":["mvInPrtnr_OR"],"name":[3],"type":["chr"],"align":["left"]},{"label":["mvInPrtnr_CI"],"name":[4],"type":["chr"],"align":["left"]},{"label":["married_OR"],"name":[5],"type":["chr"],"align":["left"]},{"label":["married_CI"],"name":[6],"type":["chr"],"align":["left"]},{"label":["divorced_OR"],"name":[7],"type":["chr"],"align":["left"]},{"label":["divorced_CI"],"name":[8],"type":["chr"],"align":["left"]},{"label":["chldbrth_OR"],"name":[9],"type":["chr"],"align":["left"]},{"label":["chldbrth_CI"],"name":[10],"type":["chr"],"align":["left"]}],"data":[{"1":"Negative Affect","2":"None","3":"<strong>1.33<\/strong>","4":"<strong>[1.21, 1.47]<\/strong>","5":"<strong>1.29<\/strong>","6":"<strong>[1.19, 1.40]<\/strong>","7":"<strong>1.58<\/strong>","8":"<strong>[1.38, 1.80]<\/strong>","9":"<strong>1.11<\/strong>","10":"<strong>[1.01, 1.22]<\/strong>"},{"1":"Negative Affect","2":"Age","3":"<strong>1.20<\/strong>","4":"<strong>[1.08, 1.33]<\/strong>","5":"<strong>1.17<\/strong>","6":"<strong>[1.08, 1.27]<\/strong>","7":"<strong>1.51<\/strong>","8":"<strong>[1.32, 1.73]<\/strong>","9":"0.99","10":"[0.90, 1.09]"},{"1":"Negative Affect","2":"Self-Rated Health","3":"<strong>1.58<\/strong>","4":"<strong>[1.42, 1.75]<\/strong>","5":"<strong>1.53<\/strong>","6":"<strong>[1.40, 1.66]<\/strong>","7":"<strong>1.70<\/strong>","8":"<strong>[1.48, 1.95]<\/strong>","9":"<strong>1.39<\/strong>","10":"<strong>[1.26, 1.53]<\/strong>"},{"1":"Negative Affect","2":"Gender","3":"<strong>1.34<\/strong>","4":"<strong>[1.21, 1.48]<\/strong>","5":"<strong>1.30<\/strong>","6":"<strong>[1.20, 1.41]<\/strong>","7":"<strong>1.59<\/strong>","8":"<strong>[1.39, 1.82]<\/strong>","9":"<strong>1.11<\/strong>","10":"<strong>[1.01, 1.22]<\/strong>"},{"1":"Negative Affect","2":"Parental Education","3":"<strong>1.35<\/strong>","4":"<strong>[1.22, 1.49]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.21, 1.43]<\/strong>","7":"<strong>1.61<\/strong>","8":"<strong>[1.40, 1.84]<\/strong>","9":"<strong>1.12<\/strong>","10":"<strong>[1.02, 1.22]<\/strong>"},{"1":"Positive Affect","2":"None","3":"<strong>1.36<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>","5":"<strong>1.63<\/strong>","6":"<strong>[1.49, 1.78]<\/strong>","7":"<strong>0.85<\/strong>","8":"<strong>[0.74, 0.97]<\/strong>","9":"<strong>1.97<\/strong>","10":"<strong>[1.77, 2.20]<\/strong>"},{"1":"Positive Affect","2":"Age","3":"1.11","4":"[0.99, 1.24]","5":"<strong>1.35<\/strong>","6":"<strong>[1.23, 1.48]<\/strong>","7":"<strong>0.77<\/strong>","8":"<strong>[0.68, 0.89]<\/strong>","9":"<strong>1.61<\/strong>","10":"<strong>[1.44, 1.80]<\/strong>"},{"1":"Positive Affect","2":"Self-Rated Health","3":"<strong>1.16<\/strong>","4":"<strong>[1.03, 1.31]<\/strong>","5":"<strong>1.42<\/strong>","6":"<strong>[1.29, 1.56]<\/strong>","7":"<strong>0.79<\/strong>","8":"<strong>[0.68, 0.91]<\/strong>","9":"<strong>1.61<\/strong>","10":"<strong>[1.44, 1.81]<\/strong>"},{"1":"Positive Affect","2":"Gender","3":"<strong>1.35<\/strong>","4":"<strong>[1.21, 1.52]<\/strong>","5":"<strong>1.63<\/strong>","6":"<strong>[1.49, 1.78]<\/strong>","7":"<strong>0.85<\/strong>","8":"<strong>[0.74, 0.97]<\/strong>","9":"<strong>1.96<\/strong>","10":"<strong>[1.76, 2.19]<\/strong>"},{"1":"Positive Affect","2":"Parental Education","3":"<strong>1.31<\/strong>","4":"<strong>[1.17, 1.47]<\/strong>","5":"<strong>1.55<\/strong>","6":"<strong>[1.42, 1.70]<\/strong>","7":"<strong>0.87<\/strong>","8":"<strong>[0.75, 1.00]<\/strong>","9":"<strong>1.87<\/strong>","10":"<strong>[1.68, 2.09]<\/strong>"},{"1":"Self-Esteem","2":"None","3":"1.00","4":"[0.88, 1.13]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.94]<\/strong>","7":"0.95","8":"[0.81, 1.12]","9":"1.09","10":"[0.97, 1.22]"},{"1":"Self-Esteem","2":"Age","3":"1.07","4":"[0.94, 1.22]","5":"<strong>0.90<\/strong>","6":"<strong>[0.83, 0.99]<\/strong>","7":"0.97","8":"[0.82, 1.15]","9":"<strong>1.18<\/strong>","10":"<strong>[1.05, 1.33]<\/strong>"},{"1":"Self-Esteem","2":"Self-Rated Health","3":"0.94","4":"[0.83, 1.07]","5":"<strong>0.79<\/strong>","6":"<strong>[0.73, 0.87]<\/strong>","7":"0.92","8":"[0.78, 1.09]","9":"0.98","10":"[0.87, 1.10]"},{"1":"Self-Esteem","2":"Gender","3":"1.00","4":"[0.88, 1.13]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.94]<\/strong>","7":"0.94","8":"[0.81, 1.12]","9":"1.09","10":"[0.98, 1.23]"},{"1":"Self-Esteem","2":"Parental Education","3":"0.99","4":"[0.87, 1.12]","5":"<strong>0.86<\/strong>","6":"<strong>[0.79, 0.93]<\/strong>","7":"0.96","8":"[0.82, 1.15]","9":"1.08","10":"[0.97, 1.22]"},{"1":"Optimism","2":"None","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.46]<\/strong>","7":"1.10","8":"[0.93, 1.30]","9":"<strong>1.68<\/strong>","10":"<strong>[1.50, 1.88]<\/strong>"},{"1":"Optimism","2":"Age","3":"<strong>1.23<\/strong>","4":"<strong>[1.08, 1.41]<\/strong>","5":"1.08","6":"[0.97, 1.20]","7":"1.00","8":"[0.85, 1.19]","9":"<strong>1.33<\/strong>","10":"<strong>[1.18, 1.50]<\/strong>"},{"1":"Optimism","2":"Self-Rated Health","3":"<strong>1.33<\/strong>","4":"<strong>[1.17, 1.51]<\/strong>","5":"1.11","6":"[1.00, 1.24]","7":"1.04","8":"[0.87, 1.24]","9":"<strong>1.31<\/strong>","10":"<strong>[1.16, 1.48]<\/strong>"},{"1":"Optimism","2":"Gender","3":"<strong>1.58<\/strong>","4":"<strong>[1.40, 1.79]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.46]<\/strong>","7":"1.10","8":"[0.93, 1.30]","9":"<strong>1.68<\/strong>","10":"<strong>[1.50, 1.89]<\/strong>"},{"1":"Optimism","2":"Parental Education","3":"<strong>1.50<\/strong>","4":"<strong>[1.32, 1.70]<\/strong>","5":"<strong>1.32<\/strong>","6":"<strong>[1.19, 1.47]<\/strong>","7":"1.07","8":"[0.90, 1.27]","9":"<strong>1.63<\/strong>","10":"<strong>[1.45, 1.83]<\/strong>"},{"1":"Depression","2":"None","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>","5":"0.99","6":"[0.93, 1.06]","7":"<strong>0.74<\/strong>","8":"<strong>[0.67, 0.82]<\/strong>","9":"<strong>1.15<\/strong>","10":"<strong>[1.07, 1.24]<\/strong>"},{"1":"Depression","2":"Age","3":"<strong>0.88<\/strong>","4":"<strong>[0.81, 0.95]<\/strong>","5":"0.99","6":"[0.92, 1.06]","7":"<strong>0.73<\/strong>","8":"<strong>[0.66, 0.80]<\/strong>","9":"<strong>1.16<\/strong>","10":"<strong>[1.07, 1.25]<\/strong>"},{"1":"Depression","2":"Self-Rated Health","3":"<strong>0.63<\/strong>","4":"<strong>[0.59, 0.69]<\/strong>","5":"<strong>0.74<\/strong>","6":"<strong>[0.69, 0.80]<\/strong>","7":"<strong>0.63<\/strong>","8":"<strong>[0.56, 0.70]<\/strong>","9":"<strong>0.77<\/strong>","10":"<strong>[0.71, 0.83]<\/strong>"},{"1":"Depression","2":"Gender","3":"<strong>0.91<\/strong>","4":"<strong>[0.85, 0.98]<\/strong>","5":"0.99","6":"[0.93, 1.06]","7":"<strong>0.74<\/strong>","8":"<strong>[0.67, 0.82]<\/strong>","9":"<strong>1.16<\/strong>","10":"<strong>[1.08, 1.25]<\/strong>"},{"1":"Depression","2":"Parental Education","3":"<strong>0.89<\/strong>","4":"<strong>[0.83, 0.96]<\/strong>","5":"0.97","6":"[0.91, 1.04]","7":"<strong>0.76<\/strong>","8":"<strong>[0.68, 0.84]<\/strong>","9":"<strong>1.12<\/strong>","10":"<strong>[1.05, 1.21]<\/strong>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Kabling the Table  

And again, for our spanned columns, we'll take advantage of our `O_names` object to create the vector in advance:


```{.r .code-style}
heads <- rep(2, 5)
heads
```

```
## [1] 2 2 2 2 2
```

```{.r .code-style}
names(heads) <- c(" ", O_names$new)
heads
```

```
##                      Move in with Partner              Married 
##                    2                    2                    2 
##             Divorced     Birth of a Child 
##                    2                    2
```

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Kabling the Table  
Now, this will proceed as before, with just a few small tweaks to the `align` and `col.names` arguments to account for the additional Covariate column.


```{.r .code-style}
tidy3 %>%
  kable(., escape = F,
        align = c("r", "r", rep("c", 8)),
        col.names = c("Trait", "Covariate", rep(c("OR", "CI"), times = 4)),
        caption = "<strong>Table 3</strong><br><em>Estimated Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  add_header_above(heads) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 3</strong><br><em>Estimated Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="2"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Move in with Partner</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Married</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Divorced</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:right;"> Covariate </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.29</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.40]</strong> </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.38, 1.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>1.20</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.33]</strong> </td>
   <td style="text-align:center;"> <strong>1.17</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.27]</strong> </td>
   <td style="text-align:center;"> <strong>1.51</strong> </td>
   <td style="text-align:center;"> <strong>[1.32, 1.73]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.90, 1.09] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.42, 1.75]</strong> </td>
   <td style="text-align:center;"> <strong>1.53</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.66]</strong> </td>
   <td style="text-align:center;"> <strong>1.70</strong> </td>
   <td style="text-align:center;"> <strong>[1.48, 1.95]</strong> </td>
   <td style="text-align:center;"> <strong>1.39</strong> </td>
   <td style="text-align:center;"> <strong>[1.26, 1.53]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.34</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.48]</strong> </td>
   <td style="text-align:center;"> <strong>1.30</strong> </td>
   <td style="text-align:center;"> <strong>[1.20, 1.41]</strong> </td>
   <td style="text-align:center;"> <strong>1.59</strong> </td>
   <td style="text-align:center;"> <strong>[1.39, 1.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Negative Affect </td>
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.22, 1.49]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.43]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.84]</strong> </td>
   <td style="text-align:center;"> <strong>1.12</strong> </td>
   <td style="text-align:center;"> <strong>[1.02, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.36</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.11 </td>
   <td style="text-align:center;"> [0.99, 1.24] </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.23, 1.48]</strong> </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.89]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.44, 1.80]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.03, 1.31]</strong> </td>
   <td style="text-align:center;"> <strong>1.42</strong> </td>
   <td style="text-align:center;"> <strong>[1.29, 1.56]</strong> </td>
   <td style="text-align:center;"> <strong>0.79</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.91]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.44, 1.81]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.96</strong> </td>
   <td style="text-align:center;"> <strong>[1.76, 2.19]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Positive Affect </td>
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.31</strong> </td>
   <td style="text-align:center;"> <strong>[1.17, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.55</strong> </td>
   <td style="text-align:center;"> <strong>[1.42, 1.70]</strong> </td>
   <td style="text-align:center;"> <strong>0.87</strong> </td>
   <td style="text-align:center;"> <strong>[0.75, 1.00]</strong> </td>
   <td style="text-align:center;"> <strong>1.87</strong> </td>
   <td style="text-align:center;"> <strong>[1.68, 2.09]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.94, 1.22] </td>
   <td style="text-align:center;"> <strong>0.90</strong> </td>
   <td style="text-align:center;"> <strong>[0.83, 0.99]</strong> </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.82, 1.15] </td>
   <td style="text-align:center;"> <strong>1.18</strong> </td>
   <td style="text-align:center;"> <strong>[1.05, 1.33]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.83, 1.07] </td>
   <td style="text-align:center;"> <strong>0.79</strong> </td>
   <td style="text-align:center;"> <strong>[0.73, 0.87]</strong> </td>
   <td style="text-align:center;"> 0.92 </td>
   <td style="text-align:center;"> [0.78, 1.09] </td>
   <td style="text-align:center;"> 0.98 </td>
   <td style="text-align:center;"> [0.87, 1.10] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.98, 1.23] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Self-Esteem </td>
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.87, 1.12] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.93]</strong> </td>
   <td style="text-align:center;"> 0.96 </td>
   <td style="text-align:center;"> [0.82, 1.15] </td>
   <td style="text-align:center;"> 1.08 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>1.23</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.41]</strong> </td>
   <td style="text-align:center;"> 1.08 </td>
   <td style="text-align:center;"> [0.97, 1.20] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.85, 1.19] </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.18, 1.50]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.17, 1.51]</strong> </td>
   <td style="text-align:center;"> 1.11 </td>
   <td style="text-align:center;"> [1.00, 1.24] </td>
   <td style="text-align:center;"> 1.04 </td>
   <td style="text-align:center;"> [0.87, 1.24] </td>
   <td style="text-align:center;"> <strong>1.31</strong> </td>
   <td style="text-align:center;"> <strong>[1.16, 1.48]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.89]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Optimism </td>
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.50</strong> </td>
   <td style="text-align:center;"> <strong>[1.32, 1.70]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.47]</strong> </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.90, 1.27] </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.45, 1.83]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>0.88</strong> </td>
   <td style="text-align:center;"> <strong>[0.81, 0.95]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.92, 1.06] </td>
   <td style="text-align:center;"> <strong>0.73</strong> </td>
   <td style="text-align:center;"> <strong>[0.66, 0.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.25]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>0.63</strong> </td>
   <td style="text-align:center;"> <strong>[0.59, 0.69]</strong> </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.69, 0.80]</strong> </td>
   <td style="text-align:center;"> <strong>0.63</strong> </td>
   <td style="text-align:center;"> <strong>[0.56, 0.70]</strong> </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.71, 0.83]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.25]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;"> Depression </td>
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>0.89</strong> </td>
   <td style="text-align:center;"> <strong>[0.83, 0.96]</strong> </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.91, 1.04] </td>
   <td style="text-align:center;"> <strong>0.76</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.84]</strong> </td>
   <td style="text-align:center;"> <strong>1.12</strong> </td>
   <td style="text-align:center;"> <strong>[1.05, 1.21]</strong> </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>

So this looks pretty good, except that it's annoying how the Trait name is repeated five times. This is where we'll use `collapse_rows()`.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Covariates  
#### Kabling the Table  
This is where we'll use `collapse_rows()`.  


```{.r .code-style}
tidy3 %>%
  kable(., escape = F,
        align = c("r", "r", rep("c", 8)),
        col.names = c("Trait", "Covariate", rep(c("OR", "CI"), times = 4)),
        caption = "<strong>Table 3</strong><br><em>Estimated Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  collapse_rows(1, valign = "top") %>%
  add_header_above(heads) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 3</strong><br><em>Estimated Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="2"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Move in with Partner</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Married</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Divorced</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:right;"> Covariate </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Negative Affect </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.29</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.40]</strong> </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.38, 1.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>1.20</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.33]</strong> </td>
   <td style="text-align:center;"> <strong>1.17</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.27]</strong> </td>
   <td style="text-align:center;"> <strong>1.51</strong> </td>
   <td style="text-align:center;"> <strong>[1.32, 1.73]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.90, 1.09] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.42, 1.75]</strong> </td>
   <td style="text-align:center;"> <strong>1.53</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.66]</strong> </td>
   <td style="text-align:center;"> <strong>1.70</strong> </td>
   <td style="text-align:center;"> <strong>[1.48, 1.95]</strong> </td>
   <td style="text-align:center;"> <strong>1.39</strong> </td>
   <td style="text-align:center;"> <strong>[1.26, 1.53]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.34</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.48]</strong> </td>
   <td style="text-align:center;"> <strong>1.30</strong> </td>
   <td style="text-align:center;"> <strong>[1.20, 1.41]</strong> </td>
   <td style="text-align:center;"> <strong>1.59</strong> </td>
   <td style="text-align:center;"> <strong>[1.39, 1.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.11</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.22]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.22, 1.49]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.43]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.84]</strong> </td>
   <td style="text-align:center;"> <strong>1.12</strong> </td>
   <td style="text-align:center;"> <strong>[1.02, 1.22]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Positive Affect </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.36</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.97</strong> </td>
   <td style="text-align:center;"> <strong>[1.77, 2.20]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.11 </td>
   <td style="text-align:center;"> [0.99, 1.24] </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.23, 1.48]</strong> </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.89]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.44, 1.80]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.03, 1.31]</strong> </td>
   <td style="text-align:center;"> <strong>1.42</strong> </td>
   <td style="text-align:center;"> <strong>[1.29, 1.56]</strong> </td>
   <td style="text-align:center;"> <strong>0.79</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.91]</strong> </td>
   <td style="text-align:center;"> <strong>1.61</strong> </td>
   <td style="text-align:center;"> <strong>[1.44, 1.81]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.35</strong> </td>
   <td style="text-align:center;"> <strong>[1.21, 1.52]</strong> </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.49, 1.78]</strong> </td>
   <td style="text-align:center;"> <strong>0.85</strong> </td>
   <td style="text-align:center;"> <strong>[0.74, 0.97]</strong> </td>
   <td style="text-align:center;"> <strong>1.96</strong> </td>
   <td style="text-align:center;"> <strong>[1.76, 2.19]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.31</strong> </td>
   <td style="text-align:center;"> <strong>[1.17, 1.47]</strong> </td>
   <td style="text-align:center;"> <strong>1.55</strong> </td>
   <td style="text-align:center;"> <strong>[1.42, 1.70]</strong> </td>
   <td style="text-align:center;"> <strong>0.87</strong> </td>
   <td style="text-align:center;"> <strong>[0.75, 1.00]</strong> </td>
   <td style="text-align:center;"> <strong>1.87</strong> </td>
   <td style="text-align:center;"> <strong>[1.68, 2.09]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Self-Esteem </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.94, 1.22] </td>
   <td style="text-align:center;"> <strong>0.90</strong> </td>
   <td style="text-align:center;"> <strong>[0.83, 0.99]</strong> </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.82, 1.15] </td>
   <td style="text-align:center;"> <strong>1.18</strong> </td>
   <td style="text-align:center;"> <strong>[1.05, 1.33]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.83, 1.07] </td>
   <td style="text-align:center;"> <strong>0.79</strong> </td>
   <td style="text-align:center;"> <strong>[0.73, 0.87]</strong> </td>
   <td style="text-align:center;"> 0.92 </td>
   <td style="text-align:center;"> [0.78, 1.09] </td>
   <td style="text-align:center;"> 0.98 </td>
   <td style="text-align:center;"> [0.87, 1.10] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.88, 1.13] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.94]</strong> </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.81, 1.12] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.98, 1.23] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.87, 1.12] </td>
   <td style="text-align:center;"> <strong>0.86</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.93]</strong> </td>
   <td style="text-align:center;"> 0.96 </td>
   <td style="text-align:center;"> [0.82, 1.15] </td>
   <td style="text-align:center;"> 1.08 </td>
   <td style="text-align:center;"> [0.97, 1.22] </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Optimism </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.88]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>1.23</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.41]</strong> </td>
   <td style="text-align:center;"> 1.08 </td>
   <td style="text-align:center;"> [0.97, 1.20] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.85, 1.19] </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.18, 1.50]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.33</strong> </td>
   <td style="text-align:center;"> <strong>[1.17, 1.51]</strong> </td>
   <td style="text-align:center;"> 1.11 </td>
   <td style="text-align:center;"> [1.00, 1.24] </td>
   <td style="text-align:center;"> 1.04 </td>
   <td style="text-align:center;"> [0.87, 1.24] </td>
   <td style="text-align:center;"> <strong>1.31</strong> </td>
   <td style="text-align:center;"> <strong>[1.16, 1.48]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>1.58</strong> </td>
   <td style="text-align:center;"> <strong>[1.40, 1.79]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.46]</strong> </td>
   <td style="text-align:center;"> 1.10 </td>
   <td style="text-align:center;"> [0.93, 1.30] </td>
   <td style="text-align:center;"> <strong>1.68</strong> </td>
   <td style="text-align:center;"> <strong>[1.50, 1.89]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>1.50</strong> </td>
   <td style="text-align:center;"> <strong>[1.32, 1.70]</strong> </td>
   <td style="text-align:center;"> <strong>1.32</strong> </td>
   <td style="text-align:center;"> <strong>[1.19, 1.47]</strong> </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.90, 1.27] </td>
   <td style="text-align:center;"> <strong>1.63</strong> </td>
   <td style="text-align:center;"> <strong>[1.45, 1.83]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Depression </td>
   <td style="text-align:right;"> None </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.15</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.24]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>0.88</strong> </td>
   <td style="text-align:center;"> <strong>[0.81, 0.95]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.92, 1.06] </td>
   <td style="text-align:center;"> <strong>0.73</strong> </td>
   <td style="text-align:center;"> <strong>[0.66, 0.80]</strong> </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.07, 1.25]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>0.63</strong> </td>
   <td style="text-align:center;"> <strong>[0.59, 0.69]</strong> </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.69, 0.80]</strong> </td>
   <td style="text-align:center;"> <strong>0.63</strong> </td>
   <td style="text-align:center;"> <strong>[0.56, 0.70]</strong> </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.71, 0.83]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender </td>
   <td style="text-align:center;"> <strong>0.91</strong> </td>
   <td style="text-align:center;"> <strong>[0.85, 0.98]</strong> </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.93, 1.06] </td>
   <td style="text-align:center;"> <strong>0.74</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.82]</strong> </td>
   <td style="text-align:center;"> <strong>1.16</strong> </td>
   <td style="text-align:center;"> <strong>[1.08, 1.25]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education </td>
   <td style="text-align:center;"> <strong>0.89</strong> </td>
   <td style="text-align:center;"> <strong>[0.83, 0.96]</strong> </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.91, 1.04] </td>
   <td style="text-align:center;"> <strong>0.76</strong> </td>
   <td style="text-align:center;"> <strong>[0.68, 0.84]</strong> </td>
   <td style="text-align:center;"> <strong>1.12</strong> </td>
   <td style="text-align:center;"> <strong>[1.05, 1.21]</strong> </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>

That's a pretty good-looking table!  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
All right, time to introduce a moderator!  

The procedure for this is going to be very close to above, with the main change being that the key term will no longer be p_value but p_value:moderator. 

But first, the set up  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Set up Data  
This time, we need to add a new grouping variable -- namely, the target moderators. We aren't going to include the "none" condition here, as we assume we've already tested and presented those results separately.    


```{.r .code-style}
gsoep_nested4 <- gsoep_long %>%
  group_by(Trait, Outcome, Covariate) %>%
  nest() %>%
  ungroup() %>%
  arrange(Outcome, Trait, Covariate)
gsoep_nested4
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["data"],"name":[4],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"age","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"gender","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"age","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"gender","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"age","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"OP","3":"age","4":"<tibble>"},{"1":"divorced","2":"OP","3":"gender","4":"<tibble>"},{"1":"divorced","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"PA","3":"age","4":"<tibble>"},{"1":"divorced","2":"PA","3":"gender","4":"<tibble>"},{"1":"divorced","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"divorced","2":"SE","3":"age","4":"<tibble>"},{"1":"divorced","2":"SE","3":"gender","4":"<tibble>"},{"1":"divorced","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"divorced","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"DEP","3":"age","4":"<tibble>"},{"1":"married","2":"DEP","3":"gender","4":"<tibble>"},{"1":"married","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"married","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"NegAff","3":"age","4":"<tibble>"},{"1":"married","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"married","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"married","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"OP","3":"age","4":"<tibble>"},{"1":"married","2":"OP","3":"gender","4":"<tibble>"},{"1":"married","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"married","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"PA","3":"age","4":"<tibble>"},{"1":"married","2":"PA","3":"gender","4":"<tibble>"},{"1":"married","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"married","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"married","2":"SE","3":"age","4":"<tibble>"},{"1":"married","2":"SE","3":"gender","4":"<tibble>"},{"1":"married","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"married","2":"SE","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Run the models  

```{.r .code-style}
factor_fun <- function(x){if(is.numeric(x)){diff(range(x, na.rm = T)) %in% 1:2 & length(unique(x)) <= 4} else{F}}

mod4_fun <- function(d, cov){
  d$o_value <- factor(d$o_value)
  d <- d %>% mutate_if(factor_fun, factor)
  if(cov == "none"){f <- formula(o_value ~ p_value)} else{f <- formula(o_value ~ p_value* c_value)}
  glm(f, data = d, family = binomial(link = "logit"))
}

gsoep_nested4 <- gsoep_nested4 %>%
  mutate(m = map2(data, Covariate, mod4_fun),
         tidy = map(m, ~tidy(., conf.int = T)))
gsoep_nested4
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Covariate"],"name":[3],"type":["chr"],"align":["left"]},{"label":["data"],"name":[4],"type":["list"],"align":["right"]},{"label":["m"],"name":[5],"type":["list"],"align":["right"]},{"label":["tidy"],"name":[6],"type":["list"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"divorced","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"married","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"<tibble>","5":"<S3: glm>","6":"<tibble>"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

Like before, we are going to have different numbers of rows. Again this is good becuase it means that the appropriate moderators and main effects were added.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Create the Table  

```{.r .code-style}
tidy4 <- gsoep_nested4 %>%
  select(Outcome, Trait, Moderator = Covariate, tidy) %>%
  unnest(tidy)
tidy4
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Outcome"],"name":[1],"type":["chr"],"align":["left"]},{"label":["Trait"],"name":[2],"type":["chr"],"align":["left"]},{"label":["Moderator"],"name":[3],"type":["chr"],"align":["left"]},{"label":["term"],"name":[4],"type":["chr"],"align":["left"]},{"label":["estimate"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["std.error"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["statistic"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["p.value"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["conf.low"],"name":[9],"type":["dbl"],"align":["right"]},{"label":["conf.high"],"name":[10],"type":["dbl"],"align":["right"]}],"data":[{"1":"chldbrth","2":"DEP","3":"age","4":"(Intercept)","5":"-4.155522e+00","6":"2.255928e-01","7":"-18.420452911","8":"9.004691e-76","9":"-4.605926e+00","10":"-3.721630693"},{"1":"chldbrth","2":"DEP","3":"age","4":"p_value","5":"1.715972e-01","6":"5.602958e-02","7":"3.062618519","8":"2.194096e-03","9":"6.303709e-02","10":"0.282670726"},{"1":"chldbrth","2":"DEP","3":"age","4":"c_value","5":"-7.254667e-02","6":"9.867058e-03","7":"-7.352411149","8":"1.946629e-13","9":"-9.205002e-02","10":"-0.053382275"},{"1":"chldbrth","2":"DEP","3":"age","4":"p_value:c_value","5":"1.572883e-03","6":"2.446635e-03","7":"0.642876117","8":"5.203045e-01","9":"-3.196888e-03","10":"0.006390689"},{"1":"chldbrth","2":"DEP","3":"gender","4":"(Intercept)","5":"-3.473870e+00","6":"2.280074e-01","7":"-15.235780662","8":"2.046678e-52","9":"-3.928186e+00","10":"-3.034087878"},{"1":"chldbrth","2":"DEP","3":"gender","4":"p_value","5":"1.276205e-01","6":"5.577798e-02","7":"2.288007993","8":"2.213706e-02","9":"1.943613e-02","10":"0.238154880"},{"1":"chldbrth","2":"DEP","3":"gender","4":"c_value1","5":"-4.754276e-02","6":"2.980698e-01","7":"-0.159502137","8":"8.732733e-01","9":"-6.299162e-01","10":"0.539025208"},{"1":"chldbrth","2":"DEP","3":"gender","4":"p_value:c_value1","5":"4.055252e-02","6":"7.399122e-02","7":"0.548072069","8":"5.836424e-01","9":"-1.048384e-01","10":"0.185287819"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-3.628746e+00","6":"1.766533e-01","7":"-20.541619480","8":"9.146149e-94","9":"-3.979476e+00","10":"-3.286905416"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"p_value","5":"1.538181e-01","6":"4.412546e-02","7":"3.485926307","8":"4.904361e-04","9":"6.800424e-02","10":"0.240997274"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"c_value1","5":"1.453782e+00","6":"3.704326e-01","7":"3.924551282","8":"8.689155e-05","9":"7.181855e-01","10":"2.171108369"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"c_value2","5":"3.934489e-01","6":"6.167772e-01","7":"0.637910837","8":"5.235317e-01","9":"-8.604230e-01","10":"1.561652099"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"p_value:c_value1","5":"-2.201377e-01","6":"9.243897e-02","7":"-2.381438453","8":"1.724517e-02","9":"-3.999677e-01","10":"-0.037458132"},{"1":"chldbrth","2":"DEP","3":"parEdu","4":"p_value:c_value2","5":"1.161816e-01","6":"1.502820e-01","7":"0.773090824","8":"4.394686e-01","9":"-1.715426e-01","10":"0.418468389"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-2.205545e+00","6":"1.647251e-01","7":"-13.389242354","8":"6.989004e-41","9":"-2.531908e+00","10":"-1.886022010"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"p_value","5":"-2.337893e-01","6":"4.321642e-02","7":"-5.409734178","8":"6.311837e-08","9":"-3.180658e-01","10":"-0.148624814"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"c_value","5":"1.504421e+00","6":"2.045885e-01","7":"7.353398511","8":"1.932297e-13","9":"1.111287e+00","10":"1.913124024"},{"1":"chldbrth","2":"DEP","3":"SRhealth","4":"p_value:c_value","5":"-1.271124e-01","6":"5.164886e-02","7":"-2.461087485","8":"1.385166e-02","9":"-2.300518e-01","10":"-0.027640436"},{"1":"chldbrth","2":"NegAff","3":"age","4":"(Intercept)","5":"-3.912671e+00","6":"2.007280e-01","7":"-19.492406111","8":"1.273500e-84","9":"-4.312088e+00","10":"-3.525333157"},{"1":"chldbrth","2":"NegAff","3":"age","4":"p_value","5":"1.037268e-02","6":"7.372480e-02","7":"0.140694602","8":"8.881112e-01","9":"-1.349260e-01","10":"0.154015827"},{"1":"chldbrth","2":"NegAff","3":"age","4":"c_value","5":"-6.639276e-02","6":"8.348896e-03","7":"-7.952279808","8":"1.831101e-15","9":"-8.291006e-02","10":"-0.050201148"},{"1":"chldbrth","2":"NegAff","3":"age","4":"p_value:c_value","5":"1.241633e-03","6":"3.061822e-03","7":"0.405520994","8":"6.850946e-01","9":"-4.753571e-03","10":"0.007239939"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"(Intercept)","5":"-3.604391e+00","6":"1.931745e-01","7":"-18.658725432","8":"1.072591e-77","9":"-3.986805e+00","10":"-3.229459597"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"p_value","5":"1.053085e-01","6":"7.342874e-02","7":"1.434159042","8":"1.515269e-01","9":"-3.976009e-02","10":"0.248130041"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"c_value1","5":"6.546312e-02","6":"2.664667e-01","7":"0.245670904","8":"8.059370e-01","9":"-4.568017e-01","10":"0.587923447"},{"1":"chldbrth","2":"NegAff","3":"gender","4":"p_value:c_value1","5":"-5.108993e-03","6":"9.664261e-02","7":"-0.052864815","8":"9.578396e-01","9":"-1.941300e-01","10":"0.184752892"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-3.629693e+00","6":"1.601037e-01","7":"-22.670892704","8":"8.681577e-114","9":"-3.946408e+00","10":"-3.318737716"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"p_value","5":"7.789528e-02","6":"5.787108e-02","7":"1.346013894","8":"1.782980e-01","9":"-3.615070e-02","10":"0.190731696"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"c_value1","5":"3.609904e-01","6":"3.179073e-01","7":"1.135521149","8":"2.561570e-01","9":"-2.675821e-01","10":"0.979106635"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"c_value2","5":"4.835088e-01","6":"5.402197e-01","7":"0.895022439","8":"3.707751e-01","9":"-6.015303e-01","10":"1.519452001"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"p_value:c_value1","5":"1.039867e-01","6":"1.137547e-01","7":"0.914130598","8":"3.606482e-01","9":"-1.199677e-01","10":"0.326124037"},{"1":"chldbrth","2":"NegAff","3":"parEdu","4":"p_value:c_value2","5":"8.916522e-02","6":"1.836468e-01","7":"0.485525562","8":"6.273036e-01","9":"-2.761487e-01","10":"0.445118672"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.501912e+00","6":"1.690222e-01","7":"-26.635031466","8":"2.668191e-156","9":"-4.836208e+00","10":"-4.173619810"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"p_value","5":"3.607629e-01","6":"5.561157e-02","7":"6.487190851","8":"8.745153e-11","9":"2.510807e-01","10":"0.469122401"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"c_value","5":"1.352415e+00","6":"1.988704e-01","7":"6.800483338","8":"1.042687e-11","9":"9.588363e-01","10":"1.738103815"},{"1":"chldbrth","2":"NegAff","3":"SRhealth","4":"p_value:c_value","5":"-9.003136e-02","6":"6.747205e-02","7":"-1.334350356","8":"1.820891e-01","9":"-2.192767e-01","10":"0.045039362"},{"1":"chldbrth","2":"OP","3":"age","4":"(Intercept)","5":"-5.045752e+00","6":"3.003053e-01","7":"-16.802071660","8":"2.356671e-63","9":"-5.650291e+00","10":"-4.472923331"},{"1":"chldbrth","2":"OP","3":"age","4":"p_value","5":"5.533096e-01","6":"9.336624e-02","7":"5.926228164","8":"3.099718e-09","9":"3.724054e-01","10":"0.738487312"},{"1":"chldbrth","2":"OP","3":"age","4":"c_value","5":"-1.116648e-01","6":"1.290620e-02","7":"-8.652025759","8":"5.059322e-18","9":"-1.372653e-01","10":"-0.086679715"},{"1":"chldbrth","2":"OP","3":"age","4":"p_value:c_value","5":"1.582754e-02","6":"4.009223e-03","7":"3.947782756","8":"7.887833e-05","9":"8.004600e-03","10":"0.023716084"},{"1":"chldbrth","2":"OP","3":"gender","4":"(Intercept)","5":"-4.276741e+00","6":"2.728885e-01","7":"-15.672117094","8":"2.346319e-55","9":"-4.825082e+00","10":"-3.754834017"},{"1":"chldbrth","2":"OP","3":"gender","4":"p_value","5":"4.896963e-01","6":"8.519060e-02","7":"5.748244108","8":"9.017499e-09","9":"3.247110e-01","10":"0.658819022"},{"1":"chldbrth","2":"OP","3":"gender","4":"c_value1","5":"-5.238368e-02","6":"3.729270e-01","7":"-0.140466299","8":"8.882916e-01","9":"-7.817000e-01","10":"0.681262734"},{"1":"chldbrth","2":"OP","3":"gender","4":"p_value:c_value1","5":"5.515143e-02","6":"1.166581e-01","7":"0.472761186","8":"6.363836e-01","9":"-1.739382e-01","10":"0.283517594"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.365057e+00","6":"2.220733e-01","7":"-19.655925732","8":"5.144190e-86","9":"-4.808981e+00","10":"-3.938178160"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"p_value","5":"5.006592e-01","6":"7.036020e-02","7":"7.115659164","8":"1.113792e-12","9":"3.639522e-01","10":"0.639851479"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"c_value1","5":"3.781081e-01","6":"5.175615e-01","7":"0.730556861","8":"4.650499e-01","9":"-6.600737e-01","10":"1.370930451"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"c_value2","5":"1.704598e+00","6":"6.987021e-01","7":"2.439663327","8":"1.470096e-02","9":"2.775677e-01","10":"3.024952010"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"p_value:c_value1","5":"3.742662e-02","6":"1.580565e-01","7":"0.236792636","8":"8.128177e-01","9":"-2.689157e-01","10":"0.351117124"},{"1":"chldbrth","2":"OP","3":"parEdu","4":"p_value:c_value2","5":"-2.589395e-01","6":"2.159605e-01","7":"-1.199013546","8":"2.305227e-01","9":"-6.749671e-01","10":"0.173769539"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-4.044367e+00","6":"2.196638e-01","7":"-18.411621137","8":"1.060016e-75","9":"-4.484453e+00","10":"-3.623034392"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"p_value","5":"3.677135e-01","6":"7.150492e-02","7":"5.142493014","8":"2.711165e-07","9":"2.287524e-01","10":"0.509132589"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"c_value","5":"1.670226e+00","6":"2.901131e-01","7":"5.757154165","8":"8.554378e-09","9":"1.112955e+00","10":"2.249131302"},{"1":"chldbrth","2":"OP","3":"SRhealth","4":"p_value:c_value","5":"-2.573017e-01","6":"9.052692e-02","7":"-2.842267059","8":"4.479395e-03","9":"-4.369920e-01","10":"-0.082494730"},{"1":"chldbrth","2":"PA","3":"age","4":"(Intercept)","5":"-6.208878e+00","6":"3.340246e-01","7":"-18.588083165","8":"4.012637e-77","9":"-6.878551e+00","10":"-5.569843610"},{"1":"chldbrth","2":"PA","3":"age","4":"p_value","5":"6.358444e-01","6":"8.523938e-02","7":"7.459514721","8":"8.684173e-14","9":"4.707586e-01","10":"0.804724872"},{"1":"chldbrth","2":"PA","3":"age","4":"c_value","5":"-9.483866e-02","6":"1.381582e-02","7":"-6.864497136","8":"6.672588e-12","9":"-1.221424e-01","10":"-0.068031073"},{"1":"chldbrth","2":"PA","3":"age","4":"p_value:c_value","5":"9.195912e-03","6":"3.506278e-03","7":"2.622699001","8":"8.723631e-03","9":"2.355012e-03","10":"0.016085510"},{"1":"chldbrth","2":"PA","3":"gender","4":"(Intercept)","5":"-5.342086e+00","6":"3.141916e-01","7":"-17.002637239","8":"7.850718e-65","9":"-5.971189e+00","10":"-4.739506453"},{"1":"chldbrth","2":"PA","3":"gender","4":"p_value","5":"5.483845e-01","6":"8.133479e-02","7":"6.742311871","8":"1.558860e-11","9":"3.907890e-01","10":"0.709628529"},{"1":"chldbrth","2":"PA","3":"gender","4":"c_value1","5":"-8.216598e-01","6":"4.375669e-01","7":"-1.877792444","8":"6.040956e-02","9":"-1.678623e+00","10":"0.037496202"},{"1":"chldbrth","2":"PA","3":"gender","4":"p_value:c_value1","5":"2.318405e-01","6":"1.112244e-01","7":"2.084439691","8":"3.712019e-02","9":"1.360133e-02","10":"0.449679287"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"(Intercept)","5":"-5.994961e+00","6":"2.665001e-01","7":"-22.495151530","8":"4.629976e-112","9":"-6.526584e+00","10":"-5.481718117"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"p_value","5":"7.012184e-01","6":"6.775238e-02","7":"10.349723291","8":"4.196970e-25","9":"5.696605e-01","10":"0.835287218"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"c_value1","5":"8.233667e-01","6":"5.482976e-01","7":"1.501678496","8":"1.331802e-01","9":"-2.705454e-01","10":"1.879831056"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"c_value2","5":"3.349030e+00","6":"7.271285e-01","7":"4.605830093","8":"4.108237e-06","9":"1.857310e+00","10":"4.715462690"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"p_value:c_value1","5":"-7.612137e-02","6":"1.370010e-01","7":"-0.555626505","8":"5.784662e-01","9":"-3.419906e-01","10":"0.195164421"},{"1":"chldbrth","2":"PA","3":"parEdu","4":"p_value:c_value2","5":"-7.131961e-01","6":"1.977123e-01","7":"-3.607241288","8":"3.094699e-04","9":"-1.093120e+00","10":"-0.316647943"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-5.352646e+00","6":"2.490410e-01","7":"-21.493025971","8":"1.809271e-102","9":"-5.850889e+00","10":"-4.874518320"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"p_value","5":"5.074184e-01","6":"6.542371e-02","7":"7.755879318","8":"8.773343e-15","9":"3.805599e-01","10":"0.637016769"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"c_value","5":"1.162007e+00","6":"3.235731e-01","7":"3.591173445","8":"3.291925e-04","9":"5.422129e-01","10":"1.808658786"},{"1":"chldbrth","2":"PA","3":"SRhealth","4":"p_value:c_value","5":"-8.129120e-02","6":"8.270129e-02","7":"-0.982949558","8":"3.256323e-01","9":"-2.459335e-01","10":"0.077725861"},{"1":"chldbrth","2":"SE","3":"age","4":"(Intercept)","5":"-5.126587e+00","6":"5.719031e-01","7":"-8.964082807","8":"3.128738e-19","9":"-6.314676e+00","10":"-4.072731074"},{"1":"chldbrth","2":"SE","3":"age","4":"p_value","5":"1.547911e-01","6":"9.614942e-02","7":"1.609901162","8":"1.074194e-01","9":"-2.617346e-02","10":"0.350815109"},{"1":"chldbrth","2":"SE","3":"age","4":"c_value","5":"-6.186018e-02","6":"2.209432e-02","7":"-2.799822816","8":"5.113066e-03","9":"-1.062605e-01","10":"-0.019739043"},{"1":"chldbrth","2":"SE","3":"age","4":"p_value:c_value","5":"-5.202535e-04","6":"3.730536e-03","7":"-0.139458114","8":"8.890882e-01","9":"-7.704460e-03","10":"0.006895006"},{"1":"chldbrth","2":"SE","3":"gender","4":"(Intercept)","5":"-5.443524e+00","6":"6.421563e-01","7":"-8.476946660","8":"2.311809e-17","9":"-6.773611e+00","10":"-4.252857501"},{"1":"chldbrth","2":"SE","3":"gender","4":"p_value","5":"2.904982e-01","6":"1.050208e-01","7":"2.766102029","8":"5.673078e-03","9":"9.268078e-02","10":"0.504895544"},{"1":"chldbrth","2":"SE","3":"gender","4":"c_value1","5":"2.017033e+00","6":"7.556707e-01","7":"2.669195445","8":"7.603319e-03","9":"5.726816e-01","10":"3.543151563"},{"1":"chldbrth","2":"SE","3":"gender","4":"p_value:c_value1","5":"-3.088626e-01","6":"1.263103e-01","7":"-2.445267757","8":"1.447446e-02","9":"-5.614905e-01","10":"-0.065506352"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.459350e+00","6":"4.382858e-01","7":"-10.174525891","8":"2.576516e-24","9":"-5.357292e+00","10":"-3.637242510"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"p_value","5":"1.157529e-01","6":"7.423624e-02","7":"1.559250478","8":"1.189371e-01","9":"-2.559204e-02","10":"0.265716660"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"c_value1","5":"1.373328e+00","6":"7.425118e-01","7":"1.849570367","8":"6.437550e-02","9":"-1.219917e-01","10":"2.800481119"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"c_value2","5":"1.578980e+00","6":"1.480313e+00","7":"1.066652843","8":"2.861286e-01","9":"-1.697006e+00","10":"4.201227507"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"p_value:c_value1","5":"-9.013337e-02","6":"1.279926e-01","7":"-0.704207828","8":"4.813034e-01","9":"-3.375948e-01","10":"0.165376439"},{"1":"chldbrth","2":"SE","3":"parEdu","4":"p_value:c_value2","5":"-1.924348e-01","6":"2.608949e-01","7":"-0.737595286","8":"4.607604e-01","9":"-6.783466e-01","10":"0.358256577"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-3.731054e+00","6":"3.757843e-01","7":"-9.928712719","8":"3.122606e-23","9":"-4.502567e+00","10":"-3.026957344"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"p_value","5":"-1.922689e-02","6":"6.615517e-02","7":"-0.290633203","8":"7.713319e-01","9":"-1.454102e-01","10":"0.114249620"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"c_value","5":"1.064101e+00","6":"5.096987e-01","7":"2.087705200","8":"3.682444e-02","9":"1.097030e-01","10":"2.098535625"},{"1":"chldbrth","2":"SE","3":"SRhealth","4":"p_value:c_value","5":"-9.469682e-03","6":"8.691861e-02","7":"-0.108948852","8":"9.132431e-01","9":"-1.848800e-01","10":"0.154148771"},{"1":"divorced","2":"DEP","3":"age","4":"(Intercept)","5":"-2.865386e+00","6":"2.056514e-01","7":"-13.933219562","8":"3.980020e-44","9":"-3.276111e+00","10":"-2.469663122"},{"1":"divorced","2":"DEP","3":"age","4":"p_value","5":"-3.060024e-01","6":"5.425651e-02","7":"-5.639919994","8":"1.701292e-08","9":"-4.113950e-01","10":"-0.198644703"},{"1":"divorced","2":"DEP","3":"age","4":"c_value","5":"-3.215528e-02","6":"1.042621e-02","7":"-3.084081018","8":"2.041820e-03","9":"-5.276195e-02","10":"-0.011917044"},{"1":"divorced","2":"DEP","3":"age","4":"p_value:c_value","5":"2.359257e-03","6":"2.744527e-03","7":"0.859622426","8":"3.899972e-01","9":"-2.985031e-03","10":"0.007764705"},{"1":"divorced","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.367282e+00","6":"2.815677e-01","7":"-8.407505033","8":"4.188244e-17","9":"-2.931822e+00","10":"-1.827513901"},{"1":"divorced","2":"DEP","3":"gender","4":"p_value","5":"-4.087048e-01","6":"7.356378e-02","7":"-5.555788011","8":"2.763621e-08","9":"-5.512490e-01","10":"-0.262742465"},{"1":"divorced","2":"DEP","3":"gender","4":"c_value1","5":"-7.362956e-01","6":"3.824432e-01","7":"-1.925241667","8":"5.419913e-02","9":"-1.484275e+00","10":"0.015903956"},{"1":"divorced","2":"DEP","3":"gender","4":"p_value:c_value1","5":"1.931194e-01","6":"1.008643e-01","7":"1.914644552","8":"5.553785e-02","9":"-4.950355e-03","10":"0.390564505"},{"1":"divorced","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.931141e+00","6":"2.221582e-01","7":"-13.193936616","8":"9.508718e-40","9":"-3.374224e+00","10":"-2.503115787"},{"1":"divorced","2":"DEP","3":"parEdu","4":"p_value","5":"-2.655036e-01","6":"5.867262e-02","7":"-4.525170383","8":"6.034687e-06","9":"-3.795555e-01","10":"-0.149495993"},{"1":"divorced","2":"DEP","3":"parEdu","4":"c_value1","5":"1.423537e-01","6":"5.982344e-01","7":"0.237956329","8":"8.119150e-01","9":"-1.069746e+00","10":"1.279259087"},{"1":"divorced","2":"DEP","3":"parEdu","4":"c_value2","5":"9.415971e-01","6":"8.335526e-01","7":"1.129619371","8":"2.586366e-01","9":"-7.877468e-01","10":"2.496143506"},{"1":"divorced","2":"DEP","3":"parEdu","4":"p_value:c_value1","5":"-1.416716e-02","6":"1.538838e-01","7":"-0.092063982","8":"9.266472e-01","9":"-3.106892e-01","10":"0.293304393"},{"1":"divorced","2":"DEP","3":"parEdu","4":"p_value:c_value2","5":"-1.606797e-01","6":"2.191155e-01","7":"-0.733310657","8":"4.633690e-01","9":"-5.808615e-01","10":"0.281843278"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-2.154398e+00","6":"2.103419e-01","7":"-10.242362690","8":"1.280700e-24","9":"-2.571942e+00","10":"-1.747236187"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"p_value","5":"-4.679866e-01","6":"5.623233e-02","7":"-8.322375960","8":"8.621733e-17","9":"-5.776646e-01","10":"-0.357193035"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"c_value","5":"7.109511e-01","6":"2.262148e-01","7":"3.142813482","8":"1.673324e-03","9":"2.788302e-01","10":"1.165586072"},{"1":"divorced","2":"DEP","3":"SRhealth","4":"p_value:c_value","5":"-9.628481e-02","6":"6.124405e-02","7":"-1.572149477","8":"1.159159e-01","9":"-2.190867e-01","10":"0.020926247"},{"1":"divorced","2":"NegAff","3":"age","4":"(Intercept)","5":"-5.547775e+00","6":"2.273157e-01","7":"-24.405597479","8":"1.491492e-131","9":"-6.000355e+00","10":"-5.108975979"},{"1":"divorced","2":"NegAff","3":"age","4":"p_value","5":"4.309794e-01","6":"7.533215e-02","7":"5.721055741","8":"1.058642e-08","9":"2.823364e-01","10":"0.577721253"},{"1":"divorced","2":"NegAff","3":"age","4":"c_value","5":"-2.554612e-02","6":"1.135074e-02","7":"-2.250611455","8":"2.441016e-02","9":"-4.792778e-02","10":"-0.003500046"},{"1":"divorced","2":"NegAff","3":"age","4":"p_value:c_value","5":"1.966484e-03","6":"3.758125e-03","7":"0.523262053","8":"6.007919e-01","9":"-5.374529e-03","10":"0.009328742"},{"1":"divorced","2":"NegAff","3":"gender","4":"(Intercept)","5":"-5.901207e+00","6":"3.046848e-01","7":"-19.368234776","8":"1.430794e-83","9":"-6.508265e+00","10":"-5.313418991"},{"1":"divorced","2":"NegAff","3":"gender","4":"p_value","5":"6.089873e-01","6":"1.015372e-01","7":"5.997676400","8":"2.001609e-09","9":"4.080463e-01","10":"0.806299436"},{"1":"divorced","2":"NegAff","3":"gender","4":"c_value1","5":"6.799289e-01","6":"4.201317e-01","7":"1.618371074","8":"1.055827e-01","9":"-1.435514e-01","10":"1.504101214"},{"1":"divorced","2":"NegAff","3":"gender","4":"p_value:c_value1","5":"-2.637337e-01","6":"1.376140e-01","7":"-1.916473434","8":"5.530486e-02","9":"-5.329751e-01","10":"0.006622716"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-5.444308e+00","6":"2.393965e-01","7":"-22.741805442","8":"1.729656e-114","9":"-5.920258e+00","10":"-4.981627222"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"p_value","5":"4.320328e-01","6":"7.874627e-02","7":"5.486391128","8":"4.102281e-08","9":"2.767734e-01","10":"0.585541992"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"c_value1","5":"-3.632271e-01","6":"6.092166e-01","7":"-0.596220025","8":"5.510282e-01","9":"-1.588525e+00","10":"0.802928349"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"c_value2","5":"-7.277292e-01","6":"9.046018e-01","7":"-0.804474625","8":"4.211229e-01","9":"-2.595380e+00","10":"0.969519535"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"p_value:c_value1","5":"1.197276e-01","6":"1.969188e-01","7":"0.608005038","8":"5.431841e-01","9":"-2.694830e-01","10":"0.503478856"},{"1":"divorced","2":"NegAff","3":"parEdu","4":"p_value:c_value2","5":"3.299791e-01","6":"2.644231e-01","7":"1.247921216","8":"2.120599e-01","9":"-1.959021e-01","10":"0.847130384"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-5.764332e+00","6":"2.213874e-01","7":"-26.037303460","8":"1.873488e-149","9":"-6.204135e+00","10":"-5.336116416"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"p_value","5":"5.325402e-01","6":"7.090942e-02","7":"7.510147193","8":"5.906092e-14","9":"3.929992e-01","10":"0.671023614"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"c_value","5":"5.316824e-01","6":"2.769009e-01","7":"1.920118336","8":"5.484295e-02","9":"-1.802026e-02","10":"1.066552096"},{"1":"divorced","2":"NegAff","3":"SRhealth","4":"p_value:c_value","5":"-6.354322e-02","6":"8.541776e-02","7":"-0.743911112","8":"4.569303e-01","9":"-2.262356e-01","10":"0.108310573"},{"1":"divorced","2":"OP","3":"age","4":"(Intercept)","5":"-4.166071e+00","6":"2.849239e-01","7":"-14.621698955","8":"2.042393e-48","9":"-4.741446e+00","10":"-3.623633415"},{"1":"divorced","2":"OP","3":"age","4":"p_value","5":"7.861650e-02","6":"9.611072e-02","7":"0.817978420","8":"4.133695e-01","9":"-1.083043e-01","10":"0.268726620"},{"1":"divorced","2":"OP","3":"age","4":"c_value","5":"-4.933665e-02","6":"1.480057e-02","7":"-3.333429775","8":"8.578232e-04","9":"-7.857770e-02","10":"-0.020631469"},{"1":"divorced","2":"OP","3":"age","4":"p_value:c_value","5":"9.326369e-03","6":"4.916244e-03","7":"1.897051555","8":"5.782113e-02","9":"-2.574817e-04","10":"0.018978821"},{"1":"divorced","2":"OP","3":"gender","4":"(Intercept)","5":"-4.032937e+00","6":"3.641365e-01","7":"-11.075342843","8":"1.652465e-28","9":"-4.772549e+00","10":"-3.343528222"},{"1":"divorced","2":"OP","3":"gender","4":"p_value","5":"6.781837e-02","6":"1.215104e-01","7":"0.558127940","8":"5.767570e-01","9":"-1.677650e-01","10":"0.309086006"},{"1":"divorced","2":"OP","3":"gender","4":"c_value1","5":"-1.454686e-01","6":"5.104719e-01","7":"-0.284968931","8":"7.756679e-01","9":"-1.145824e+00","10":"0.858555156"},{"1":"divorced","2":"OP","3":"gender","4":"p_value:c_value1","5":"5.870705e-02","6":"1.699999e-01","7":"0.345335794","8":"7.298419e-01","9":"-2.749777e-01","10":"0.391855867"},{"1":"divorced","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.065945e+00","6":"2.909193e-01","7":"-13.976192289","8":"2.178347e-44","9":"-4.651993e+00","10":"-3.510786357"},{"1":"divorced","2":"OP","3":"parEdu","4":"p_value","5":"7.615416e-02","6":"9.865383e-02","7":"0.771933102","8":"4.401541e-01","9":"-1.157910e-01","10":"0.271201557"},{"1":"divorced","2":"OP","3":"parEdu","4":"c_value1","5":"8.828916e-01","6":"7.970587e-01","7":"1.107687144","8":"2.679970e-01","9":"-7.569171e-01","10":"2.377621849"},{"1":"divorced","2":"OP","3":"parEdu","4":"c_value2","5":"-1.512400e+00","6":"1.467743e+00","7":"-1.030425289","8":"3.028104e-01","9":"-4.743190e+00","10":"1.080045894"},{"1":"divorced","2":"OP","3":"parEdu","4":"p_value:c_value1","5":"-3.068094e-01","6":"2.665383e-01","7":"-1.151089397","8":"2.496955e-01","9":"-8.228030e-01","10":"0.224324625"},{"1":"divorced","2":"OP","3":"parEdu","4":"p_value:c_value2","5":"5.478101e-01","6":"4.292783e-01","7":"1.276118775","8":"2.019135e-01","9":"-2.472338e-01","10":"1.454264901"},{"1":"divorced","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.957689e+00","6":"2.664838e-01","7":"-14.851520350","8":"6.800861e-50","9":"-4.491705e+00","10":"-3.446592947"},{"1":"divorced","2":"OP","3":"SRhealth","4":"p_value","5":"3.840728e-02","6":"9.086026e-02","7":"0.422707118","8":"6.725090e-01","9":"-1.388556e-01","10":"0.217502087"},{"1":"divorced","2":"OP","3":"SRhealth","4":"c_value","5":"1.494762e-01","6":"3.119758e-01","7":"0.479127606","8":"6.318479e-01","9":"-4.407781e-01","10":"0.782029029"},{"1":"divorced","2":"OP","3":"SRhealth","4":"p_value:c_value","5":"1.360953e-02","6":"1.058047e-01","7":"0.128628737","8":"8.976514e-01","9":"-1.996350e-01","10":"0.214818315"},{"1":"divorced","2":"PA","3":"age","4":"(Intercept)","5":"-3.712110e+00","6":"2.657187e-01","7":"-13.970070669","8":"2.373910e-44","9":"-4.248634e+00","10":"-3.206589952"},{"1":"divorced","2":"PA","3":"age","4":"p_value","5":"-1.892755e-01","6":"7.695967e-02","7":"-2.459411210","8":"1.391651e-02","9":"-3.382424e-01","10":"-0.036486356"},{"1":"divorced","2":"PA","3":"age","4":"c_value","5":"-5.225284e-02","6":"1.347748e-02","7":"-3.877048787","8":"1.057312e-04","9":"-7.893085e-02","10":"-0.026172758"},{"1":"divorced","2":"PA","3":"age","4":"p_value:c_value","5":"8.406049e-03","6":"3.854244e-03","7":"2.180985055","8":"2.918452e-02","9":"9.193245e-04","10":"0.015997128"},{"1":"divorced","2":"PA","3":"gender","4":"(Intercept)","5":"-3.498341e+00","6":"3.502072e-01","7":"-9.989343947","8":"1.697011e-23","9":"-4.209525e+00","10":"-2.835836952"},{"1":"divorced","2":"PA","3":"gender","4":"p_value","5":"-2.333143e-01","6":"1.020227e-01","7":"-2.286887598","8":"2.220238e-02","9":"-4.302728e-01","10":"-0.030132045"},{"1":"divorced","2":"PA","3":"gender","4":"c_value1","5":"-3.468050e-01","6":"4.804937e-01","7":"-0.721768170","8":"4.704370e-01","9":"-1.286846e+00","10":"0.599404001"},{"1":"divorced","2":"PA","3":"gender","4":"p_value:c_value1","5":"1.195648e-01","6":"1.382120e-01","7":"0.865082717","8":"3.869934e-01","9":"-1.519654e-01","10":"0.390074754"},{"1":"divorced","2":"PA","3":"parEdu","4":"(Intercept)","5":"-3.978469e+00","6":"2.843620e-01","7":"-13.990858914","8":"1.772581e-44","9":"-4.551351e+00","10":"-3.436308766"},{"1":"divorced","2":"PA","3":"parEdu","4":"p_value","5":"-8.250835e-02","6":"8.094579e-02","7":"-1.019303833","8":"3.080587e-01","9":"-2.392514e-01","10":"0.078135671"},{"1":"divorced","2":"PA","3":"parEdu","4":"c_value1","5":"8.074773e-02","6":"7.666230e-01","7":"0.105329125","8":"9.161147e-01","9":"-1.497185e+00","10":"1.514066626"},{"1":"divorced","2":"PA","3":"parEdu","4":"c_value2","5":"2.762233e+00","6":"7.933424e-01","7":"3.481766390","8":"4.981180e-04","9":"1.098971e+00","10":"4.238230692"},{"1":"divorced","2":"PA","3":"parEdu","4":"p_value:c_value1","5":"-2.288596e-02","6":"2.093934e-01","7":"-0.109296468","8":"9.129673e-01","9":"-4.240716e-01","10":"0.397542269"},{"1":"divorced","2":"PA","3":"parEdu","4":"p_value:c_value2","5":"-7.512843e-01","6":"2.572047e-01","7":"-2.920958701","8":"3.489561e-03","9":"-1.255821e+00","10":"-0.239784526"},{"1":"divorced","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-3.463609e+00","6":"2.508440e-01","7":"-13.807819125","8":"2.286495e-43","9":"-3.966650e+00","10":"-2.983157877"},{"1":"divorced","2":"PA","3":"SRhealth","4":"p_value","5":"-2.390116e-01","6":"7.291619e-02","7":"-3.277894739","8":"1.045844e-03","9":"-3.806252e-01","10":"-0.094743379"},{"1":"divorced","2":"PA","3":"SRhealth","4":"c_value","5":"2.305549e-01","6":"2.976702e-01","7":"0.774531263","8":"4.386167e-01","9":"-3.337871e-01","10":"0.832303884"},{"1":"divorced","2":"PA","3":"SRhealth","4":"p_value:c_value","5":"1.107420e-02","6":"8.768830e-02","7":"0.126290552","8":"8.995019e-01","9":"-1.654599e-01","10":"0.177857623"},{"1":"divorced","2":"SE","3":"age","4":"(Intercept)","5":"-4.637431e+00","6":"5.486187e-01","7":"-8.452921615","8":"2.841024e-17","9":"-5.788873e+00","10":"-3.632760167"},{"1":"divorced","2":"SE","3":"age","4":"p_value","5":"-2.123933e-02","6":"9.513053e-02","7":"-0.223265152","8":"8.233291e-01","9":"-2.001382e-01","10":"0.173605874"},{"1":"divorced","2":"SE","3":"age","4":"c_value","5":"-2.718237e-02","6":"2.558859e-02","7":"-1.062284866","8":"2.881064e-01","9":"-7.766369e-02","10":"0.022419153"},{"1":"divorced","2":"SE","3":"age","4":"p_value:c_value","5":"1.185839e-03","6":"4.469693e-03","7":"0.265306477","8":"7.907734e-01","9":"-7.538738e-03","10":"0.009905077"},{"1":"divorced","2":"SE","3":"gender","4":"(Intercept)","5":"-3.914707e+00","6":"6.821991e-01","7":"-5.738364834","8":"9.559501e-09","9":"-5.360224e+00","10":"-2.677604584"},{"1":"divorced","2":"SE","3":"gender","4":"p_value","5":"-1.329941e-01","6":"1.211221e-01","7":"-1.098017128","8":"2.721970e-01","9":"-3.597597e-01","10":"0.116363825"},{"1":"divorced","2":"SE","3":"gender","4":"c_value1","5":"-7.867655e-01","6":"9.514511e-01","7":"-0.826911092","8":"4.082875e-01","9":"-2.657154e+00","10":"1.099266255"},{"1":"divorced","2":"SE","3":"gender","4":"p_value:c_value1","5":"1.411823e-01","6":"1.680140e-01","7":"0.840300917","8":"4.007397e-01","9":"-1.903721e-01","10":"0.470546820"},{"1":"divorced","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.423640e+00","6":"5.553800e-01","7":"-7.965068930","8":"1.651317e-15","9":"-5.581234e+00","10":"-3.399873717"},{"1":"divorced","2":"SE","3":"parEdu","4":"p_value","5":"-4.175211e-02","6":"9.760395e-02","7":"-0.427770755","8":"6.688180e-01","9":"-2.262042e-01","10":"0.157085122"},{"1":"divorced","2":"SE","3":"parEdu","4":"c_value1","5":"8.353891e-01","6":"1.159572e+00","7":"0.720429077","8":"4.712609e-01","9":"-1.640471e+00","10":"2.963616034"},{"1":"divorced","2":"SE","3":"parEdu","4":"c_value2","5":"-9.818923e+01","6":"2.812559e+03","7":"-0.034910998","8":"9.721507e-01","9":"-1.110321e+03","10":"6.399880915"},{"1":"divorced","2":"SE","3":"parEdu","4":"p_value:c_value1","5":"-1.298855e-01","6":"2.107022e-01","7":"-0.616440956","8":"5.376035e-01","9":"-5.277139e-01","10":"0.304962299"},{"1":"divorced","2":"SE","3":"parEdu","4":"p_value:c_value2","5":"1.418489e+01","6":"4.017942e+02","7":"0.035303864","8":"9.718374e-01","9":"-4.159542e+01","10":"NA"},{"1":"divorced","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-4.261216e+00","6":"4.909634e-01","7":"-8.679294853","8":"3.982301e-18","9":"-5.274503e+00","10":"-3.347211805"},{"1":"divorced","2":"SE","3":"SRhealth","4":"p_value","5":"-8.090093e-02","6":"8.712359e-02","7":"-0.928576627","8":"3.531085e-01","9":"-2.468236e-01","10":"0.095174725"},{"1":"divorced","2":"SE","3":"SRhealth","4":"c_value","5":"-3.877310e-01","6":"5.539990e-01","7":"-0.699876757","8":"4.840043e-01","9":"-1.392719e+00","10":"0.779966284"},{"1":"divorced","2":"SE","3":"SRhealth","4":"p_value:c_value","5":"1.222857e-01","6":"9.873707e-02","7":"1.238498056","8":"2.155314e-01","9":"-8.428517e-02","10":"0.302180046"},{"1":"married","2":"DEP","3":"age","4":"(Intercept)","5":"-3.427237e+00","6":"1.843114e-01","7":"-18.594820144","8":"3.538979e-77","9":"-3.794091e+00","10":"-3.071535807"},{"1":"married","2":"DEP","3":"age","4":"p_value","5":"6.056195e-02","6":"4.617629e-02","7":"1.311537881","8":"1.896761e-01","9":"-2.913809e-02","10":"0.151887792"},{"1":"married","2":"DEP","3":"age","4":"c_value","5":"-7.636151e-02","6":"8.448927e-03","7":"-9.038012841","8":"1.595456e-19","9":"-9.304585e-02","10":"-0.059931255"},{"1":"married","2":"DEP","3":"age","4":"p_value:c_value","5":"5.247833e-03","6":"2.115196e-03","7":"2.481014732","8":"1.310090e-02","9":"1.120273e-03","10":"0.009410162"},{"1":"married","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.666782e+00","6":"1.941298e-01","7":"-13.737110167","8":"6.085678e-43","9":"-3.052523e+00","10":"-2.291386081"},{"1":"married","2":"DEP","3":"gender","4":"p_value","5":"-2.339328e-02","6":"4.828131e-02","7":"-0.484520346","8":"6.280166e-01","9":"-1.172364e-01","10":"0.072060269"},{"1":"married","2":"DEP","3":"gender","4":"c_value1","5":"-1.124818e-01","6":"2.584157e-01","7":"-0.435274809","8":"6.633630e-01","9":"-6.179101e-01","10":"0.395332574"},{"1":"married","2":"DEP","3":"gender","4":"p_value:c_value1","5":"2.970348e-02","6":"6.538667e-02","7":"0.454274326","8":"6.496314e-01","9":"-9.865816e-02","10":"0.157692883"},{"1":"married","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.743441e+00","6":"1.493634e-01","7":"-18.367556147","8":"2.389306e-75","9":"-3.039303e+00","10":"-2.453723603"},{"1":"married","2":"DEP","3":"parEdu","4":"p_value","5":"-1.675295e-02","6":"3.811127e-02","7":"-0.439580055","8":"6.602413e-01","9":"-9.101871e-02","10":"0.058395502"},{"1":"married","2":"DEP","3":"parEdu","4":"c_value1","5":"7.878125e-01","6":"3.349226e-01","7":"2.352222576","8":"1.866160e-02","9":"1.231279e-01","10":"1.436676813"},{"1":"married","2":"DEP","3":"parEdu","4":"c_value2","5":"1.549922e-01","6":"5.917925e-01","7":"0.261902895","8":"7.933963e-01","9":"-1.048989e+00","10":"1.275575863"},{"1":"married","2":"DEP","3":"parEdu","4":"p_value:c_value1","5":"-6.653493e-02","6":"8.411328e-02","7":"-0.791015786","8":"4.289348e-01","9":"-2.302552e-01","10":"0.099591095"},{"1":"married","2":"DEP","3":"parEdu","4":"p_value:c_value2","5":"6.173763e-02","6":"1.472406e-01","7":"0.419297592","8":"6.749987e-01","9":"-2.207951e-01","10":"0.357418300"},{"1":"married","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-1.748466e+00","6":"1.393191e-01","7":"-12.550077184","8":"3.971227e-36","9":"-2.023523e+00","10":"-1.477340292"},{"1":"married","2":"DEP","3":"SRhealth","4":"p_value","5":"-2.814363e-01","6":"3.664695e-02","7":"-7.679666124","8":"1.595038e-14","9":"-3.530550e-01","10":"-0.209387435"},{"1":"married","2":"DEP","3":"SRhealth","4":"c_value","5":"1.219187e+00","6":"1.691844e-01","7":"7.206264063","8":"5.750783e-13","9":"8.933088e-01","10":"1.556399517"},{"1":"married","2":"DEP","3":"SRhealth","4":"p_value:c_value","5":"-1.368699e-01","6":"4.358084e-02","7":"-3.140599228","8":"1.686026e-03","9":"-2.235577e-01","10":"-0.052760249"},{"1":"married","2":"NegAff","3":"age","4":"(Intercept)","5":"-3.938328e+00","6":"1.681082e-01","7":"-23.427342445","8":"2.250383e-121","9":"-4.272117e+00","10":"-3.613177025"},{"1":"married","2":"NegAff","3":"age","4":"p_value","5":"1.646481e-01","6":"5.963514e-02","7":"2.760923794","8":"5.763812e-03","9":"4.731100e-02","10":"0.281052847"},{"1":"married","2":"NegAff","3":"age","4":"c_value","5":"-5.801033e-02","6":"7.305794e-03","7":"-7.940318824","8":"2.016622e-15","9":"-7.243445e-02","10":"-0.043807721"},{"1":"married","2":"NegAff","3":"age","4":"p_value:c_value","5":"5.652972e-04","6":"2.593619e-03","7":"0.217956892","8":"8.274627e-01","9":"-4.518006e-03","10":"0.005643733"},{"1":"married","2":"NegAff","3":"gender","4":"(Intercept)","5":"-3.486382e+00","6":"1.643774e-01","7":"-21.209623740","8":"7.783317e-100","9":"-3.811419e+00","10":"-3.166975888"},{"1":"married","2":"NegAff","3":"gender","4":"p_value","5":"2.003120e-01","6":"6.110116e-02","7":"3.278367380","8":"1.044094e-03","9":"7.990291e-02","10":"0.319462415"},{"1":"married","2":"NegAff","3":"gender","4":"c_value1","5":"-4.247154e-01","6":"2.357394e-01","7":"-1.801630768","8":"7.160352e-02","9":"-8.872331e-01","10":"0.037004104"},{"1":"married","2":"NegAff","3":"gender","4":"p_value:c_value1","5":"1.195940e-01","6":"8.212832e-02","7":"1.456185009","8":"1.453415e-01","9":"-4.114009e-02","10":"0.280833885"},{"1":"married","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-4.009920e+00","6":"1.435697e-01","7":"-27.930134230","8":"1.149212e-171","9":"-4.293802e+00","10":"-3.730951219"},{"1":"married","2":"NegAff","3":"parEdu","4":"p_value","5":"3.320685e-01","6":"4.873201e-02","7":"6.814177281","8":"9.480474e-12","9":"2.362840e-01","10":"0.427340890"},{"1":"married","2":"NegAff","3":"parEdu","4":"c_value1","5":"1.222472e+00","6":"2.728841e-01","7":"4.479821952","8":"7.470533e-06","9":"6.841839e-01","10":"1.754213974"},{"1":"married","2":"NegAff","3":"parEdu","4":"c_value2","5":"9.623064e-01","6":"5.354305e-01","7":"1.797257559","8":"7.229474e-02","9":"-1.135876e-01","10":"1.988207629"},{"1":"married","2":"NegAff","3":"parEdu","4":"p_value:c_value1","5":"-1.861453e-01","6":"9.654934e-02","7":"-1.927981207","8":"5.385746e-02","9":"-3.760367e-01","10":"0.002552022"},{"1":"married","2":"NegAff","3":"parEdu","4":"p_value:c_value2","5":"-2.275195e-01","6":"1.838781e-01","7":"-1.237338693","8":"2.159614e-01","9":"-5.932748e-01","10":"0.128694760"},{"1":"married","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.271275e+00","6":"1.360142e-01","7":"-31.403154541","8":"1.832372e-216","9":"-4.540163e+00","10":"-4.006947021"},{"1":"married","2":"NegAff","3":"SRhealth","4":"p_value","5":"4.297144e-01","6":"4.443154e-02","7":"9.671380727","8":"3.989587e-22","9":"3.424252e-01","10":"0.516621091"},{"1":"married","2":"NegAff","3":"SRhealth","4":"c_value","5":"8.813359e-01","6":"1.699927e-01","7":"5.184552380","8":"2.165343e-07","9":"5.456038e-01","10":"1.211779400"},{"1":"married","2":"NegAff","3":"SRhealth","4":"p_value:c_value","5":"-2.960367e-02","6":"5.602747e-02","7":"-0.528377879","8":"5.972371e-01","9":"-1.374655e-01","10":"0.082079079"},{"1":"married","2":"OP","3":"age","4":"(Intercept)","5":"-3.592630e+00","6":"2.181616e-01","7":"-16.467750182","8":"6.255389e-61","9":"-4.030080e+00","10":"-3.174621271"},{"1":"married","2":"OP","3":"age","4":"p_value","5":"1.823231e-01","6":"7.270629e-02","7":"2.507666494","8":"1.215313e-02","9":"4.086214e-02","10":"0.325948702"},{"1":"married","2":"OP","3":"age","4":"c_value","5":"-7.356477e-02","6":"1.033920e-02","7":"-7.115132159","8":"1.118056e-12","9":"-9.407276e-02","10":"-0.053549698"},{"1":"married","2":"OP","3":"age","4":"p_value:c_value","5":"7.676586e-03","6":"3.377517e-03","7":"2.272849020","8":"2.303528e-02","9":"1.095274e-03","10":"0.014330478"},{"1":"married","2":"OP","3":"gender","4":"(Intercept)","5":"-3.472611e+00","6":"2.297335e-01","7":"-15.115818891","8":"1.273777e-51","9":"-3.932363e+00","10":"-3.031410101"},{"1":"married","2":"OP","3":"gender","4":"p_value","5":"2.923078e-01","6":"7.421166e-02","7":"3.938840158","8":"8.187645e-05","9":"1.480886e-01","10":"0.439121583"},{"1":"married","2":"OP","3":"gender","4":"c_value1","5":"9.778752e-02","6":"3.200570e-01","7":"0.305531549","8":"7.599613e-01","9":"-5.290101e-01","10":"0.726357039"},{"1":"married","2":"OP","3":"gender","4":"p_value:c_value1","5":"-3.545763e-02","6":"1.040994e-01","7":"-0.340613066","8":"7.333949e-01","9":"-2.397166e-01","10":"0.168460730"},{"1":"married","2":"OP","3":"parEdu","4":"(Intercept)","5":"-3.524480e+00","6":"1.874453e-01","7":"-18.802707094","8":"7.175872e-79","9":"-3.897966e+00","10":"-3.162955256"},{"1":"married","2":"OP","3":"parEdu","4":"p_value","5":"2.923234e-01","6":"6.155856e-02","7":"4.748703439","8":"2.047249e-06","9":"1.723937e-01","10":"0.413773705"},{"1":"married","2":"OP","3":"parEdu","4":"c_value1","5":"8.819446e-01","6":"4.439479e-01","7":"1.986594817","8":"4.696732e-02","9":"-5.629245e-03","10":"1.736388079"},{"1":"married","2":"OP","3":"parEdu","4":"c_value2","5":"-4.980686e-01","6":"8.186850e-01","7":"-0.608376355","8":"5.429379e-01","9":"-2.198696e+00","10":"1.021913165"},{"1":"married","2":"OP","3":"parEdu","4":"p_value:c_value1","5":"-1.602082e-01","6":"1.421052e-01","7":"-1.127391396","8":"2.595770e-01","9":"-4.367497e-01","10":"0.120694492"},{"1":"married","2":"OP","3":"parEdu","4":"p_value:c_value2","5":"2.653133e-01","6":"2.458060e-01","7":"1.079360592","8":"2.804270e-01","9":"-2.023198e-01","10":"0.764209357"},{"1":"married","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.063045e+00","6":"1.696295e-01","7":"-18.057272440","8":"6.915631e-73","9":"-3.400246e+00","10":"-2.735125643"},{"1":"married","2":"OP","3":"SRhealth","4":"p_value","5":"1.211426e-01","6":"5.794735e-02","7":"2.090563363","8":"3.656722e-02","9":"7.963544e-03","10":"0.235169434"},{"1":"married","2":"OP","3":"SRhealth","4":"c_value","5":"7.834016e-01","6":"2.195852e-01","7":"3.567642190","8":"3.602079e-04","9":"3.624568e-01","10":"1.222886059"},{"1":"married","2":"OP","3":"SRhealth","4":"p_value:c_value","5":"-7.251118e-02","6":"7.222817e-02","7":"-1.003918157","8":"3.154181e-01","9":"-2.163807e-01","10":"0.066608643"},{"1":"married","2":"PA","3":"age","4":"(Intercept)","5":"-4.806125e+00","6":"2.487483e-01","7":"-19.321236375","8":"3.560204e-83","9":"-5.304410e+00","10":"-4.329588189"},{"1":"married","2":"PA","3":"age","4":"p_value","5":"3.670041e-01","6":"6.629200e-02","7":"5.536174384","8":"3.091498e-08","9":"2.385964e-01","10":"0.498378215"},{"1":"married","2":"PA","3":"age","4":"c_value","5":"-7.026800e-02","6":"1.097672e-02","7":"-6.401548870","8":"1.538086e-10","9":"-9.202088e-02","10":"-0.049021720"},{"1":"married","2":"PA","3":"age","4":"p_value:c_value","5":"4.193353e-03","6":"2.876084e-03","7":"1.458008074","8":"1.448383e-01","9":"-1.399205e-03","10":"0.009865823"},{"1":"married","2":"PA","3":"gender","4":"(Intercept)","5":"-4.442724e+00","6":"2.514090e-01","7":"-17.671301784","8":"6.976817e-70","9":"-4.944617e+00","10":"-3.959053550"},{"1":"married","2":"PA","3":"gender","4":"p_value","5":"4.069341e-01","6":"6.656668e-02","7":"6.113181108","8":"9.766443e-10","9":"2.777614e-01","10":"0.538710393"},{"1":"married","2":"PA","3":"gender","4":"c_value1","5":"-6.006563e-01","6":"3.544444e-01","7":"-1.694641955","8":"9.014336e-02","9":"-1.295350e+00","10":"0.094552532"},{"1":"married","2":"PA","3":"gender","4":"p_value:c_value1","5":"1.515126e-01","6":"9.245003e-02","7":"1.638859566","8":"1.012425e-01","9":"-2.979319e-02","10":"0.332650602"},{"1":"married","2":"PA","3":"parEdu","4":"(Intercept)","5":"-4.768792e+00","6":"2.119061e-01","7":"-22.504264134","8":"3.770159e-112","9":"-5.190622e+00","10":"-4.359921846"},{"1":"married","2":"PA","3":"parEdu","4":"p_value","5":"4.616859e-01","6":"5.574079e-02","7":"8.282730831","8":"1.203823e-16","9":"3.533308e-01","10":"0.571840561"},{"1":"married","2":"PA","3":"parEdu","4":"c_value1","5":"1.104799e+00","6":"4.222552e-01","7":"2.616425135","8":"8.885585e-03","9":"2.651611e-01","10":"1.921149320"},{"1":"married","2":"PA","3":"parEdu","4":"c_value2","5":"-1.855957e-01","6":"8.644024e-01","7":"-0.214709860","8":"8.299935e-01","9":"-1.967909e+00","10":"1.422975834"},{"1":"married","2":"PA","3":"parEdu","4":"p_value:c_value1","5":"-1.237253e-01","6":"1.093505e-01","7":"-1.131456829","8":"2.578629e-01","9":"-3.363783e-01","10":"0.092369008"},{"1":"married","2":"PA","3":"parEdu","4":"p_value:c_value2","5":"1.306080e-01","6":"2.202929e-01","7":"0.592883478","8":"5.532591e-01","9":"-2.886030e-01","10":"0.575345977"},{"1":"married","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-4.382418e+00","6":"1.870142e-01","7":"-23.433603452","8":"1.942811e-121","9":"-4.754212e+00","10":"-4.020958948"},{"1":"married","2":"PA","3":"SRhealth","4":"p_value","5":"3.611928e-01","6":"5.024484e-02","7":"7.188655016","8":"6.543267e-13","9":"2.633833e-01","10":"0.460369939"},{"1":"married","2":"PA","3":"SRhealth","4":"c_value","5":"7.551192e-01","6":"2.446703e-01","7":"3.086272833","8":"2.026828e-03","9":"2.854716e-01","10":"1.243921294"},{"1":"married","2":"PA","3":"SRhealth","4":"p_value:c_value","5":"-5.375409e-02","6":"6.441859e-02","7":"-0.834450035","8":"4.040274e-01","9":"-1.820184e-01","10":"0.070299364"},{"1":"married","2":"SE","3":"age","4":"(Intercept)","5":"-3.506471e+00","6":"4.009638e-01","7":"-8.745106502","8":"2.228054e-18","9":"-4.328106e+00","10":"-2.756302371"},{"1":"married","2":"SE","3":"age","4":"p_value","5":"-6.475111e-02","6":"7.058245e-02","7":"-0.917382623","8":"3.589422e-01","9":"-1.994468e-01","10":"0.077277498"},{"1":"married","2":"SE","3":"age","4":"c_value","5":"-7.296519e-02","6":"1.590026e-02","7":"-4.588929235","8":"4.455254e-06","9":"-1.048387e-01","10":"-0.042537511"},{"1":"married","2":"SE","3":"age","4":"p_value:c_value","5":"1.889150e-03","6":"2.814872e-03","7":"0.671131942","8":"5.021365e-01","9":"-3.554797e-03","10":"0.007470167"},{"1":"married","2":"SE","3":"gender","4":"(Intercept)","5":"-2.741016e+00","6":"3.776187e-01","7":"-7.258686582","8":"3.908670e-13","9":"-3.510602e+00","10":"-2.028595393"},{"1":"married","2":"SE","3":"gender","4":"p_value","5":"-9.249085e-02","6":"6.643288e-02","7":"-1.392245149","8":"1.638482e-01","9":"-2.197348e-01","10":"0.040949993"},{"1":"married","2":"SE","3":"gender","4":"c_value1","5":"5.524314e-01","6":"4.840008e-01","7":"1.141385193","8":"2.537097e-01","9":"-3.879447e-01","10":"1.512779295"},{"1":"married","2":"SE","3":"gender","4":"p_value:c_value1","5":"-1.004295e-01","6":"8.687798e-02","7":"-1.155983371","8":"2.476880e-01","9":"-2.719634e-01","10":"0.068911620"},{"1":"married","2":"SE","3":"parEdu","4":"(Intercept)","5":"-2.577175e+00","6":"2.946319e-01","7":"-8.747101736","8":"2.189023e-18","9":"-3.173803e+00","10":"-2.017774845"},{"1":"married","2":"SE","3":"parEdu","4":"p_value","5":"-1.546601e-01","6":"5.323681e-02","7":"-2.905135130","8":"3.670946e-03","9":"-2.572087e-01","10":"-0.048349773"},{"1":"married","2":"SE","3":"parEdu","4":"c_value1","5":"8.987935e-01","6":"5.256314e-01","7":"1.709931037","8":"8.727863e-02","9":"-1.515357e-01","10":"1.914043787"},{"1":"married","2":"SE","3":"parEdu","4":"c_value2","5":"5.123218e-01","6":"1.184464e+00","7":"0.432534596","8":"6.653529e-01","9":"-2.045977e+00","10":"2.657676967"},{"1":"married","2":"SE","3":"parEdu","4":"p_value:c_value1","5":"1.213475e-03","6":"9.522021e-02","7":"0.012743884","8":"9.898321e-01","9":"-1.839676e-01","10":"0.189864486"},{"1":"married","2":"SE","3":"parEdu","4":"p_value:c_value2","5":"-2.862127e-02","6":"2.140658e-01","7":"-0.133703088","8":"8.936374e-01","9":"-4.343920e-01","10":"0.413891271"},{"1":"married","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-2.078678e+00","6":"2.438006e-01","7":"-8.526141284","8":"1.513104e-17","9":"-2.568964e+00","10":"-1.612453582"},{"1":"married","2":"SE","3":"SRhealth","4":"p_value","5":"-2.289133e-01","6":"4.522286e-02","7":"-5.061893245","8":"4.151135e-07","9":"-3.165262e-01","10":"-0.139125580"},{"1":"married","2":"SE","3":"SRhealth","4":"c_value","5":"7.588210e-01","6":"3.283364e-01","7":"2.311108618","8":"2.082686e-02","9":"1.385793e-01","10":"1.424018850"},{"1":"married","2":"SE","3":"SRhealth","4":"p_value:c_value","5":"-1.161863e-02","6":"5.957445e-02","7":"-0.195027046","8":"8.453718e-01","9":"-1.315971e-01","10":"0.101541172"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"(Intercept)","5":"-3.282788e+00","6":"2.183471e-01","7":"-15.034722324","8":"4.348680e-51","9":"-3.718844e+00","10":"-2.862950776"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"p_value","5":"-9.004883e-02","6":"5.592628e-02","7":"-1.610134374","8":"1.073685e-01","9":"-1.985126e-01","10":"0.020716403"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"c_value","5":"-7.723722e-02","6":"9.675008e-03","7":"-7.983168638","8":"1.426238e-15","9":"-9.639620e-02","10":"-0.058481808"},{"1":"mvInPrtnr","2":"DEP","3":"age","4":"p_value:c_value","5":"2.627909e-03","6":"2.463910e-03","7":"1.066560199","8":"2.861705e-01","9":"-2.169129e-03","10":"0.007485857"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"(Intercept)","5":"-2.580536e+00","6":"2.144024e-01","7":"-12.035949270","8":"2.299734e-33","9":"-3.007458e+00","10":"-2.166742924"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"p_value","5":"-1.190241e-01","6":"5.386168e-02","7":"-2.209810572","8":"2.711831e-02","9":"-2.236330e-01","10":"-0.012430286"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"c_value1","5":"-1.736087e-01","6":"2.844720e-01","7":"-0.610283792","8":"5.416738e-01","9":"-7.297989e-01","10":"0.385754913"},{"1":"mvInPrtnr","2":"DEP","3":"gender","4":"p_value:c_value1","5":"4.770114e-02","6":"7.265914e-02","7":"0.656505628","8":"5.114988e-01","9":"-9.498648e-02","10":"0.189906991"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"(Intercept)","5":"-2.824489e+00","6":"1.716621e-01","7":"-16.453775118","8":"7.879987e-61","9":"-3.165268e+00","10":"-2.492245672"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"p_value","5":"-9.170610e-02","6":"4.420068e-02","7":"-2.074766701","8":"3.800816e-02","9":"-1.777551e-01","10":"-0.004462667"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"c_value1","5":"1.348044e+00","6":"3.495238e-01","7":"3.856800638","8":"1.148808e-04","9":"6.555370e-01","10":"2.026453598"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"c_value2","5":"6.507281e-01","6":"5.674042e-01","7":"1.146850902","8":"2.514432e-01","9":"-4.984811e-01","10":"1.730164064"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"p_value:c_value1","5":"-1.361090e-01","6":"8.876967e-02","7":"-1.533282735","8":"1.252062e-01","9":"-3.091716e-01","10":"0.038947294"},{"1":"mvInPrtnr","2":"DEP","3":"parEdu","4":"p_value:c_value2","5":"4.872842e-02","6":"1.425499e-01","7":"0.341834108","8":"7.324757e-01","9":"-2.258079e-01","10":"0.333925409"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"(Intercept)","5":"-1.447966e+00","6":"1.538081e-01","7":"-9.414110181","8":"4.771063e-21","9":"-1.751907e+00","10":"-1.148898692"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"p_value","5":"-4.493868e-01","6":"4.140255e-02","7":"-10.854086578","8":"1.907039e-27","9":"-5.303073e-01","10":"-0.367990693"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"c_value","5":"9.665226e-01","6":"1.766916e-01","7":"5.470112049","8":"4.497512e-08","9":"6.275717e-01","10":"1.320185444"},{"1":"mvInPrtnr","2":"DEP","3":"SRhealth","4":"p_value:c_value","5":"-2.880964e-02","6":"4.627089e-02","7":"-0.622629901","8":"5.335278e-01","9":"-1.212156e-01","10":"0.060145858"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"(Intercept)","5":"-4.729912e+00","6":"2.313583e-01","7":"-20.444102608","8":"6.779367e-93","9":"-5.190840e+00","10":"-4.284095429"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"p_value","5":"2.315759e-01","6":"8.022858e-02","7":"2.886451312","8":"3.896130e-03","9":"7.342409e-02","10":"0.387828459"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"c_value","5":"-7.011197e-02","6":"9.583305e-03","7":"-7.316053576","8":"2.553697e-13","9":"-8.905288e-02","10":"-0.051516960"},{"1":"mvInPrtnr","2":"NegAff","3":"age","4":"p_value:c_value","5":"2.683144e-03","6":"3.333842e-03","7":"0.804820532","8":"4.209233e-01","9":"-3.853633e-03","10":"0.009201737"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"(Intercept)","5":"-4.169364e+00","6":"2.146126e-01","7":"-19.427399114","8":"4.527282e-84","9":"-4.594849e+00","10":"-3.753424664"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"p_value","5":"2.517969e-01","6":"7.864697e-02","7":"3.201609024","8":"1.366623e-03","9":"9.641243e-02","10":"0.404779461"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"c_value1","5":"-2.214671e-01","6":"3.013378e-01","7":"-0.734946291","8":"4.623722e-01","9":"-8.124428e-01","10":"0.369077453"},{"1":"mvInPrtnr","2":"NegAff","3":"gender","4":"p_value:c_value1","5":"6.803590e-02","6":"1.043989e-01","7":"0.651691793","8":"5.146000e-01","9":"-1.361368e-01","10":"0.273166444"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"(Intercept)","5":"-4.506962e+00","6":"1.897731e-01","7":"-23.749216311","8":"1.119417e-124","9":"-4.883173e+00","10":"-4.139122464"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"p_value","5":"2.961332e-01","6":"6.494735e-02","7":"4.559588856","8":"5.125387e-06","9":"1.681860e-01","10":"0.422832797"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"c_value1","5":"7.036027e-01","6":"3.489304e-01","7":"2.016455466","8":"4.375237e-02","9":"1.368524e-02","10":"1.382169971"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"c_value2","5":"1.346013e+00","6":"5.568257e-01","7":"2.417296005","8":"1.563630e-02","9":"2.301109e-01","10":"2.415837479"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"p_value:c_value1","5":"6.802061e-02","6":"1.194056e-01","7":"0.569659996","8":"5.689083e-01","9":"-1.666231e-01","10":"0.301657107"},{"1":"mvInPrtnr","2":"NegAff","3":"parEdu","4":"p_value:c_value2","5":"-1.674789e-01","6":"1.897246e-01","7":"-0.882747754","8":"3.773726e-01","9":"-5.452683e-01","10":"0.199650037"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"(Intercept)","5":"-4.903277e+00","6":"1.743645e-01","7":"-28.120840404","8":"5.448884e-174","9":"-5.248710e+00","10":"-4.565137204"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"p_value","5":"4.699666e-01","6":"5.621022e-02","7":"8.360873833","8":"6.225540e-17","9":"3.593809e-01","10":"0.579767003"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"c_value","5":"9.769297e-01","6":"2.161978e-01","7":"4.518684178","8":"6.222513e-06","9":"5.486343e-01","10":"1.395697192"},{"1":"mvInPrtnr","2":"NegAff","3":"SRhealth","4":"p_value:c_value","5":"-5.558590e-02","6":"7.037432e-02","7":"-0.789860513","8":"4.296092e-01","9":"-1.902432e-01","10":"0.085433040"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"(Intercept)","5":"-4.397850e+00","6":"3.154091e-01","7":"-13.943321232","8":"3.454817e-44","9":"-5.036799e+00","10":"-3.800241826"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"p_value","5":"2.219787e-01","6":"1.044474e-01","7":"2.125267614","8":"3.356429e-02","9":"1.998417e-02","10":"0.429462523"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"c_value","5":"-7.349182e-02","6":"1.367314e-02","7":"-5.374904511","8":"7.662328e-08","9":"-1.007725e-01","10":"-0.047204540"},{"1":"mvInPrtnr","2":"OP","3":"age","4":"p_value:c_value","5":"6.868992e-04","6":"4.430346e-03","7":"0.155044142","8":"8.767865e-01","9":"-7.908626e-03","10":"0.009444534"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"(Intercept)","5":"-4.305954e+00","6":"2.901900e-01","7":"-14.838393722","8":"8.271285e-50","9":"-4.890021e+00","10":"-3.751839795"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"p_value","5":"4.422980e-01","6":"9.128948e-02","7":"4.845004663","8":"1.266087e-06","9":"2.655192e-01","10":"0.623571354"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"c_value1","5":"1.429192e-03","6":"3.972373e-01","7":"0.003597829","8":"9.971294e-01","9":"-7.754415e-01","10":"0.783045661"},{"1":"mvInPrtnr","2":"OP","3":"gender","4":"p_value:c_value1","5":"3.293508e-02","6":"1.253505e-01","7":"0.262743846","8":"7.927480e-01","9":"-2.132256e-01","10":"0.278331073"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"(Intercept)","5":"-4.337069e+00","6":"2.387520e-01","7":"-18.165584784","8":"9.667151e-74","9":"-4.815067e+00","10":"-3.878817858"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"p_value","5":"4.222003e-01","6":"7.655589e-02","7":"5.514929371","8":"3.489201e-08","9":"2.734479e-01","10":"0.573656685"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"c_value1","5":"1.058103e+00","6":"4.969931e-01","7":"2.129010297","8":"3.325341e-02","9":"6.520261e-02","10":"2.015531525"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"c_value2","5":"5.339994e-01","6":"8.647328e-01","7":"0.617531109","8":"5.368845e-01","9":"-1.262491e+00","10":"2.139880386"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"p_value:c_value1","5":"-8.957093e-02","6":"1.556868e-01","7":"-0.575327567","8":"5.650698e-01","9":"-3.922671e-01","10":"0.218439660"},{"1":"mvInPrtnr","2":"OP","3":"parEdu","4":"p_value:c_value2","5":"2.715200e-02","6":"2.621473e-01","7":"0.103575362","8":"9.175063e-01","9":"-4.722730e-01","10":"0.558515170"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"(Intercept)","5":"-3.967889e+00","6":"2.130685e-01","7":"-18.622596622","8":"2.107431e-77","9":"-4.393499e+00","10":"-3.557857074"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"p_value","5":"3.139585e-01","6":"7.036403e-02","7":"4.461917298","8":"8.122958e-06","9":"1.768878e-01","10":"0.452825855"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"c_value","5":"9.612227e-01","6":"2.803823e-01","7":"3.428257348","8":"6.074693e-04","9":"4.251706e-01","10":"1.523339994"},{"1":"mvInPrtnr","2":"OP","3":"SRhealth","4":"p_value:c_value","5":"-1.219387e-01","6":"8.899002e-02","7":"-1.370251758","8":"1.706083e-01","9":"-2.994740e-01","10":"0.049022497"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"(Intercept)","5":"-3.985340e+00","6":"2.752354e-01","7":"-14.479748823","8":"1.626906e-47","9":"-4.544394e+00","10":"-3.465031952"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"p_value","5":"-3.929651e-02","6":"7.929988e-02","7":"-0.495543089","8":"6.202168e-01","9":"-1.918058e-01","10":"0.119112255"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"c_value","5":"-3.239754e-02","6":"1.207516e-02","7":"-2.682990229","8":"7.296712e-03","9":"-5.665718e-02","10":"-0.009353744"},{"1":"mvInPrtnr","2":"PA","3":"age","4":"p_value:c_value","5":"-8.421728e-03","6":"3.325894e-03","7":"-2.532170205","8":"1.133590e-02","9":"-1.480308e-02","10":"-0.001777525"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"(Intercept)","5":"-4.446377e+00","6":"3.117495e-01","7":"-14.262660727","8":"3.739215e-46","9":"-5.072653e+00","10":"-3.850470082"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"p_value","5":"2.553346e-01","6":"8.436817e-02","7":"3.026433321","8":"2.474574e-03","9":"9.204978e-02","10":"0.422791115"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"c_value1","5":"-2.634983e-01","6":"4.265166e-01","7":"-0.617791439","8":"5.367128e-01","9":"-1.098028e+00","10":"0.574981520"},{"1":"mvInPrtnr","2":"PA","3":"gender","4":"p_value:c_value1","5":"8.765263e-02","6":"1.144079e-01","7":"0.766141691","8":"4.435920e-01","9":"-1.369451e-01","10":"0.311617516"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"(Intercept)","5":"-4.732756e+00","6":"2.661450e-01","7":"-17.782622527","8":"9.636508e-71","9":"-5.265703e+00","10":"-4.222227708"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"p_value","5":"2.877257e-01","6":"7.190851e-02","7":"4.001274554","8":"6.300220e-05","9":"1.482701e-01","10":"0.430186664"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"c_value1","5":"1.148168e+00","6":"5.090108e-01","7":"2.255685582","8":"2.409033e-02","9":"1.332819e-01","10":"2.130177039"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"c_value2","5":"6.749124e-01","6":"8.229322e-01","7":"0.820131284","8":"4.121413e-01","9":"-1.021333e+00","10":"2.209580634"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"p_value:c_value1","5":"-8.274337e-02","6":"1.347271e-01","7":"-0.614155587","8":"5.391125e-01","9":"-3.445641e-01","10":"0.183738974"},{"1":"mvInPrtnr","2":"PA","3":"parEdu","4":"p_value:c_value2","5":"5.583689e-02","6":"2.160387e-01","7":"0.258457863","8":"7.960536e-01","9":"-3.563486e-01","10":"0.491268464"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"(Intercept)","5":"-4.048804e+00","6":"2.167272e-01","7":"-18.681566104","8":"6.993684e-78","9":"-4.480947e+00","10":"-3.631164518"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"p_value","5":"1.131578e-01","6":"6.081263e-02","7":"1.860761938","8":"6.277780e-02","9":"-5.151518e-03","10":"0.233286522"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"c_value","5":"-9.332520e-02","6":"2.512102e-01","7":"-0.371502474","8":"7.102633e-01","9":"-5.696354e-01","10":"0.415327333"},{"1":"mvInPrtnr","2":"PA","3":"SRhealth","4":"p_value:c_value","5":"2.016644e-01","6":"6.900549e-02","7":"2.922439805","8":"3.473007e-03","9":"6.240762e-02","10":"0.332912663"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"(Intercept)","5":"-4.734831e+00","6":"5.721130e-01","7":"-8.276042483","8":"1.273366e-16","9":"-5.930023e+00","10":"-3.686904034"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"p_value","5":"5.558069e-02","6":"9.804222e-02","7":"0.566905622","8":"5.707783e-01","9":"-1.285691e-01","10":"0.255902612"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"c_value","5":"-5.434195e-02","6":"2.281203e-02","7":"-2.382162207","8":"1.721131e-02","9":"-1.001782e-01","10":"-0.010889783"},{"1":"mvInPrtnr","2":"SE","3":"age","4":"p_value:c_value","5":"-6.762776e-04","6":"3.933583e-03","7":"-0.171924077","8":"8.634972e-01","9":"-8.250750e-03","10":"0.007131034"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"(Intercept)","5":"-4.710900e+00","6":"6.092170e-01","7":"-7.732713962","8":"1.052777e-14","9":"-5.976129e+00","10":"-3.583629814"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"p_value","5":"1.465407e-01","6":"1.019729e-01","7":"1.437055432","8":"1.507022e-01","9":"-4.566050e-02","10":"0.354734959"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"c_value1","5":"1.404005e+00","6":"7.605900e-01","7":"1.845941758","8":"6.490066e-02","9":"-6.471431e-02","10":"2.928128579"},{"1":"mvInPrtnr","2":"SE","3":"gender","4":"p_value:c_value1","5":"-2.645658e-01","6":"1.310858e-01","7":"-2.018264963","8":"4.356367e-02","9":"-5.250665e-01","10":"-0.010127092"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"(Intercept)","5":"-4.089182e+00","6":"4.600000e-01","7":"-8.889526295","8":"6.136998e-19","9":"-5.036902e+00","10":"-3.231242466"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"p_value","5":"-1.161989e-03","6":"7.992876e-02","7":"-0.014537812","8":"9.884009e-01","9":"-1.530927e-01","10":"0.160615102"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"c_value1","5":"9.002365e-01","6":"8.586927e-01","7":"1.048380212","8":"2.944635e-01","9":"-8.546543e-01","10":"2.530085617"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"c_value2","5":"1.369793e+00","6":"1.312391e+00","7":"1.043737911","8":"2.966067e-01","9":"-1.457283e+00","10":"3.748421543"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"p_value:c_value1","5":"-2.614593e-02","6":"1.503592e-01","7":"-0.173889814","8":"8.619521e-01","9":"-3.149125e-01","10":"0.276497495"},{"1":"mvInPrtnr","2":"SE","3":"parEdu","4":"p_value:c_value2","5":"-4.727366e-02","6":"2.286291e-01","7":"-0.206770113","8":"8.361894e-01","9":"-4.762004e-01","10":"0.428234776"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"(Intercept)","5":"-3.602046e+00","6":"3.725394e-01","7":"-9.668899368","8":"4.087502e-22","9":"-4.361335e+00","10":"-2.899434696"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"p_value","5":"-6.746959e-02","6":"6.611931e-02","7":"-1.020421886","8":"3.075284e-01","9":"-1.942705e-01","10":"0.065166238"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"c_value","5":"4.206358e-01","6":"4.940411e-01","7":"0.851418517","8":"3.945369e-01","9":"-4.939860e-01","10":"1.432399249"},{"1":"mvInPrtnr","2":"SE","3":"SRhealth","4":"p_value:c_value","5":"2.709925e-02","6":"8.609944e-02","7":"0.314743628","8":"7.529563e-01","9":"-1.481736e-01","10":"0.187392071"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Create the Table  

The basic steps from here are similar: filter target terms, index significance, exponentiate, format values, create CI's, bold significance, select needed columns.  

We do need to a bit of work on how we index our moderators. Specifically, for the factor variables, we want to make sure we indicate what the levels are. 


```{.r .code-style}
tidy4 <- tidy4 %>%
  filter(grepl("p_value:", term)) %>%
  mutate(term = str_replace(term, "c_value", Moderator),
         Moderator = str_remove(term, "p_value:"),
         sig = ifelse(sign(conf.low) == sign(conf.high), "sig", "ns")) %>%
  mutate_at(vars(estimate, conf.low, conf.high), exp) %>%
  mutate_at(vars(estimate, conf.low, conf.high), ~sprintf("%.2f", .)) %>%
  mutate(CI = sprintf("[%s, %s]", conf.low, conf.high)) %>%
  mutate_at(vars(estimate, CI), ~ifelse(sig == "sig", sprintf("<strong>%s</strong>", .), .)) %>%
  select(Outcome, Trait, Moderator, OR = estimate, CI)
```

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Create the Table  

Now we're ready for the `pivot_longer()`, `unite()`, `pivot_wider()`, order columns (using `select()`) combo from before. The only difference is that I'm going to use `pivot_wider()` to do the uniting for me this time!  


```{.r .code-style}
O_names <- tibble(
  old = c("mvInPrtnr", "married", "divorced", "chldbrth"),
  new = c("Move in with Partner", "Married", "Divorced", "Birth of a Child")
)
levs <- paste(rep(O_names$old, each = 2), rep(c("OR","CI"), times = 4), sep = "_")

tidy4 <- tidy4 %>%
  pivot_longer(cols = c(OR, CI), names_to = "est", values_to = "value") %>%
  pivot_wider(names_from = c("Outcome", "est"), values_from = "value", names_sep = "_") %>%
  select(Trait, Moderator, all_of(levs))
```

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Kabling the Table  

All right, time to use `kable()`! This will proceed almost exactly as the example with covariates, with the main difference being that our target term is now the interaction.  

Given that we have multiple moderators, some of which have multiple levels, we'll also want to order them. So we'll make the Moderator column a factor as well. But we'll do so after we've given our covariates nicer names.  While we're at it, we'll go ahead and do the same things for our Trait column.  

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Kabling the Table  
All right, time to use `kable()`! This will proceed almost exactly as the example with covariates, with the main difference being that our target term is now the interaction.  

Given that we have multiple moderators, some of which have multiple levels, we'll also want to order them. So we'll make the Moderator column a factor as well. But we'll do so after we've given our covariates nicer names.  While we're at it, we'll go ahead and do the same things for our Trait column.  


```{.r .code-style}
m_names <- tibble(
  old = c("age", "SRhealth", "gender1", "parEdu1", "parEdu2"),
  new = c("Age", "Self-Rated Health", "Gender (Female)", 
          "Parental Education (College)", "Parental Education (Beyond College)")
)

tidy4 <- tidy4 %>%
  mutate(Trait = mapvalues(Trait, from = p_names$old, to = p_names$new),
         Trait = factor(Trait, levels = p_names$new),
         Moderator = mapvalues(Moderator, from = m_names$old, to = m_names$new),
         Moderator = factor(Moderator, levels = m_names$new)) %>%
  arrange(Trait, Moderator)
```

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Kabling the Table  
And again, for our spanned columns, we'll take advantage of our `O_names` object to create the vector in advance:


```{.r .code-style}
heads <- rep(2, 5)
heads
```

```
## [1] 2 2 2 2 2
```

```{.r .code-style}
names(heads) <- c(" ", O_names$new)
heads
```

```
##                      Move in with Partner              Married 
##                    2                    2                    2 
##             Divorced     Birth of a Child 
##                    2                    2
```

### Advanced Lessons: Multiple DVs/Outcomes, Multiple Model Terms, Additional Formatting -- Moderators  
#### Kabling the Table  

Now, this will proceed as before, including using `collapse_rows()`. The change this time will simply be to our table caption.  


```{.r .code-style}
tidy4 %>%
  kable(., escape = F,
        align = c("r", "r", rep("c", 8)),
        col.names = c("Trait", "Moderator", rep(c("OR", "CI"), times = 4)),
        caption = "<strong>Table 4</strong><br><em>Estimated Moderators of Personality-Outcome Associations</em>") %>%
  kable_styling(full_width = F) %>%
  collapse_rows(1, valign = "top") %>%
  add_header_above(heads) %>%
  add_footnote(label = "Bold values indicate terms whose confidence intervals did not overlap with 0", notation = "none")
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
<caption>
<strong>Table 4</strong><br><em>Estimated Moderators of Personality-Outcome Associations</em>
</caption>
 <thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="2"></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Move in with Partner</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Married</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Divorced</div></th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="2"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">Birth of a Child</div></th>
</tr>
  <tr>
   <th style="text-align:right;"> Trait </th>
   <th style="text-align:right;"> Moderator </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
   <th style="text-align:center;"> OR </th>
   <th style="text-align:center;"> CI </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Negative Affect </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.99, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.83, 1.09] </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.87, 1.09] </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.80, 1.11] </td>
   <td style="text-align:center;"> 0.91 </td>
   <td style="text-align:center;"> [0.80, 1.05] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender (Female) </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.87, 1.31] </td>
   <td style="text-align:center;"> 1.13 </td>
   <td style="text-align:center;"> [0.96, 1.32] </td>
   <td style="text-align:center;"> 0.77 </td>
   <td style="text-align:center;"> [0.59, 1.01] </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.82, 1.20] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (College) </td>
   <td style="text-align:center;"> 1.07 </td>
   <td style="text-align:center;"> [0.85, 1.35] </td>
   <td style="text-align:center;"> 0.83 </td>
   <td style="text-align:center;"> [0.69, 1.00] </td>
   <td style="text-align:center;"> 1.13 </td>
   <td style="text-align:center;"> [0.76, 1.65] </td>
   <td style="text-align:center;"> 1.11 </td>
   <td style="text-align:center;"> [0.89, 1.39] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (Beyond College) </td>
   <td style="text-align:center;"> 0.85 </td>
   <td style="text-align:center;"> [0.58, 1.22] </td>
   <td style="text-align:center;"> 0.80 </td>
   <td style="text-align:center;"> [0.55, 1.14] </td>
   <td style="text-align:center;"> 1.39 </td>
   <td style="text-align:center;"> [0.82, 2.33] </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.76, 1.56] </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Positive Affect </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> <strong>0.99</strong> </td>
   <td style="text-align:center;"> <strong>[0.99, 1.00]</strong> </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> <strong>1.01</strong> </td>
   <td style="text-align:center;"> <strong>[1.00, 1.02]</strong> </td>
   <td style="text-align:center;"> <strong>1.01</strong> </td>
   <td style="text-align:center;"> <strong>[1.00, 1.02]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> <strong>1.22</strong> </td>
   <td style="text-align:center;"> <strong>[1.06, 1.40]</strong> </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.83, 1.07] </td>
   <td style="text-align:center;"> 1.01 </td>
   <td style="text-align:center;"> [0.85, 1.19] </td>
   <td style="text-align:center;"> 0.92 </td>
   <td style="text-align:center;"> [0.78, 1.08] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender (Female) </td>
   <td style="text-align:center;"> 1.09 </td>
   <td style="text-align:center;"> [0.87, 1.37] </td>
   <td style="text-align:center;"> 1.16 </td>
   <td style="text-align:center;"> [0.97, 1.39] </td>
   <td style="text-align:center;"> 1.13 </td>
   <td style="text-align:center;"> [0.86, 1.48] </td>
   <td style="text-align:center;"> <strong>1.26</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.57]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (College) </td>
   <td style="text-align:center;"> 0.92 </td>
   <td style="text-align:center;"> [0.71, 1.20] </td>
   <td style="text-align:center;"> 0.88 </td>
   <td style="text-align:center;"> [0.71, 1.10] </td>
   <td style="text-align:center;"> 0.98 </td>
   <td style="text-align:center;"> [0.65, 1.49] </td>
   <td style="text-align:center;"> 0.93 </td>
   <td style="text-align:center;"> [0.71, 1.22] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (Beyond College) </td>
   <td style="text-align:center;"> 1.06 </td>
   <td style="text-align:center;"> [0.70, 1.63] </td>
   <td style="text-align:center;"> 1.14 </td>
   <td style="text-align:center;"> [0.75, 1.78] </td>
   <td style="text-align:center;"> <strong>0.47</strong> </td>
   <td style="text-align:center;"> <strong>[0.28, 0.79]</strong> </td>
   <td style="text-align:center;"> <strong>0.49</strong> </td>
   <td style="text-align:center;"> <strong>[0.34, 0.73]</strong> </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Self-Esteem </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.99, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.99, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.99, 1.01] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 1.03 </td>
   <td style="text-align:center;"> [0.86, 1.21] </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.88, 1.11] </td>
   <td style="text-align:center;"> 1.13 </td>
   <td style="text-align:center;"> [0.92, 1.35] </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.83, 1.17] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender (Female) </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.59, 0.99]</strong> </td>
   <td style="text-align:center;"> 0.90 </td>
   <td style="text-align:center;"> [0.76, 1.07] </td>
   <td style="text-align:center;"> 1.15 </td>
   <td style="text-align:center;"> [0.83, 1.60] </td>
   <td style="text-align:center;"> <strong>0.73</strong> </td>
   <td style="text-align:center;"> <strong>[0.57, 0.94]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (College) </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.73, 1.32] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.83, 1.21] </td>
   <td style="text-align:center;"> 0.88 </td>
   <td style="text-align:center;"> [0.59, 1.36] </td>
   <td style="text-align:center;"> 0.91 </td>
   <td style="text-align:center;"> [0.71, 1.18] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (Beyond College) </td>
   <td style="text-align:center;"> 0.95 </td>
   <td style="text-align:center;"> [0.62, 1.53] </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.65, 1.51] </td>
   <td style="text-align:center;"> NA </td>
   <td style="text-align:center;"> NA </td>
   <td style="text-align:center;"> 0.82 </td>
   <td style="text-align:center;"> [0.51, 1.43] </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Optimism </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [0.99, 1.01] </td>
   <td style="text-align:center;"> <strong>1.01</strong> </td>
   <td style="text-align:center;"> <strong>[1.00, 1.01]</strong> </td>
   <td style="text-align:center;"> 1.01 </td>
   <td style="text-align:center;"> [1.00, 1.02] </td>
   <td style="text-align:center;"> <strong>1.02</strong> </td>
   <td style="text-align:center;"> <strong>[1.01, 1.02]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 0.89 </td>
   <td style="text-align:center;"> [0.74, 1.05] </td>
   <td style="text-align:center;"> 0.93 </td>
   <td style="text-align:center;"> [0.81, 1.07] </td>
   <td style="text-align:center;"> 1.01 </td>
   <td style="text-align:center;"> [0.82, 1.24] </td>
   <td style="text-align:center;"> <strong>0.77</strong> </td>
   <td style="text-align:center;"> <strong>[0.65, 0.92]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender (Female) </td>
   <td style="text-align:center;"> 1.03 </td>
   <td style="text-align:center;"> [0.81, 1.32] </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.79, 1.18] </td>
   <td style="text-align:center;"> 1.06 </td>
   <td style="text-align:center;"> [0.76, 1.48] </td>
   <td style="text-align:center;"> 1.06 </td>
   <td style="text-align:center;"> [0.84, 1.33] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (College) </td>
   <td style="text-align:center;"> 0.91 </td>
   <td style="text-align:center;"> [0.68, 1.24] </td>
   <td style="text-align:center;"> 0.85 </td>
   <td style="text-align:center;"> [0.65, 1.13] </td>
   <td style="text-align:center;"> 0.74 </td>
   <td style="text-align:center;"> [0.44, 1.25] </td>
   <td style="text-align:center;"> 1.04 </td>
   <td style="text-align:center;"> [0.76, 1.42] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (Beyond College) </td>
   <td style="text-align:center;"> 1.03 </td>
   <td style="text-align:center;"> [0.62, 1.75] </td>
   <td style="text-align:center;"> 1.30 </td>
   <td style="text-align:center;"> [0.82, 2.15] </td>
   <td style="text-align:center;"> 1.73 </td>
   <td style="text-align:center;"> [0.78, 4.28] </td>
   <td style="text-align:center;"> 0.77 </td>
   <td style="text-align:center;"> [0.51, 1.19] </td>
  </tr>
  <tr>
   <td style="text-align:right;vertical-align: top !important;" rowspan="5"> Depression </td>
   <td style="text-align:right;"> Age </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> <strong>1.01</strong> </td>
   <td style="text-align:center;"> <strong>[1.00, 1.01]</strong> </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
   <td style="text-align:center;"> 1.00 </td>
   <td style="text-align:center;"> [1.00, 1.01] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Self-Rated Health </td>
   <td style="text-align:center;"> 0.97 </td>
   <td style="text-align:center;"> [0.89, 1.06] </td>
   <td style="text-align:center;"> <strong>0.87</strong> </td>
   <td style="text-align:center;"> <strong>[0.80, 0.95]</strong> </td>
   <td style="text-align:center;"> 0.91 </td>
   <td style="text-align:center;"> [0.80, 1.02] </td>
   <td style="text-align:center;"> <strong>0.88</strong> </td>
   <td style="text-align:center;"> <strong>[0.79, 0.97]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Gender (Female) </td>
   <td style="text-align:center;"> 1.05 </td>
   <td style="text-align:center;"> [0.91, 1.21] </td>
   <td style="text-align:center;"> 1.03 </td>
   <td style="text-align:center;"> [0.91, 1.17] </td>
   <td style="text-align:center;"> 1.21 </td>
   <td style="text-align:center;"> [1.00, 1.48] </td>
   <td style="text-align:center;"> 1.04 </td>
   <td style="text-align:center;"> [0.90, 1.20] </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (College) </td>
   <td style="text-align:center;"> 0.87 </td>
   <td style="text-align:center;"> [0.73, 1.04] </td>
   <td style="text-align:center;"> 0.94 </td>
   <td style="text-align:center;"> [0.79, 1.10] </td>
   <td style="text-align:center;"> 0.99 </td>
   <td style="text-align:center;"> [0.73, 1.34] </td>
   <td style="text-align:center;"> <strong>0.80</strong> </td>
   <td style="text-align:center;"> <strong>[0.67, 0.96]</strong> </td>
  </tr>
  <tr>
   
   <td style="text-align:right;"> Parental Education (Beyond College) </td>
   <td style="text-align:center;"> 1.05 </td>
   <td style="text-align:center;"> [0.80, 1.40] </td>
   <td style="text-align:center;"> 1.06 </td>
   <td style="text-align:center;"> [0.80, 1.43] </td>
   <td style="text-align:center;"> 0.85 </td>
   <td style="text-align:center;"> [0.56, 1.33] </td>
   <td style="text-align:center;"> 1.12 </td>
   <td style="text-align:center;"> [0.84, 1.52] </td>
  </tr>
</tbody>
<tfoot>
<tr>
<td style = 'padding: 0; border:0;' colspan='100%'><sup></sup> Bold values indicate terms whose confidence intervals did not overlap with 0</td>
</tr>
</tfoot>
</table>