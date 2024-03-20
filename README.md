---
title: "palmer_penguins"
author: "Shining Chen"
date: "2024-03-19"
output: html_document
---

## INTRODUCTION

This notebook uses the famous [Palmer Penguins data package](https://allisonhorst.github.io/palmerpenguins/) collected by [Dr. Kristen Gorman](https://www.uaf.edu/cfos/people/faculty/detail/kristen-gorman.php) and the [Palmer Station, Antartica LTER](https://pallter.marine.rutgers.edu/).

The goal of this notebook is to provide beginners of R some useful examples of data transformation and visualisation. This notebook is divided into 4 sections:

1.  Install & load packages, libraries and data
2.  View data info & summary
3.  Transform, sort & filter data
4.  Plot data & annotate on plots

## INSTALL & LOAD PACKAGES, LIBRARIES, AND DATA

The 'palmerpenguins' package contains two datasets. One is called 'penguins', which is a simplified version of the raw data. And the other dataset is called 'penguins_raw'. We will load both, but we will only work with 'penguins' in this notebook.

In this notebook, we will need several libraries to help us perform data transformation and visualisation:

-   [tidyverse](https://www.tidyverse.org/packages/) contains a collection of useful libraries such as dplyr, lubridate, and ggplot2
-   [dplyr](https://dplyr.tidyverse.org/) stands for 'dataframe pliers' and is very useful in performing data transformation
-   [skimr](https://www.rdocumentation.org/packages/skimr/versions/2.1.5) for making a statistical summary of the data
-   [ggplot2](https://ggplot2.tidyverse.org/) stands for 'grammar of graphics plot 2' and is very useful in generating data visualisations

We will not install or load the following libraries and you will see that they are commented out in the code chunk below. However, they are generally very helpful should you decide to write more scripts:

-   [lubridate](https://www.rdocumentation.org/packages/lubridate/versions/1.9.3) for dates and times
-   [readr](https://readr.tidyverse.org/) for importing data
-   [here](https://here.r-lib.org/) that makes our lives easier while working with folders and files
-   [janitor](https://www.rdocumentation.org/packages/janitor/versions/2.2.0) that helps us improve data hygiene

```{r -- install & load, echo=TRUE}
install.packages("tidyverse")
library(tidyverse)
library(dplyr)
library(ggplot2)
# library(lubridate) -- this is a recommended library for data analysis in general
# library(readr) -- this is a recommended library for data analysis in general

install.packages("palmerpenguins")
library(palmerpenguins)
data(package = 'palmerpenguins')

install.packages('skimr')
library(skimr)

# install.packages("here")
# library(here) -- this is a recommended library for data analysis in general
#
# install.packages("janitor")
# library(janitor) -- this is a recommended library for data analysis in general
```

## VIEW DATA INFO & SUMMARY

To get familiar with the penguin data set, let's take a look at what columns there are currently:

```{r -- view all column names, echo=TRUE}
colnames(penguins)
```

*Note:* If the column names seem confusing, it is recommended to rename columns, for instance by using for instance the function `rename()` from the library 'dplyr,' or by using the library 'janitor' to clean up column names.

One way to get a detailed summary is by calling the function `skim_without_charts()` or `skim()` from the library 'skimr':

```{r -- get a detailed summary of the data by using skim}
skim_without_charts(penguins)
```

We see that there are some missing values in the dataset:

-   Column 'sex' has 11 missing pieces of data

-   Columns 'bill_length_mm', 'bill_depth_mm', 'flipper_length_mm', and 'body_mass_g' each has 2 missing pieces of data

-   Columns 'species', 'island' and 'year' have no missing data

## TRANSFORM, SORT & FILTER DATA

In the first example, suppose we have a **hypothesis that species and body weights are related**. So, we want to see **which penguin species has the largest average body weight and so on**.

Suppose that we want to list the **heaviest** species on the top. Then, we can use the `arrange()` function to order the list by **descending** 'body_mass_g' by placing a **negative (-) sign** inside the function.

After that, we can group penguins by species using the function `group_by()` from the library 'dplyr.' In addition, we can drop empty values by using the function `drop_na()`.

Finally, we can calculate the average body weight by using the the function `mean()` and asking for a summary by using the function `summarise()` from the library 'dplyr.'

```{r -- print average penguin weight by species, echo=TRUE}
penguins %>% 
  arrange(-body_mass_g) %>% 
  group_by(species) %>% 
  drop_na() %>% 
  summarise(mean_weight_g = mean(body_mass_g))
```

In this second example, suppose we have another **hypothesis that maybe body weights and island are related**. So we want to see **if the same species on different islands have about the same body weight**.

This time, we will group by the values of 2 columns: 'species' and 'island':

```{r -- print average penguin weight by species & island, echo=TRUE}
penguins %>% 
  group_by(species, island) %>% 
  drop_na() %>% 
  summarise(mean_weight_g = mean(body_mass_g), max_weight_g = max(body_mass_g), min_weight_g = min(body_mass_g), stddev_weight = sd(body_mass_g))
```

Suppose we decide that we only want to look at the **species Adelie** because **they are the only species that live on different islands**.

Then, we can filter this species by using the function `filter()` from the library 'dplyr,' sort it by 'island' and 'sex,' and save it as a new variable called 'adelie_penguins.'

Finally, we can print out a summary of this new dataset to see how many missing values there are and so on.

```{r -- filter and save adelie penguins as a variable, echo=TRUE}
adelie_penguins <- penguins %>% 
  filter(species == "Adelie") %>% 
  arrange(island, sex)
skim_without_charts(adelie_penguins)
```

## PLOT DATA & ANNOTATE ON PLOTS

In this final section, we are going to plot our data in different ways to discover insights.

First, suppose that we want to find out **if the body weight is related to the flipper length** for all 3 species. So we can plot the dataset 'penguins' by using the functions `ggplot()` and `geom_point()` to print a scatter plot from the library 'ggplot2.' The arguments in `geom_point()` allows us to show different species of the data points in different colors.

Then, we can add a fitted linear regression model for all species and color the model black by using the function `geom_smooth` from the same library.

Finally, as best practice, we should add a title that provides our insight to the plot by using the function `ggtitle()`.

```{r --body weight vs. flipper length, echo=TRUE}
ggplot(data = penguins, mapping = aes(x = body_mass_g, y = flipper_length_mm)) + 
  geom_point(mapping = aes(color = species)) + 
  geom_smooth(method = lm, color = 'black') +
  ggtitle('Heavier penguins have longer flippers')
```

Suppose now we want to find out **if the hypothesis that heavier penguins have longer flippers is TRUE for ALL species**. We might want to compare the fitted linear regression models by species side-by-side. The function `face_wrap()` can help us do just that.

```{r --compare linear regression models for body weight vs. flipper length by species, echo=TRUE}
ggplot(data = penguins, mapping = aes(x = body_mass_g, y = flipper_length_mm, color = species)) + 
  geom_point() + 
  geom_smooth(method = lm) + 
  facet_wrap(~species) +
  ggtitle('In all species, heavier penguins have longer flippers')
```

Suppose now we want to **direct our audience's attention to only the species Adelie** and the dataset that we created for this species. We can do so by plotting a bar chart by species using the function `geom_bar()` and **highlighting the Adelie penguins by using a different color** using the function `scale_fill_manual()`.

As usualy, we will add a title, but this time we also want a subtitle. The function `labs()` offers another way to add a title and a subtitle.

```{r --highlight the Adelie penguins by a bar chart, echo=TRUE}
ggplot(data = penguins, mapping = aes(x=species, fill = species)) + 
  geom_bar() + 
  scale_fill_manual(values = c("pink", "gray", "gray")) +
  labs(title = 'From now on, we will only examine the Adelie penguins', subtitle = 'Because only Adelie penguins live on all 3 islands')
```

When we proceed to analyse Adelie penguins, let's say we want to find out **if the relationship between body weight and flipper length is similar across sexes and/or islands**.  We could plot this out by using `facet_grid()` to facet the graphs based on these 2 columns.

First, let's drop the empty values in the dataset by calling `drop_na()` before proceeding to plotting.

Let's say we want to view the female and male data points more visually and to distinguish them from the previous dataset that contains all species.  Then, we can change the transparency of the data point colors by using the argument `alpha` and view overlapping data points better by calling the function `position_jitter()` as well.

```{r --drop empty values and plot adelie_penguins by sex and by island, echo=TRUE}
adelie_penguins %>%
  drop_na() %>%
  ggplot(mapping = aes(x = body_mass_g, y = flipper_length_mm)) + 
  geom_point(mapping = aes(color = sex), alpha = 0.5,  position = position_jitter()) +
  geom_smooth(method = lm, color = 'black') + 
  facet_grid(sex~island) +
  labs(title = 'Heavier Adelie penguins tend to have longer flppers', subtitle = 'Except when they are females on Torgersen')
```

Since we found out that female Adelie penguins on Torgersen is the only group among all Adelie penguins that rejects our hypothesis, we might want to find out **if this is related to the sex or the island**.

So we proceed to check this first withn the female penguin groups:

```{r --plot only female penguins by island and by species, echo=TRUE}
penguins %>%
  drop_na() %>%
  filter(sex == "female") %>%  
  ggplot(aes(x = body_mass_g, y = flipper_length_mm, colour = species)) +  
  geom_point(position = position_jitter()) + stat_smooth(method = "lm") +
  facet_grid(island~species) +
  labs(title = 'Heavier female penguins of other 2 species tend to have longer flippers', subtitle = 'Female Adelie on Torgersen are the only exceptions')
```

As we can see, female Adelie penguins on Torgersen are still the only exceptions across all female penguin groups.  That means **sex is not related to why female Adelie penguins on Torgersen reject our hypothesis**.  **Perhaps the environments on different islands are!**

So we proceed to check how male Adelie penguins on Torgersen are compared to other male penguin groups:

```{r --plot only male penguins by island and by species, echo=TRUE}
penguins %>%
  drop_na() %>%
  filter(sex == "male") %>%  
  ggplot(aes(x = body_mass_g, y = flipper_length_mm, colour = species)) +  
  geom_point(position = position_jitter()) + stat_smooth(method = "lm") +
  facet_grid(island~species) +
  labs(title = 'The environment on Torgersen might be related to why light penguins there often have long flippers (might not be a cause-and-effect relationship)', subtitle = 'Because male Adelie on Torgersen show the least correlation relationship of all male penguins groups')
```

As we can see, male Adelie on Torgersen show the least correlation relationship between body weight and flipper length compared to other male penguins groups.  So we can conclude safely that **The environment on Torgersen might be related to why light penguins there often have long flippers**.  Let's put a reminder to the audience though that this **does NOT necessarily mean that the island is a cause** of this different relationship between body weight and flipper length.
