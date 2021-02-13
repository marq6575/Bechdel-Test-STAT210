Bechdel Test
================
Created by Mine Çetinkaya-Rundel

<!--- you might change html_document to github_document if you are working with GitHub--->

In this mini analysis we work with the data used in the FiveThirtyEight
story titled [“The Dollar-And-Cents Case Against Hollywood’s Exclusion
of
Women”](https://fivethirtyeight.com/features/the-dollar-and-cents-case-against-hollywoods-exclusion-of-women/).

1.  Your task is to fill in the blanks denoted by `___` with text or
    code.  
2.  Fix any errors.  
3.  Identify specific R code function and how you looked them up.

## Data and packages

We start with loading the packages we’ll use.

``` r
library(fivethirtyeight)
library(tidyverse) ## error fixed: changed T to t
```

The dataset contains information on 1794 movies released between 1970
and 2013. However we’ll focus our analysis on movies released between
1990 and 2013.

``` r
bechdel90_13 <- bechdel %>% 
  filter(between(year, 1990, 2013))
```

> What is the `%>%` inside the code? What does it do? How did you
> determine your answer?

The `%>%` inside the code is called a “pipe”. Their function is similar
to actual pipes; results from one function get piped into the input of
the next function, and so on until there are no more pipes. I determined
this answer based on knowledge I picked up from classes I have had in
the past that utilized R.

There are 1615 such movies.

The financial variables we’ll focus on are the following:

  - `budget_2013`: Budget in 2013 inflation adjusted dollars
  - `domgross_2013`: Domestic gross (US) in 2013 inflation adjusted
    dollars
  - `intgross_2013`: Total International (i.e., worldwide) gross in 2013
    inflation adjusted dollars

And we’ll also use the `binary` and `clean_test` variables for
**grouping**.

## Analysis

Let’s take a look at how median budget and gross vary by whether the
movie passed the Bechdel test, which is stored in the `binary` variable.

``` r
bechdel90_13 %>%
  group_by(binary) %>%
  summarise(med_budget = median(budget_2013),
            med_domgross = median(domgross_2013, na.rm = TRUE),
            med_intgross = median(intgross_2013, na.rm = TRUE))
```

    # A tibble: 2 x 4
      binary med_budget med_domgross med_intgross
      <chr>       <dbl>        <dbl>        <dbl>
    1 FAIL    48385984.    57318606.    104475669
    2 PASS    31070724     45330446.     80124349

> What does `summarise()` do? How did you look it up?

`summarise()` allows us to summarize the data using summary statistics.
In this case, the function was used to create multiple new variables
within the new dataset at once so to store and call the median values
for these variables more easily. I already knew this from prior classes
as well, but I still Googled the function just to make sure since I had
never seen it used like that before.

Next, let’s take a look at how median budget and gross vary by a more
detailed indicator of the Bechdel test result. This information is
stored in the `clean_test` variable, which takes on the following
values:

  - `ok` = passes test
  - `dubious`
  - `men` = women only talk about men
  - `notalk` = women don’t talk to each other
  - `nowomen` = fewer than two women

<!-- end list -->

``` r
bechdel90_13 %>%
  group_by(clean_test) %>% ## error fixed: filled in gap with clean_test variable
  summarise(med_budget = median(budget_2013),
            med_domgross = median(domgross_2013, na.rm = TRUE),
            med_intgross = median(intgross_2013, na.rm = TRUE))
```

    # A tibble: 5 x 4
      clean_test med_budget med_domgross med_intgross
      <ord>           <dbl>        <dbl>        <dbl>
    1 nowomen     43373066     44891296.    89509349 
    2 notalk      56570084.    63890455    123102194 
    3 men         39737690.    56392786     99578022.
    4 dubious     35790994     49173429     89883201 
    5 ok          31070724     45330446.    80124349 

In order to evaluate how return on investment varies among movies that
pass and fail the Bechdel test, we’ll first create a new variable called
`roi` as the ratio of the gross to budget.

``` r
bechdel90_13 <- bechdel90_13 %>%
  mutate(roi = (intgross_2013 + domgross_2013) / budget_2013) ## error fixed: added missing parenthesis at end
```

Let’s see which movies have the highest return on investment.

``` r
bechdel90_13 %>%
  arrange(desc(roi)) %>% 
  select(title, roi, year)
```

    # A tibble: 1,615 x 3
       title                     roi  year
       <chr>                   <dbl> <int>
     1 Paranormal Activity      671.  2007
     2 The Blair Witch Project  648.  1999
     3 El Mariachi              583.  1992
     4 Clerks.                  258.  1994
     5 In the Company of Men    231.  1997
     6 Napoleon Dynamite        227.  2004
     7 Once                     190.  2006
     8 The Devil Inside         155.  2012
     9 Primer                   142.  2004
    10 Fireproof                134.  2008
    # … with 1,605 more rows

> What does `select()` do? How did you look it up?

`select()` basically does exactly what it sounds like. In this case, it
selected three columns from the dataset that was piped into it and
printed them as a stand alone dataset (with roi in descending order
thanks to the `arrange()` function). I am pretty familiar with the dplyr
package thanks to the classes I took last semester (shoutout to
STAT310), so I didn’t have to look this function up to be familiar with
it.

Below is a visualization of the return on investment by test result,
however it’s difficult to see the distributions due to a few extreme
observations.

``` r
ggplot(data = bechdel90_13, 
       mapping = aes(x = clean_test, y = roi, color = binary)) + ## error fixed: missing comma after y = roi
  geom_boxplot() +
  labs(title = "Return on investment vs. Bechdel test result",
       x = "Detailed Bechdel result",
       y = "ROI",
       color = "Binary Bechdel result")
```

![](The_Bechdel_Test_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

What are those movies with *very* high returns on investment?

``` r
bechdel90_13 %>%
  filter(roi > 400) %>%
  select(title, budget_2013, domgross_2013, year)
```

    # A tibble: 3 x 4
      title                   budget_2013 domgross_2013  year
      <chr>                         <int>         <dbl> <int>
    1 Paranormal Activity          505595     121251476  2007
    2 The Blair Witch Project      839077     196538593  1999
    3 El Mariachi                   11622       3388636  1992

Zooming in on the movies with `roi > 400` provides a better view of how
the medians across the categories compare:

``` r
ggplot(data = bechdel90_13, mapping = aes(x = clean_test, y = roi, color = binary)) +
  geom_boxplot() +
  labs(title = "Return on investment vs. Bechdel test result",
       subtitle = "Zoomed to only include cases where ROI was less than 400", ## I wasn't exactly sure what you meant by this so I put what I thought worked based on the info given. 
       x = "Detailed Bechdel result",
       y = "Return on investment",
       color = "Binary Bechdel result") +
  coord_cartesian(ylim = c(0, 15))
```

![](The_Bechdel_Test_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

> What did you learn from the data analysis included in the document?

Movies where women have little to no substantial role generally receive
higher budgets, however movies that pass the binary bechdel test are
more likely to have higher returns of investment.
