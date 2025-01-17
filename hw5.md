hw5
================
Henry Stoddard
11/10/2020

``` r
library(readr)
library(tidyverse)
```

    ## -- Attaching packages ---------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v dplyr   1.0.1
    ## v tibble  3.0.3     v stringr 1.4.0
    ## v tidyr   1.1.1     v forcats 0.5.0
    ## v purrr   0.3.4

    ## -- Conflicts ------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggplot2)
library(patchwork)

knitr::opts_chunk$set(
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)
theme_set(theme_minimal() + theme(legend.position = "bottom"))
options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis"
)
scale_colour_discrete = scale_color_viridis_d
scale_fill_discrete = scale_fill_viridis_d
set.seed(123)
```

Load libraries, read in data \#\# Problem 1

``` r
homicide_df =
  read_csv("data/homicide-data.csv") %>% 
  mutate(
    city_state = str_c(city, state, sep = "_"),
    resolved = case_when(
      disposition == "Closed without arrest" ~ "unsolved",
      disposition == "Open/No arrest" ~ "unsolved",
      disposition == "Closed by arrest" ~ "solved",
    )
  ) %>% 
  select(city_state, resolved) %>% 
  filter(city_state != "Tulsa_AL")
```

    ## Parsed with column specification:
    ## cols(
    ##   uid = col_character(),
    ##   reported_date = col_double(),
    ##   victim_last = col_character(),
    ##   victim_first = col_character(),
    ##   victim_race = col_character(),
    ##   victim_age = col_character(),
    ##   victim_sex = col_character(),
    ##   city = col_character(),
    ##   state = col_character(),
    ##   lat = col_double(),
    ##   lon = col_double(),
    ##   disposition = col_character()
    ## )

This dataset contains information on homicide reports by city. It
contains the report date, victim’s name, age, and sex, and the outcome
of the case.

Looking closer, city by city

``` r
aggregate_df =
homicide_df %>% 
  group_by(city_state) %>% 
  summarize(
    hom_total = n(),
    hom_unsolved = sum(resolved == "unsolved")
  )
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

Prop test for single city

``` r
prop.test(
  aggregate_df %>% filter(city_state == "Baltimore_MD") %>% pull(hom_unsolved), aggregate_df %>% filter(city_state == "Baltimore_MD") %>% pull(hom_total)) %>% broom::tidy()
```

    ## # A tibble: 1 x 8
    ##   estimate statistic  p.value parameter conf.low conf.high method    alternative
    ##      <dbl>     <dbl>    <dbl>     <int>    <dbl>     <dbl> <chr>     <chr>      
    ## 1    0.646      239. 6.46e-54         1    0.628     0.663 1-sample~ two.sided

Now trying to iterate We want to run that above prop.test on every row
within the aggregate\_df and create a new column for those results
(using map2 because mapping over multiple inputs)

``` r
results_df =
aggregate_df %>% 
  mutate(
    prop_tests = map2(.x = hom_unsolved, .y = hom_total, ~prop.test(x = .x, n = .y)),
    tidy_tests = map(.x = prop_tests, ~broom::tidy(.x))
  ) %>% 
  select(-prop_tests) %>% 
  unnest(tidy_tests) %>% 
  select(city_state, estimate, conf.low, conf.high)
```

``` r
results_df %>%
  mutate(city_state = fct_reorder(city_state, estimate)) %>% 
  ggplot(aes(x = city_state, y = estimate)) +
  geom_point() +
  geom_errorbar(aes(ymin = conf.low, ymax = conf.high)) +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
```

<img src="hw5_files/figure-gfm/unnamed-chunk-6-1.png" width="90%" />

Above code creates a graph showing the proportion of unsolved homicides
with confidence intervals by city (in ascending order).

## Question 2

Importing and tidying new dataset.

``` r
path_df = 
  tibble(
  path = list.files("data2")) %>% 
  mutate(path = str_c("data2/", path),
         data = map(.x = path, ~read_csv(.x))) %>% 
  unnest(data) %>% 
  mutate(ID1 = str_replace(path, ".*/", ""),
         ID2 = str_replace(ID1, ".csv", "")) %>% 
  select(-ID1, -path) %>% 
  select(ID2, everything()) %>% 
  separate(ID2, c("treatment", "ID"), sep = "([_])", convert = TRUE) %>% 
  pivot_longer(week_1:week_8, names_to = "time", values_to = "score")
```

    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )

``` r
data_1 = read_csv("data2/con_01.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   week_1 = col_double(),
    ##   week_2 = col_double(),
    ##   week_3 = col_double(),
    ##   week_4 = col_double(),
    ##   week_5 = col_double(),
    ##   week_6 = col_double(),
    ##   week_7 = col_double(),
    ##   week_8 = col_double()
    ## )

This dataset has been tidied and now contains treatment (control or
experimental), ID \#, and measurements from weeks 1-8. Now creating a
spaghetti plot showing observations on each subject over time.

``` r
spag_plot = 
path_df %>% 
  ggplot(aes(x = time, y = score, group = ID, color = ID)) +
  geom_point() +
  geom_line() +
  facet_grid(. ~ treatment)
