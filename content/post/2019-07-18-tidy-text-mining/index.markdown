---
title: Tidy Text Mining
author: Yuan Du
date: '2019-07-19'
slug: tidy-text-mining
categories:
  - R
tags:
  - Tidytext
  - Tidyverse
subtitle: ''
summary: ''
authors: []
#lastmod: '2019-08-19T00:18:01-04:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---
Thanks to the short course at SDSS 2019, I learned how to do tf-idf to topic modeling and sentiment analysis by using tidytext taught by Julia Silge, author of [Text Mining with R](https://www.tidytextmining.com/) and Mara Averick. They did a great job on teaching the four hour class. I didn't expect to have so much covered in the short course. 

Here is an example that I used the method to analyze **A Tale of Two Cities** and **Great Expectations** by Charles Dickens by using sentiment analysis.

1. **Install the `tidytext` package for text mining.**

`install.packages("tidytext")`

2. **Read the book from gutenbergr package, after install gutenbergr package**

We can Downloading books by ID (98 and 1400) from [Project Gutenbergr](http://www.gutenberg.org/).


```r
library(gutenbergr)
library(dplyr)

book <-  gutenberg_download(c(98, 1400), meta_fields = "title") %>%
  group_by(title) %>%
  mutate(line = row_number()) %>%
  ungroup()

book
```

```
## # A tibble: 35,889 x 4
##    gutenberg_id text                             title                 line
##           <int> <chr>                            <chr>                <int>
##  1           98 A TALE OF TWO CITIES             A Tale of Two Cities     1
##  2           98 ""                               A Tale of Two Cities     2
##  3           98 A STORY OF THE FRENCH REVOLUTION A Tale of Two Cities     3
##  4           98 ""                               A Tale of Two Cities     4
##  5           98 By Charles Dickens               A Tale of Two Cities     5
##  6           98 ""                               A Tale of Two Cities     6
##  7           98 ""                               A Tale of Two Cities     7
##  8           98 CONTENTS                         A Tale of Two Cities     8
##  9           98 ""                               A Tale of Two Cities     9
## 10           98 ""                               A Tale of Two Cities    10
## # ... with 35,879 more rows
```
3. **Process books into chapters and words in tidy data form**

we need to restructure it as one-token-per-row format. As pre-processing, we divide these into chapters, use tidytext’s `unnest_tokens` to separate them into words, then remove `stop_word`s. We’re treating every chapter as a separate “document”, each with a name like *A Tale of Two cities* or *Great Expectations*.

```r
library(tidytext)
tidy_book <- book %>%
  unnest_tokens(word, text)

tidy_book
```

```
## # A tibble: 323,972 x 4
##    gutenberg_id title                 line word  
##           <int> <chr>                <int> <chr> 
##  1           98 A Tale of Two Cities     1 a     
##  2           98 A Tale of Two Cities     1 tale  
##  3           98 A Tale of Two Cities     1 of    
##  4           98 A Tale of Two Cities     1 two   
##  5           98 A Tale of Two Cities     1 cities
##  6           98 A Tale of Two Cities     3 a     
##  7           98 A Tale of Two Cities     3 story 
##  8           98 A Tale of Two Cities     3 of    
##  9           98 A Tale of Two Cities     3 the   
## 10           98 A Tale of Two Cities     3 french
## # ... with 323,962 more rows
```

```r
tidy_book <- tidy_book %>%
  anti_join(get_stopwords())

#We can also use count to find the most common words in all the book as a whole
tidy_book %>%
  count(word, sort = TRUE)
```

```
## # A tibble: 14,594 x 2
##    word       n
##    <chr>  <int>
##  1 said    2010
##  2 mr      1333
##  3 one      940
##  4 now      715
##  5 joe      698
##  6 upon     655
##  7 time     640
##  8 little   638
##  9 miss     616
## 10 know     613
## # ... with 14,584 more rows
```

4. **Sentiment analysis**

Sentiment analysis can be done as an inner join. Three sentiment lexicons are available via the `get_sentiments()` function. Let's examine how sentiment changes during each novel. Let's find a sentiment score for each word using the Bing lexicon, then count the number of positive and negative words in defined sections of each novel.


```r
library(tidyr)
get_sentiments("bing")
```

```
## # A tibble: 6,786 x 2
##    word        sentiment
##    <chr>       <chr>    
##  1 2-faces     negative 
##  2 abnormal    negative 
##  3 abolish     negative 
##  4 abominable  negative 
##  5 abominably  negative 
##  6 abominate   negative 
##  7 abomination negative 
##  8 abort       negative 
##  9 aborted     negative 
## 10 aborts      negative 
## # ... with 6,776 more rows
```

```r
sentiment <- tidy_book %>%
  inner_join(get_sentiments("bing"), by = "word") %>% 
  count(title, index = line %/% 80, sentiment) %>% 
  spread(sentiment, n, fill = 0) %>% 
  mutate(sentiment = positive - negative)

sentiment
```

```
## # A tibble: 450 x 5
##    title                index negative positive sentiment
##    <chr>                <dbl>    <dbl>    <dbl>     <dbl>
##  1 A Tale of Two Cities     0        5        9         4
##  2 A Tale of Two Cities     1       28       28         0
##  3 A Tale of Two Cities     2       27       15       -12
##  4 A Tale of Two Cities     3       17       10        -7
##  5 A Tale of Two Cities     4       13       12        -1
##  6 A Tale of Two Cities     5       19       18        -1
##  7 A Tale of Two Cities     6       24       16        -8
##  8 A Tale of Two Cities     7       24       19        -5
##  9 A Tale of Two Cities     8        9       27        18
## 10 A Tale of Two Cities     9       29       23        -6
## # ... with 440 more rows
```

**Now we can plot these sentiment scores across the plot trajectory of each novel.**

```r
library(ggplot2)

ggplot(sentiment, aes(index, sentiment, fill = title)) +
  geom_bar(stat = "identity", show.legend = FALSE) +
  facet_wrap(~title, ncol = 2, scales = "free_x")
```

<img src="/post/2019-07-18-tidy-text-mining/index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

It looks like that **A Table of Two Cities** has more negative emotions and **Great Expectations** is more balanced on emotions.

5. **Most common positive and negative words**

One advantage of having the data frame with both sentiment and word is that we can analyze word counts that contribute to each sentiment.


```r
bing_word_counts <- tidy_book %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE)

bing_word_counts
```

```
## # A tibble: 2,575 x 3
##    word   sentiment     n
##    <chr>  <chr>     <int>
##  1 miss   negative    616
##  2 like   positive    541
##  3 well   positive    483
##  4 good   positive    473
##  5 great  positive    360
##  6 better positive    221
##  7 right  positive    172
##  8 poor   negative    164
##  9 dark   negative    160
## 10 work   positive    152
## # ... with 2,565 more rows
```

This can be shown visually, and we can pipe straight into ggplot2 because of the way we are consistently using tools built for handling tidy data frames.

```r
bing_word_counts %>%
  filter(n > 100) %>%
  mutate(n = ifelse(sentiment == "negative", -n, n)) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col() +
  coord_flip() +
  labs(y = "Contribution to sentiment")
```

<img src="/post/2019-07-18-tidy-text-mining/index_files/figure-html/unnamed-chunk-4-1.png" width="672" />
This lets us spot an anomaly in the sentiment analysis; the word “miss” is coded as negative but it is used as a title for young, unmarried women in Jane Austen’s works. If it were appropriate for our purposes, we could easily add “miss” to a custom `stop-words` list using `bind_rows`.


```r
custom_stop_words <- bind_rows(get_stopwords(),
                               tibble(word = "miss",
                                          lexicon = "custom"))

tidy_book2 <- tidy_book %>%
  anti_join(custom_stop_words) %>%
  count(word, sort = TRUE)
```
6. **Wordclouds**

We’ve seen that this tidy text mining approach works well with ggplot2, but having our data in a tidy format is useful for other plots as well.

For example, consider the wordcloud package. Let’s look at the most common words in Charles Dickens’ two books as a whole again.


```r
library(wordcloud)

tidy_book %>%
  count(word) %>%
  with(wordcloud(word, n, max.words = 100))
```

<img src="/post/2019-07-18-tidy-text-mining/index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

In other functions, such as comparison.cloud, you may need to turn it into a matrix with `reshape2`’s acast. Let’s do the sentiment analysis to tag positive and negative words using an inner join, then find the most common positive and negative words. Until the step where we need to send the data to comparison.cloud, this can all be done with joins, piping, and dplyr because our data is in tidy format.

```r
library(reshape2)

tidy_book %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("#F8766D", "#00BFC4"),
                   max.words = 100)
```

<img src="/post/2019-07-18-tidy-text-mining/index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

*For "Converting to and from Document-Term Matrix and Corpus objects", You can visit [here](https://cran.r-project.org/web/packages/tidytext/vignettes/tidying_casting.html).*
