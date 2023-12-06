Asian American Quality of Life Project
================

# Links

- [Dataset
  Source](https://data.austintexas.gov/dataset/Final-Report-of-the-Asian-American-Quality-of-Life/hc5t-p62z)

# Explanation of Project and Information about Data

This dataset that we are examining involves the fastest growing minority
group in the United States, Asian Americans. Within this dataset, the
broad term includes individuals having origins in the Far East,
Southeast Asia, or the Indian subcontinent. The Final Report of the
Asian American Quality of Life (AAQoL) comes from data provided by the
city of Austin, TX, and includes 2,609 rows of data in which each row is
an individual survey. There are 231 columns in the dataset, each column
providing a different piece of information about the respondents from a
question on the survey they were provided. Our project is evaluating the
quality of life of Asian Americans and factors that contribute to that.
Our data set will show why the quality of life may change based upon
different variables and why it is significant.

# Goals for Project

Some goals of our project include exploring what factors influence
economic well-being in Asian American populations. We examine
relationships between income and education, English proficiency and
religion. Using visualizations and data modeling we would like to
determine how these variables are related and if we can infer why see
differences between groups.

# Hypotheses

Hypothesis 2:

Income Bracket v.s. English Proficiency We predict that the level of
English proficiency in the Asian American population will be positively
related with their income bracket.

Hypothesis 3:

Income Bracket v.s. Religion We predict that income brackets will vary
based on religion and that individuals identifying as Hindu have the
highest percentage of top earners on average.

Hypothesis 4:

Employment Status v.s. Sense of Belonging We predict that those who are
employed full-time will have greater levels of belonging and those who
are unemployed will have lower levels of belonging.

Hypothesis 1:

Income Bracket v.s. Education vs. Ethnicity We predict a variation in
educational attainment amongst the different types of Asian ethnicities.
We also expect to see a positive general relationship between education
and income. \# Hypothesis 1 Visualization

# Hypothesis 2 Visualization

``` r
library(tidyverse)
library(extrafont)
```

    ## Registering fonts with R

``` r
loadfonts()


df_unfiltered <- read_csv("Final_Report_of_the_Asian_American_Quality_of_Life__AAQoL_.csv")
```

    ## New names:
    ## • `Other` -> `Other...17`
    ## • `Other` -> `Other...89`

    ## Rows: 2609 Columns: 231
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (190): Gender, Ethnicity, Marital Status, No One, Spouse, Children, Gran...
    ## dbl  (41): Survey ID, Age, Education Completed, Household Size, Grandparent,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# Data Wrangling, 2609 observations to 1620 observations
df <- df_unfiltered %>%
  filter(`US Born` == 'No') %>% # Filter for only immigrants, 256 rows removed.
  filter(('Full Time Employment' == 'Employed full time')|('Part Time Employment' == 'Employed part time')|(Retired=='Retired')) %>%
  filter(Income != '') %>% # Filter for only response with income information, 167 rows removed
  filter('English Speaking' != '') %>% # Filter for only responses with proficiency information, 6 rows removed
  select(1,Income,'Proficiency'='English Speaking',21:22,Retired,Age) %>% # Select only relevant variables
  na.omit() # Omit all cells with N/A, 263 rows removed.
  # TO-DO
  # Ask TA About how to join the employment columns

# Redefining Proficiency as a ordinal variable
df$Proficiency <- factor(df$Proficiency, ordered = TRUE,
                         levels = c("Not at all", "Not well", "Well", "Very well"))

# Primary Visualization
df %>% ggplot(aes(x = Income, fill = Proficiency)) +
              geom_bar(position = 'fill') +
              scale_fill_brewer(palette = 15) +
              theme(axis.text.x  = element_text(angle = 15, hjust = 0.7, size = 8, family = 'serif'),
                    axis.text.y  = element_text(angle = 90, hjust = 0.5, family = "serif"),
                    axis.title   = element_text(family = "serif"),
                    plot.title   = element_text(family = "serif", face = "bold"),
                    legend.title = element_text(family = "serif", face = "bold"),
                    legend.text  = element_text(family = "serif")) +
              labs(title = "Levels of English Proficiency Within Different Income Brackets",
                    x = "Income Brackets",
                    y = "Percentage")
```

![](Readme_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
# Redefining Age and Income for the second visualization
df <- df %>%
  mutate(Income5 = recode(Income, "$10,000 - $19,999" = "$10,000 - $29,999",
                                  "$20,000 - $29,999" = "$10,000 - $29,999",
                                  "$30,000 - $39,999" = "$30,000 - $49,999",
                                  "$40,000 - $49,999" = "$30,000 - $49,999",
                                  "$50,000 - $59,999" = "$50,000 - $69,999",
                                  "$60,000 - $69,999" = "$50,000 - $69,999")) %>% # Recode Income into 5 categories
  mutate(AgeCat = case_when(Age >= 18 & Age <= 30 ~ "Age 18-30 (n=302)",
                            Age >= 31 & Age <= 40 ~ "Age 31-40 (n=344)",
                            Age >= 41 & Age <= 55 ~ "Age 41-55 (n=414)",
                            Age >= 55 ~ "Age 55+ (n=403)",
                            TRUE ~ NA_character_)) # Create a categorical variable for age

# Make sure each each gorup is evenly sized
sum(df$AgeCat == "Age 18-30 (n=302)")
```

    ## [1] 0

``` r
sum(df$AgeCat == "Age 31-40 (n=344)")
```

    ## [1] 1

``` r
sum(df$AgeCat == "Age 41-55 (n=414)")
```

    ## [1] 5

``` r
sum(df$AgeCat == "Age 55+ (n=403)")
```

    ## [1] 245

# Hypothesis 3 Visualization

``` r
IncomeBasedOnReligion <- AAQoL %>% 
  select(Income, Religion) %>% 
  na.omit() %>% 
  group_by(Income, Religion) %>% 
  summarize(n = n())
```

    ## `summarise()` has grouped output by 'Income'. You can override using the
    ## `.groups` argument.

``` r
totalresp <- AAQoL %>% select(Income, Religion) %>% 
  na.omit() %>%
  group_by(Religion) %>%
  summarize(total_responses = n())

IncomeBasedOnReligionFinal <- merge(IncomeBasedOnReligion, totalresp, by = "Religion") %>% mutate(percentage = n / total_responses * 100)

ggplot(IncomeBasedOnReligionFinal, aes(x = Religion, y = percentage, fill = Income)) +
  geom_bar(stat = "identity") +
  labs(title = "Percentage of People in Each Religion Group by Income",
       x = "Income",
       y = "Percentage") +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  theme_minimal()
```

![](Readme_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
#V1 with labels

ggplot(IncomeBasedOnReligionFinal, aes(x = Religion, y = percentage, fill = Income)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = sprintf("%.1f%%", percentage)),
            position = position_stack(vjust = 0.5), # Adjust the position of labels
            size = 3, color = "white") +  # You can customize size and color of the labels
  labs(title = "Percentage of People in Each Religion Group by Income",
       x = "Income",
       y = "Percentage") +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  theme_minimal()
```

![](Readme_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
#V2

IncomeBasedOnReligionFinal1 <- IncomeBasedOnReligionFinal

Income_level <- c("$70,000 and over", "$60,000 - $69,999", "$50,000 - $59,999", "$40,000 - $49,999", "$30,000 - $39,999", "$20,000 - $29,999", "$10,000 - $19,999", "$0 - $9,999")

IncomeBasedOnReligionFinal1 <- IncomeBasedOnReligionFinal1 %>% mutate(Income = factor(Income, levels = Income_level))

ggplot(IncomeBasedOnReligionFinal1, aes(x = Religion, y = percentage, fill = Income)) +
  geom_bar(stat = "identity") +
  labs(title = "Percentage of People in Each Religion Group by Income",
       x = "Income",
       y = "Percentage") +
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  theme_minimal()
```

![](Readme_files/figure-gfm/unnamed-chunk-3-3.png)<!-- -->

\#Hypothesis 4 Visualization

``` r
BelongingAndFullTimeEmployment <- AAQoL %>% 
  select(Belonging, Full.Time.Employment, Part.Time.Employment, Student, Homemaker) %>% 
  na.omit() %>% mutate(Employment = if_else(Full.Time.Employment != "0", Full.Time.Employment, Part.Time.Employment)) %>% 
  mutate(Employment = if_else(Employment != "0", Employment, Student))  %>%
  mutate(Employment = if_else(Employment != "0", Employment, Homemaker)) %>%
  select(-Full.Time.Employment, -Part.Time.Employment, -Student, -Homemaker) %>% group_by(Belonging, Employment) %>%
  filter(Employment != 0)%>% 
  summarize(n = n())
```

    ## `summarise()` has grouped output by 'Belonging'. You can override using the
    ## `.groups` argument.

``` r
totalbelong <- BelongingAndFullTimeEmployment %>%
  group_by(Belonging) %>%
  summarize(total_belonging = sum(n))



BelongingAndEmployment <- merge(BelongingAndFullTimeEmployment, totalbelong, by = "Belonging") %>% mutate(percentage = n / total_belonging * 100)

# Plot using ggplot
ggplot(BelongingAndEmployment, aes(x = Belonging, y = percentage, fill = Employment)) +
  geom_bar(stat = "identity") +
  labs(title = "Level of belonging based on employment",
       x = "Belonging",
       y = "Percentage") + geom_text(aes(label = sprintf("%.1f%%", percentage)),
                                     position = position_stack(vjust = 0.5), # Adjust the position of labels
                                     size = 3, color = "white") + 
  scale_y_continuous(labels = scales::percent_format(scale = 1)) +
  theme_minimal()
```

![](Readme_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->