spag_plot
```

<img src="hw5_files/figure-gfm/unnamed-chunk-8-1.png" width="90%" /> The
control group appears to have started with lower scores and remained
low. The experimental group appears to have started with higher scores
and got even higher scores over time.

## Question 3

``` r
sim_ttest = function(samp_size = 30, mu, sigma = 5) {
  
  sim_data = 
    tibble(
      x = rnorm(n = samp_size, mean = mu, sd = sigma)
    ) %>% t.test(alternative = 'two.sided', paired = FALSE, conf.level = 0.95) %>% 
    broom::tidy() %>% select(estimate, p.value)
   
  return(sim_data)
}


sim_results = 
  rerun(5000, sim_ttest(mu = 0)) %>% 
  bind_rows()

mu_list =
  list("mu = 1" = 1,
       "mu = 2" = 2,
       "mu = 3" = 3,
       "mu = 4" = 4,
       "mu = 5" = 5,
       "mu = 6" = 6)
output = vector("list", length = 6)
for (i in 1:6) {
  output[[i]] =
    rerun(5000, sim_ttest(mu = mu_list[[i]])) %>% 
    bind_rows()
}

mu1_df =
  output[[1]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 1) %>% 
      summarise(proportion = mean(reject),
                mu = 1)

mu2_df =
  output[[2]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 2) %>% 
      summarise(proportion = mean(reject),
                mu = 2)

mu3_df =
  output[[3]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 3) %>% 
      summarise(proportion = mean(reject),
                mu = 3)
mu4_df =
  output[[4]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 4) %>% 
      summarise(proportion = mean(reject),
                mu = 4)
mu5_df =
  output[[5]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 5) %>% 
      summarise(proportion = mean(reject),
                mu = 5)
mu6_df =
  output[[6]] %>% 
  mutate(
    reject = if_else(p.value < 0.05, 1, 0),
    mu = 6) %>% 
      summarise(proportion = mean(reject),
                mu = 6)
bigmu = full_join(mu1_df, mu2_df, by = c("proportion", "mu"))
bigmu2 = full_join(bigmu, mu3_df, by = c("proportion", "mu"))
bigmu3 = full_join(bigmu2, mu4_df, by = c("proportion", "mu"))
bigmu4 = full_join(bigmu3, mu5_df, by = c("proportion", "mu"))
bigmu5 = full_join(bigmu4, mu6_df, by = c("proportion", "mu"))

plot1 = 
  bigmu5 %>% 
  ggplot(aes(x = mu, y = proportion)) +
  geom_point() +
  geom_line()
```

My goodness that’s an ugly way to produce that graph, sorry\! It appears
that as effect size increases (i.e. as our sample mu increases), the
proportion of nulls rejected increases (power increases).

``` r
mu_list =
  list("mu = 1" = 1,
       "mu = 2" = 2,
       "mu = 3" = 3,
       "mu = 4" = 4,
       "mu = 5" = 5,
       "mu = 6" = 6)
output = vector("list", length = 6)
for (i in 1:6) {
  output[[i]] =
    rerun(5000, sim_ttest(mu = mu_list[[i]])) %>% 
    bind_rows() %>% mutate(mu = i)
}

q3p2 = 
  output %>% 
  bind_rows() %>% 
  group_by(mu) %>% 
  summarise(avg_estimate = mean(estimate),
            mu = mu) %>% 
  distinct(.keep_all = TRUE)
```

    ## `summarise()` regrouping output by 'mu' (override with `.groups` argument)

``` r
plot1 =
  q3p2 %>% 
  ggplot(aes(x = mu, y = avg_estimate)) +
  geom_point() +
  geom_line()+
  labs(title = "avg muhat estimate by true mu")
  

q3p2.2 = 
  output %>% 
  bind_rows() %>% 
  filter(p.value < 0.05) %>% 
  group_by(mu) %>% 
  summarise(avg_estimate = mean(estimate),
            mu = mu) %>% 
  distinct(.keep_all = TRUE)
```

    ## `summarise()` regrouping output by 'mu' (override with `.groups` argument)

``` r
plot2 =
  q3p2.2 %>% 
  ggplot(aes(x = mu, y = avg_estimate)) +
  geom_point() +
  geom_line() +
  labs(title = "avg muhat estimate by true mu where null rejected")

plot1 + plot2
```

<img src="hw5_files/figure-gfm/unnamed-chunk-10-1.png" width="90%" />

Maybe, the sample average of muhat across tests for which the null is
rejected is not exactly equal to the true value of mu, but it does seem
close. I am running this with only 10 datasets at a time instead of
5000, so it is possible that with 5000 datasets the approximation would
be closer. However, since we are filtering by those that rejected the
null value, it likely means these samples had mean estimates far from
the true value, making them inherently bad estimates.
