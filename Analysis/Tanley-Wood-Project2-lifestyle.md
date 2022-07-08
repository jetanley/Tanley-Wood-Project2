Tanley-Wood-Project2
================
Jordan Tanley and Jonathan Wood
2022-07-05

# Introduction - Jonathan

## Data

The data in this analysis will be the [online news popularity
dataset](https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity).
This data has a set of features on articles from Mashable.com over a two
year period.

The goal of this project is to determine the number of shares (how many
times the article was shared over social media) the article has. We will
use this information to predict if an article can be popular by the
number of shares.

## Notable Variables

While there are 61 variables in the data set, we will not use all of
them for this project. The notable variables are the following:

-   “shares” - the number of shares the article has gotten over social
    media. This is the label or variable we want our models to predict
    for new articles
-   “data_channel_is” - a set of variables that tells if the article is
    in a particular category, such as business, sports, or lifestyle.
-   “weekday_is” - a set of variables that tells what day of the week
    the article was published on.
-   “num_keywords” - the number of keywords within the article
-   “num_images” - the number of images within the article
-   “num_videos” - the number of videos within the article

## Methods

Multiple methods will be used for this project to predict the number of
shares a new article can generate, including

-   Linear regression
-   Tree-based models
    -   Random forest
    -   Boosted tree

# Data - Jordan

In order to read in the data using a relative path, be sure to have the
data file saved in your working directory.

``` r
# read in the data
news <- read_csv("OnlineNewsPopularity/OnlineNewsPopularity.csv")
```

    ## Rows: 39644 Columns: 61
    ## ── Column specification ───────────────────────────
    ## Delimiter: ","
    ## chr  (1): url
    ## dbl (60): timedelta, n_tokens_title, n_tokens_c...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# sneek peek at the dataset
head(news)
```

``` r
# Creating a weekday variable (basically undoing the 7 dummy variables that came with the data) for EDA
news$weekday <- ifelse(news$weekday_is_friday == 1, "Friday",
                       ifelse(news$weekday_is_monday == 1, "Monday",
                              ifelse(news$weekday_is_tuesday == 1, "Tuesday",
                                     ifelse(news$weekday_is_wednesday == 1, "Wednesday",
                                            ifelse(news$weekday_is_thursday == 1, "Thursday",
                                                   ifelse(news$weekday_is_saturday == 1, "Saturday", 
                                                          "Sunday"))))))
```

Next, let’s subset the data so that we can only look at the data channel
of interest. We will look at articles with the “Social Media” data
channel.

``` r
# Subset the data to  one of the parameterized data channels and drop unnecessary variables
chan <- paste0("data_channel_is_", params$channel)

print(chan)
```

    ## [1] "data_channel_is_lifestyle"

``` r
filtered_channel <- news %>% 
                as_tibble() %>% 
                filter(news[chan] == 1) %>% 
                select(-c(url, timedelta))

# take a peek at the data
filtered_channel %>%
  select(ends_with(chan))
```

# Summarizations - Both (3 plots each)

For the numerical summaries, we can look at several aspects. Contingency
tables allow us to examine frequencies of categorical variables. The
first output below, for example, shows the counts for each weekday.
Similarly, the fifth table outputted shows the frequencies of number of
tokens in the article content. Another set of summary statistics to look
at are the 5 Number Summaries. These provide the minmum, 1st quantile,
median, 3rd quantile, and maximum for a particular variable.
Additionally, it may also be helful to look at the average. These are
helpful in determining the skewness (if mean = median vs. mean \< or \>
median) and helps in looking for outliers (anything outside (Q3 - Q1)1.5
from the median is generally considered an outlier). Below, the 5 Number
summaries (plus mean) are shown for Shares, Number of words in the
content, Number of words in the content for the upper quantile of
Shares, number of images in the article, number of videos in the
article, positive word rate, and negative word rate.

``` r
# Contingency table of frequencies for days of the week, added caption for clarity
kable(table(filtered_channel$weekday), 
      col.names = c("Weekday", "Frequency"), 
      caption = "Contingency table of frequencies for days of the week")
```

| Weekday   | Frequency |
|:----------|----------:|
| Friday    |       305 |
| Monday    |       322 |
| Saturday  |       182 |
| Sunday    |       210 |
| Thursday  |       358 |
| Tuesday   |       334 |
| Wednesday |       388 |

Contingency table of frequencies for days of the week

``` r
# Numerical Summary of Shares, added caption for clarity
filtered_channel %>% summarise(Minimum = min(shares), 
                          Q1 = quantile(shares, prob = 0.25), 
                          Average = mean(shares), 
                          Median = median(shares), 
                          Q3 = quantile(shares, prob = 0.75), 
                          Maximum = max(shares)) %>% 
                kable(caption = "Numerical Summary of Shares")
```

| Minimum |   Q1 |  Average | Median |   Q3 | Maximum |
|--------:|-----:|---------:|-------:|-----:|--------:|
|      28 | 1100 | 3682.123 |   1700 | 3250 |  208300 |

Numerical Summary of Shares

``` r
# Numerical Summary of Number of words in the content, added caption for clarity
filtered_channel %>% summarise(Minimum = min(n_tokens_content), 
                          Q1 = quantile(n_tokens_content, prob = 0.25), 
                          Average = mean(n_tokens_content), 
                          Median = median(n_tokens_content), 
                          Q3 = quantile(n_tokens_content, prob = 0.75), 
                          Maximum = max(n_tokens_content)) %>% 
                kable(caption = "Numerical Summary of Number of words in the content")
```

| Minimum |    Q1 |  Average | Median |  Q3 | Maximum |
|--------:|------:|---------:|-------:|----:|--------:|
|       0 | 308.5 | 621.3273 |    502 | 795 |    8474 |

Numerical Summary of Number of words in the content

``` r
# Numerical Summary of Number of words in the content for the upper quantile of Shares, added caption for clarity
filtered_channel %>% filter(shares > quantile(shares, prob = 0.75)) %>%
                summarise(Minimum = min(n_tokens_content), 
                          Q1 = quantile(n_tokens_content, prob = 0.25), 
                          Average = mean(n_tokens_content), 
                          Median = median(n_tokens_content), 
                          Q3 = quantile(n_tokens_content, prob = 0.75), 
                          Maximum = max(n_tokens_content)) %>% 
                kable(caption = "Numerical Summary of Number of words in the content for the upper quantile of Shares")
```

| Minimum |  Q1 |  Average | Median |  Q3 | Maximum |
|--------:|----:|---------:|-------:|----:|--------:|
|       0 | 306 | 679.1752 |    505 | 883 |    8474 |

Numerical Summary of Number of words in the content for the upper
quantile of Shares

``` r
kable(table(filtered_channel$n_tokens_content),
  col.names = c("Tokens", "Frequency"), 
  caption = "Contingency table of frequencies for number of tokens in the article content")
```

| Tokens | Frequency |
|:-------|----------:|
| 0      |        22 |
| 68     |         1 |
| 77     |         1 |
| 81     |         2 |
| 85     |         1 |
| 89     |         1 |
| 91     |         2 |
| 96     |         1 |
| 97     |         1 |
| 98     |         1 |
| 99     |         2 |
| 101    |         1 |
| 102    |         1 |
| 103    |         1 |
| 105    |         2 |
| 106    |         1 |
| 107    |         1 |
| 110    |         2 |
| 112    |         1 |
| 113    |         2 |
| 114    |         2 |
| 115    |         3 |
| 116    |         5 |
| 117    |         2 |
| 118    |         1 |
| 120    |         1 |
| 121    |         3 |
| 123    |         6 |
| 124    |         1 |
| 126    |         1 |
| 128    |         1 |
| 129    |         1 |
| 130    |         1 |
| 131    |         2 |
| 132    |         1 |
| 135    |         1 |
| 137    |         3 |
| 138    |         3 |
| 139    |         2 |
| 140    |         2 |
| 141    |         3 |
| 143    |         1 |
| 144    |         2 |
| 145    |         1 |
| 146    |         2 |
| 147    |         3 |
| 149    |         4 |
| 150    |         3 |
| 151    |         1 |
| 153    |         1 |
| 154    |         3 |
| 155    |         1 |
| 156    |         2 |
| 157    |         2 |
| 158    |         3 |
| 159    |         4 |
| 160    |         1 |
| 161    |         2 |
| 163    |         3 |
| 165    |         1 |
| 166    |         2 |
| 167    |         1 |
| 169    |         4 |
| 171    |         4 |
| 172    |         1 |
| 173    |         3 |
| 174    |         4 |
| 175    |         4 |
| 178    |         3 |
| 179    |         3 |
| 180    |         2 |
| 181    |         1 |
| 182    |         2 |
| 183    |         2 |
| 184    |         3 |
| 187    |         3 |
| 188    |         1 |
| 189    |         2 |
| 191    |         2 |
| 192    |         1 |
| 193    |         2 |
| 194    |         2 |
| 195    |         5 |
| 196    |         1 |
| 197    |         5 |
| 198    |         2 |
| 199    |         4 |
| 200    |         2 |
| 202    |         4 |
| 203    |         5 |
| 204    |         5 |
| 205    |         1 |
| 206    |         4 |
| 207    |         3 |
| 208    |         1 |
| 209    |         2 |
| 210    |         6 |
| 211    |         3 |
| 212    |         1 |
| 213    |         3 |
| 214    |         6 |
| 215    |         4 |
| 216    |         4 |
| 217    |         3 |
| 218    |         2 |
| 219    |         1 |
| 221    |         2 |
| 222    |         7 |
| 223    |         2 |
| 224    |         3 |
| 225    |         1 |
| 226    |         4 |
| 228    |         2 |
| 229    |         3 |
| 230    |         4 |
| 231    |         2 |
| 232    |         3 |
| 234    |         2 |
| 235    |         4 |
| 236    |         2 |
| 237    |         4 |
| 238    |         2 |
| 239    |         3 |
| 240    |         4 |
| 241    |         3 |
| 242    |         3 |
| 243    |         7 |
| 244    |         1 |
| 246    |         3 |
| 247    |         4 |
| 249    |         6 |
| 250    |         1 |
| 251    |         1 |
| 252    |         3 |
| 253    |         2 |
| 254    |         4 |
| 255    |         2 |
| 256    |         7 |
| 257    |         1 |
| 258    |         6 |
| 259    |         2 |
| 260    |         1 |
| 261    |         7 |
| 262    |         2 |
| 263    |         3 |
| 264    |         2 |
| 265    |         6 |
| 266    |         6 |
| 267    |         4 |
| 268    |         6 |
| 269    |         4 |
| 270    |         5 |
| 271    |         3 |
| 272    |         5 |
| 273    |         5 |
| 275    |         1 |
| 276    |         9 |
| 277    |         3 |
| 278    |         3 |
| 279    |         4 |
| 280    |         2 |
| 281    |         2 |
| 282    |         4 |
| 283    |         5 |
| 285    |         1 |
| 286    |         2 |
| 287    |         3 |
| 289    |         2 |
| 290    |         1 |
| 291    |         6 |
| 292    |         5 |
| 293    |         4 |
| 294    |         5 |
| 295    |         2 |
| 296    |         1 |
| 297    |         5 |
| 298    |         2 |
| 299    |         3 |
| 300    |         4 |
| 301    |         2 |
| 303    |         1 |
| 304    |         1 |
| 305    |         4 |
| 306    |         3 |
| 307    |         2 |
| 308    |         3 |
| 309    |         3 |
| 310    |         2 |
| 311    |         2 |
| 312    |         3 |
| 313    |         4 |
| 314    |         1 |
| 315    |         5 |
| 316    |         3 |
| 317    |         4 |
| 318    |         6 |
| 319    |         3 |
| 320    |         3 |
| 322    |         3 |
| 323    |         4 |
| 324    |         1 |
| 325    |         7 |
| 326    |         5 |
| 327    |         3 |
| 328    |         2 |
| 329    |         2 |
| 330    |         3 |
| 331    |         4 |
| 332    |         6 |
| 333    |         1 |
| 334    |         4 |
| 335    |         5 |
| 336    |         3 |
| 337    |         5 |
| 338    |         5 |
| 339    |         3 |
| 340    |         4 |
| 341    |         2 |
| 342    |         8 |
| 344    |         4 |
| 345    |         3 |
| 346    |         2 |
| 347    |         4 |
| 348    |         4 |
| 349    |         2 |
| 350    |         4 |
| 351    |         2 |
| 352    |         5 |
| 353    |         4 |
| 354    |         5 |
| 355    |         4 |
| 356    |         3 |
| 357    |         2 |
| 358    |         2 |
| 359    |         1 |
| 360    |         3 |
| 361    |         2 |
| 362    |         3 |
| 363    |         1 |
| 364    |         3 |
| 365    |         3 |
| 366    |         3 |
| 367    |         5 |
| 368    |         5 |
| 369    |         1 |
| 370    |         4 |
| 371    |         1 |
| 372    |         1 |
| 373    |         1 |
| 374    |         2 |
| 375    |         2 |
| 376    |         4 |
| 377    |         1 |
| 378    |         5 |
| 379    |         5 |
| 380    |         3 |
| 381    |         3 |
| 382    |         5 |
| 383    |         5 |
| 385    |         1 |
| 386    |         1 |
| 387    |         3 |
| 388    |         3 |
| 389    |         1 |
| 390    |         2 |
| 391    |         2 |
| 393    |         3 |
| 394    |         4 |
| 395    |         5 |
| 396    |         2 |
| 397    |         5 |
| 398    |         4 |
| 399    |         6 |
| 400    |         3 |
| 401    |         2 |
| 402    |         3 |
| 403    |         3 |
| 404    |         2 |
| 405    |         3 |
| 406    |         3 |
| 407    |         3 |
| 408    |         4 |
| 409    |         6 |
| 410    |         4 |
| 411    |         4 |
| 412    |         4 |
| 413    |         3 |
| 414    |         4 |
| 415    |         2 |
| 416    |         3 |
| 417    |         2 |
| 418    |         3 |
| 419    |         2 |
| 420    |         3 |
| 421    |         1 |
| 422    |         1 |
| 423    |         3 |
| 424    |         1 |
| 425    |         1 |
| 427    |         2 |
| 428    |         4 |
| 429    |         3 |
| 430    |         3 |
| 432    |         2 |
| 434    |         4 |
| 436    |         2 |
| 437    |         1 |
| 438    |         1 |
| 439    |         2 |
| 440    |         5 |
| 441    |         4 |
| 442    |         2 |
| 443    |         2 |
| 444    |         1 |
| 445    |         4 |
| 446    |         6 |
| 447    |         2 |
| 448    |        10 |
| 451    |         1 |
| 452    |         3 |
| 453    |         5 |
| 454    |         2 |
| 455    |         1 |
| 456    |         2 |
| 457    |         6 |
| 458    |         2 |
| 459    |         4 |
| 460    |         1 |
| 461    |         2 |
| 463    |         3 |
| 464    |         2 |
| 465    |         1 |
| 466    |         2 |
| 467    |         1 |
| 468    |         3 |
| 469    |         1 |
| 470    |         1 |
| 471    |         3 |
| 473    |         2 |
| 475    |         1 |
| 476    |         2 |
| 477    |         1 |
| 478    |         2 |
| 480    |         4 |
| 481    |         2 |
| 482    |         2 |
| 483    |         3 |
| 484    |         1 |
| 485    |         5 |
| 486    |         4 |
| 487    |         3 |
| 488    |         2 |
| 489    |         2 |
| 490    |         4 |
| 491    |         2 |
| 492    |         1 |
| 494    |         1 |
| 495    |         2 |
| 496    |         2 |
| 497    |         2 |
| 498    |         4 |
| 499    |         4 |
| 500    |         2 |
| 501    |         2 |
| 502    |         2 |
| 503    |         4 |
| 504    |         2 |
| 505    |         4 |
| 506    |         3 |
| 507    |         1 |
| 508    |         2 |
| 509    |         3 |
| 510    |         4 |
| 511    |         4 |
| 512    |         1 |
| 513    |         3 |
| 514    |         4 |
| 516    |         3 |
| 517    |         4 |
| 518    |         4 |
| 519    |         1 |
| 521    |         2 |
| 522    |         2 |
| 524    |         1 |
| 525    |         1 |
| 526    |         1 |
| 527    |         4 |
| 528    |         1 |
| 529    |         4 |
| 530    |         5 |
| 531    |         2 |
| 532    |         1 |
| 533    |         2 |
| 534    |         3 |
| 535    |         6 |
| 536    |         3 |
| 537    |         1 |
| 538    |         2 |
| 539    |         3 |
| 540    |         2 |
| 541    |         3 |
| 542    |         2 |
| 543    |         2 |
| 544    |         4 |
| 545    |         2 |
| 546    |         1 |
| 547    |         1 |
| 548    |         7 |
| 549    |         5 |
| 550    |         2 |
| 551    |         2 |
| 552    |         1 |
| 554    |         3 |
| 555    |         3 |
| 556    |         1 |
| 557    |         4 |
| 558    |         1 |
| 560    |         2 |
| 561    |         2 |
| 562    |         3 |
| 563    |         1 |
| 564    |         1 |
| 565    |         3 |
| 566    |         1 |
| 567    |         3 |
| 568    |         4 |
| 569    |         2 |
| 570    |         3 |
| 571    |         6 |
| 572    |         2 |
| 573    |         3 |
| 574    |         2 |
| 575    |         3 |
| 576    |         4 |
| 577    |         2 |
| 578    |         3 |
| 579    |         1 |
| 580    |         3 |
| 581    |         1 |
| 582    |         3 |
| 583    |         3 |
| 584    |         3 |
| 586    |         1 |
| 587    |         1 |
| 588    |         2 |
| 589    |         4 |
| 590    |         2 |
| 591    |         4 |
| 592    |         2 |
| 593    |         1 |
| 594    |         3 |
| 595    |         2 |
| 596    |         2 |
| 598    |         1 |
| 599    |         3 |
| 600    |         1 |
| 601    |         3 |
| 602    |         2 |
| 603    |         1 |
| 604    |         2 |
| 605    |         1 |
| 606    |         2 |
| 607    |         1 |
| 608    |         1 |
| 609    |         2 |
| 610    |         2 |
| 611    |         3 |
| 612    |         3 |
| 613    |         2 |
| 614    |         3 |
| 615    |         3 |
| 616    |         1 |
| 619    |         2 |
| 620    |         2 |
| 621    |         1 |
| 622    |         2 |
| 623    |         1 |
| 624    |         2 |
| 625    |         2 |
| 626    |         3 |
| 627    |         2 |
| 628    |         2 |
| 629    |         1 |
| 630    |         1 |
| 631    |         1 |
| 632    |         2 |
| 633    |         1 |
| 634    |         1 |
| 635    |         1 |
| 636    |         3 |
| 637    |         3 |
| 638    |         4 |
| 639    |         5 |
| 640    |         2 |
| 641    |         2 |
| 642    |         1 |
| 643    |         1 |
| 645    |         3 |
| 646    |         2 |
| 647    |         2 |
| 651    |         2 |
| 652    |         2 |
| 653    |         2 |
| 654    |         1 |
| 655    |         1 |
| 659    |         2 |
| 660    |         3 |
| 661    |         3 |
| 662    |         1 |
| 663    |         5 |
| 664    |         2 |
| 665    |         1 |
| 666    |         1 |
| 667    |         2 |
| 668    |         1 |
| 670    |         3 |
| 671    |         2 |
| 672    |         3 |
| 673    |         3 |
| 674    |         1 |
| 675    |         3 |
| 676    |         1 |
| 677    |         2 |
| 679    |         1 |
| 680    |         1 |
| 681    |         1 |
| 682    |         1 |
| 683    |         4 |
| 687    |         2 |
| 689    |         2 |
| 690    |         3 |
| 691    |         3 |
| 692    |         1 |
| 695    |         4 |
| 696    |         2 |
| 697    |         2 |
| 698    |         4 |
| 699    |         3 |
| 702    |         3 |
| 703    |         1 |
| 704    |         2 |
| 706    |         1 |
| 707    |         4 |
| 708    |         3 |
| 710    |         1 |
| 712    |         3 |
| 713    |         2 |
| 715    |         2 |
| 716    |         3 |
| 717    |         2 |
| 719    |         3 |
| 720    |         3 |
| 723    |         2 |
| 724    |         1 |
| 725    |         3 |
| 726    |         3 |
| 727    |         2 |
| 728    |         1 |
| 729    |         1 |
| 730    |         2 |
| 731    |         3 |
| 732    |         5 |
| 733    |         1 |
| 734    |         4 |
| 736    |         1 |
| 739    |         3 |
| 743    |         2 |
| 744    |         1 |
| 745    |         2 |
| 746    |         1 |
| 747    |         1 |
| 748    |         1 |
| 749    |         3 |
| 752    |         1 |
| 754    |         2 |
| 755    |         1 |
| 756    |         1 |
| 757    |         2 |
| 759    |         2 |
| 760    |         1 |
| 762    |         5 |
| 763    |         3 |
| 764    |         1 |
| 768    |         4 |
| 769    |         2 |
| 771    |         1 |
| 772    |         1 |
| 773    |         1 |
| 774    |         3 |
| 776    |         2 |
| 781    |         1 |
| 783    |         2 |
| 784    |         2 |
| 785    |         1 |
| 788    |         1 |
| 791    |         1 |
| 792    |         2 |
| 793    |         3 |
| 794    |         2 |
| 795    |         2 |
| 796    |         1 |
| 799    |         1 |
| 801    |         2 |
| 802    |         1 |
| 803    |         2 |
| 804    |         2 |
| 806    |         2 |
| 807    |         3 |
| 808    |         2 |
| 809    |         3 |
| 810    |         1 |
| 811    |         1 |
| 812    |         2 |
| 814    |         2 |
| 816    |         1 |
| 817    |         2 |
| 818    |         2 |
| 819    |         3 |
| 820    |         1 |
| 822    |         1 |
| 823    |         1 |
| 824    |         3 |
| 825    |         1 |
| 826    |         1 |
| 827    |         1 |
| 828    |         2 |
| 830    |         2 |
| 831    |         2 |
| 832    |         2 |
| 837    |         1 |
| 839    |         1 |
| 840    |         2 |
| 841    |         1 |
| 842    |         1 |
| 843    |         1 |
| 847    |         2 |
| 851    |         2 |
| 852    |         1 |
| 854    |         2 |
| 855    |         2 |
| 857    |         2 |
| 858    |         1 |
| 859    |         5 |
| 860    |         2 |
| 861    |         2 |
| 863    |         7 |
| 864    |         2 |
| 865    |         1 |
| 868    |         1 |
| 869    |         1 |
| 870    |         1 |
| 871    |         1 |
| 872    |         2 |
| 873    |         3 |
| 874    |         1 |
| 877    |         1 |
| 878    |         2 |
| 879    |         1 |
| 880    |         1 |
| 881    |         2 |
| 883    |         3 |
| 886    |         1 |
| 888    |         1 |
| 889    |         1 |
| 890    |         2 |
| 891    |         2 |
| 893    |         2 |
| 894    |         1 |
| 895    |         4 |
| 897    |         1 |
| 899    |         4 |
| 900    |         1 |
| 901    |         1 |
| 902    |         2 |
| 903    |         1 |
| 906    |         2 |
| 907    |         3 |
| 908    |         1 |
| 909    |         1 |
| 910    |         1 |
| 911    |         2 |
| 912    |         2 |
| 913    |         2 |
| 914    |         1 |
| 916    |         1 |
| 917    |         3 |
| 918    |         1 |
| 919    |         2 |
| 920    |         3 |
| 921    |         1 |
| 926    |         1 |
| 927    |         1 |
| 928    |         2 |
| 930    |         2 |
| 931    |         1 |
| 932    |         3 |
| 934    |         1 |
| 935    |         3 |
| 937    |         1 |
| 939    |         2 |
| 940    |         2 |
| 941    |         1 |
| 942    |         1 |
| 947    |         1 |
| 948    |         3 |
| 950    |         1 |
| 953    |         2 |
| 955    |         3 |
| 957    |         1 |
| 958    |         1 |
| 960    |         2 |
| 962    |         1 |
| 963    |         1 |
| 964    |         1 |
| 965    |         3 |
| 968    |         1 |
| 970    |         2 |
| 971    |         1 |
| 972    |         2 |
| 974    |         1 |
| 977    |         4 |
| 983    |         3 |
| 984    |         1 |
| 986    |         1 |
| 988    |         1 |
| 992    |         2 |
| 999    |         2 |
| 1000   |         1 |
| 1002   |         1 |
| 1005   |         1 |
| 1006   |         1 |
| 1007   |         2 |
| 1009   |         1 |
| 1010   |         1 |
| 1011   |         1 |
| 1013   |         1 |
| 1014   |         1 |
| 1015   |         2 |
| 1018   |         2 |
| 1019   |         1 |
| 1020   |         3 |
| 1021   |         2 |
| 1022   |         2 |
| 1023   |         1 |
| 1024   |         3 |
| 1026   |         1 |
| 1028   |         3 |
| 1029   |         1 |
| 1030   |         1 |
| 1034   |         4 |
| 1035   |         1 |
| 1037   |         1 |
| 1041   |         2 |
| 1044   |         1 |
| 1045   |         2 |
| 1046   |         1 |
| 1050   |         1 |
| 1053   |         2 |
| 1058   |         1 |
| 1059   |         1 |
| 1060   |         2 |
| 1064   |         1 |
| 1067   |         1 |
| 1069   |         1 |
| 1074   |         1 |
| 1075   |         1 |
| 1076   |         1 |
| 1079   |         1 |
| 1080   |         1 |
| 1084   |         2 |
| 1085   |         1 |
| 1089   |         1 |
| 1091   |         1 |
| 1092   |         1 |
| 1094   |         1 |
| 1095   |         1 |
| 1096   |         1 |
| 1099   |         2 |
| 1100   |         1 |
| 1103   |         2 |
| 1105   |         3 |
| 1106   |         1 |
| 1107   |         1 |
| 1110   |         1 |
| 1111   |         1 |
| 1117   |         1 |
| 1118   |         1 |
| 1119   |         1 |
| 1122   |         1 |
| 1127   |         1 |
| 1132   |         1 |
| 1133   |         2 |
| 1134   |         1 |
| 1135   |         1 |
| 1138   |         1 |
| 1140   |         2 |
| 1141   |         3 |
| 1144   |         2 |
| 1145   |         1 |
| 1146   |         1 |
| 1147   |         2 |
| 1148   |         1 |
| 1151   |         1 |
| 1153   |         1 |
| 1158   |         2 |
| 1159   |         1 |
| 1160   |         1 |
| 1167   |         1 |
| 1168   |         2 |
| 1172   |         1 |
| 1175   |         1 |
| 1177   |         1 |
| 1180   |         1 |
| 1181   |         1 |
| 1182   |         1 |
| 1183   |         2 |
| 1184   |         2 |
| 1185   |         1 |
| 1190   |         1 |
| 1191   |         2 |
| 1197   |         1 |
| 1198   |         1 |
| 1200   |         2 |
| 1205   |         1 |
| 1210   |         1 |
| 1215   |         2 |
| 1223   |         2 |
| 1224   |         1 |
| 1225   |         1 |
| 1226   |         1 |
| 1227   |         2 |
| 1229   |         1 |
| 1238   |         1 |
| 1239   |         1 |
| 1240   |         1 |
| 1245   |         1 |
| 1247   |         1 |
| 1248   |         1 |
| 1249   |         1 |
| 1256   |         1 |
| 1257   |         1 |
| 1266   |         2 |
| 1267   |         1 |
| 1271   |         1 |
| 1278   |         1 |
| 1279   |         2 |
| 1281   |         1 |
| 1282   |         1 |
| 1284   |         1 |
| 1285   |         2 |
| 1286   |         1 |
| 1287   |         1 |
| 1293   |         1 |
| 1294   |         2 |
| 1295   |         1 |
| 1303   |         1 |
| 1307   |         1 |
| 1311   |         2 |
| 1314   |         1 |
| 1316   |         3 |
| 1318   |         1 |
| 1322   |         1 |
| 1323   |         1 |
| 1326   |         1 |
| 1327   |         1 |
| 1331   |         1 |
| 1338   |         1 |
| 1344   |         1 |
| 1347   |         1 |
| 1348   |         1 |
| 1349   |         1 |
| 1359   |         1 |
| 1361   |         1 |
| 1365   |         1 |
| 1368   |         1 |
| 1370   |         1 |
| 1375   |         1 |
| 1376   |         1 |
| 1378   |         2 |
| 1381   |         1 |
| 1388   |         1 |
| 1393   |         1 |
| 1398   |         1 |
| 1401   |         1 |
| 1411   |         1 |
| 1416   |         1 |
| 1418   |         1 |
| 1427   |         1 |
| 1434   |         1 |
| 1435   |         1 |
| 1437   |         1 |
| 1441   |         1 |
| 1444   |         1 |
| 1445   |         1 |
| 1447   |         1 |
| 1460   |         1 |
| 1471   |         2 |
| 1477   |         1 |
| 1487   |         2 |
| 1494   |         1 |
| 1498   |         1 |
| 1506   |         1 |
| 1508   |         1 |
| 1519   |         1 |
| 1524   |         1 |
| 1525   |         1 |
| 1528   |         1 |
| 1530   |         1 |
| 1539   |         1 |
| 1542   |         2 |
| 1546   |         2 |
| 1547   |         1 |
| 1549   |         1 |
| 1556   |         1 |
| 1557   |         1 |
| 1559   |         1 |
| 1572   |         1 |
| 1578   |         1 |
| 1579   |         1 |
| 1582   |         1 |
| 1587   |         1 |
| 1601   |         1 |
| 1611   |         1 |
| 1613   |         1 |
| 1618   |         1 |
| 1643   |         1 |
| 1671   |         1 |
| 1672   |         2 |
| 1701   |         1 |
| 1727   |         1 |
| 1752   |         1 |
| 1784   |         2 |
| 1798   |         1 |
| 1809   |         1 |
| 1821   |         1 |
| 1822   |         1 |
| 1834   |         1 |
| 1866   |         1 |
| 1889   |         1 |
| 1901   |         1 |
| 1906   |         1 |
| 1939   |         2 |
| 1963   |         1 |
| 1971   |         1 |
| 1976   |         1 |
| 1987   |         1 |
| 2034   |         1 |
| 2055   |         1 |
| 2061   |         1 |
| 2071   |         1 |
| 2119   |         1 |
| 2147   |         1 |
| 2168   |         1 |
| 2182   |         1 |
| 2199   |         1 |
| 2301   |         1 |
| 2392   |         1 |
| 2405   |         1 |
| 2448   |         1 |
| 2454   |         1 |
| 2509   |         1 |
| 2542   |         1 |
| 2571   |         1 |
| 2781   |         1 |
| 2873   |         1 |
| 2907   |         1 |
| 3007   |         1 |
| 3083   |         1 |
| 3233   |         1 |
| 3727   |         1 |
| 4089   |         1 |
| 4130   |         1 |
| 7004   |         1 |
| 7034   |         1 |
| 7185   |         1 |
| 7413   |         1 |
| 7764   |         1 |
| 8474   |         1 |

Contingency table of frequencies for number of tokens in the article
content

``` r
# Summarizing the number of images in the article
filtered_channel %>% 
  summarise(Minimum = min(num_imgs), 
      Q1 = quantile(num_imgs, prob = 0.25), 
      Average = mean(num_imgs), 
      Median = median(num_imgs), 
      Q3 = quantile(num_imgs, prob = 0.75), 
      Maximum = max(num_imgs)) %>% 
  kable(caption = "Numerical summary of number of images in an article")
```

| Minimum |  Q1 |  Average | Median |  Q3 | Maximum |
|--------:|----:|---------:|-------:|----:|--------:|
|       0 |   1 | 4.904717 |      1 |   8 |     111 |

Numerical summary of number of images in an article

``` r
# Summarizing the number of videos in the article
filtered_channel %>% 
  summarise(Minimum = min(num_videos), 
      Q1 = quantile(num_videos, prob = 0.25), 
      Average = mean(num_videos), 
      Median = median(num_videos), 
      Q3 = quantile(num_videos, prob = 0.75), 
      Maximum = max(num_videos)) %>% 
  kable(caption = "Numerical summary of number of videos in an article")
```

| Minimum |  Q1 |   Average | Median |  Q3 | Maximum |
|--------:|----:|----------:|-------:|----:|--------:|
|       0 |   0 | 0.4749881 |      0 |   0 |      50 |

Numerical summary of number of videos in an article

``` r
# Summarizing the number of positive word rate
filtered_channel %>% 
  summarise(Minimum = min(rate_positive_words), 
      Q1 = quantile(rate_positive_words, prob = 0.25), 
      Average = mean(rate_positive_words), 
      Median = median(rate_positive_words), 
      Q3 = quantile(rate_positive_words, prob = 0.75), 
      Maximum = max(rate_positive_words)) %>% 
  kable(caption = "Numerical Summary of the rate of positive words in an article")
```

| Minimum |        Q1 |   Average |    Median |     Q3 | Maximum |
|--------:|----------:|----------:|----------:|-------:|--------:|
|       0 | 0.6624941 | 0.7226337 | 0.7377049 | 0.8125 |       1 |

Numerical Summary of the rate of positive words in an article

``` r
# Summarizing the number of negative word rate
filtered_channel %>% 
  summarise(Minimum = min(rate_negative_words), 
      Q1 = quantile(rate_negative_words, prob = 0.25), 
      Average = mean(rate_negative_words), 
      Median = median(rate_negative_words), 
      Q3 = quantile(rate_negative_words, prob = 0.75), 
      Maximum = max(rate_negative_words)) %>% 
  kable(caption = "Numerical Summary of the rate of negative words in an article")
```

| Minimum |        Q1 |   Average |    Median |        Q3 | Maximum |
|--------:|----------:|----------:|----------:|----------:|--------:|
|       0 | 0.1836735 | 0.2668851 | 0.2580645 | 0.3333333 |       1 |

Numerical Summary of the rate of negative words in an article

The graphical summaries more dramatically show the trends in the data,
including skewness and outliers. The boxplots below show a visual
representation of the 5 Number summaries for Shares, split up by
weekday, and shares split up by text sentiment polarity. Boxplots make
it even easier to look out for outliers (look for the dots separated
from the main boxplot). Next, we can examine several scatterplots.
Scatterplots allow us to look at one numerical variable vs another to
see if there is any correlation between them. Look out for any plots
that have most of the points on a diagonal line! There are four
scatterplots below, investigating shares vs Number of words in the
content, Number of words in the title, rate of positive words, and rate
of negative words. Finally, a histogram can show the overall
distribution of a numerical variable, including skewness. The histogram
below sows the distribution of the shares variable. Look for a left or
right tail to signify skewness, and look out for multiple peaks to
signify a multi-modal variable.

``` r
# Boxplot of Shares for Each Weekday, colored gray with classic theme, added labels and title
ggplot(filtered_channel, aes(x = weekday, y = shares)) + 
          geom_boxplot(fill = "grey") + 
          labs(x = "Weekday", title = "Boxplot of Shares for Each Weekday", y = "Shares") + 
          theme_classic()
```

![](./images/graphsJT-1.png)<!-- -->

``` r
# Scatterplot of Number of words in the content vs Shares, colored gray with classic theme, added labels and title
ggplot(filtered_channel, aes(x = n_tokens_content, y = shares)) + 
          geom_point(color = "grey") +
          labs(x = "Number of words in the content", y = "Shares", 
               title = "Scatterplot of Number of words in the content vs Shares") +
          theme_classic()
```

![](./images/graphsJT-2.png)<!-- -->

``` r
# Scatterplot of Number of words in the title vs Shares, colored gray with classic theme, added labels and title
ggplot(filtered_channel, aes(x = n_tokens_title, y = shares)) + 
          geom_point(color = "grey") +
          labs(x = "Number of words in the title", y = "Shares", 
               title = "Scatterplot of Number of words in the title vs Shares") +
          theme_classic()
```

![](./images/graphsJT-3.png)<!-- -->

``` r
ggplot(filtered_channel, aes(x=shares)) +
  geom_histogram(color="grey") +
  labs(x = "Number of images in an article", 
               title = "Histogram of number of shares") +
  theme_classic()
```

    ## `stat_bin()` using `bins = 30`. Pick better
    ## value with `binwidth`.

![](./images/graphsJW-1.png)<!-- -->

``` r
ggplot(filtered_channel, aes(x=rate_positive_words, y=shares)) +
  geom_point(color="grey") +
  labs(x = "Number of images in an article", y = "Shares", 
               title = "Scatterplot of rate of positive words in an article vs shares") +
  theme_classic()
```

![](./images/graphsJW-2.png)<!-- -->

``` r
ggplot(filtered_channel, aes(x=rate_negative_words, y=shares)) +
  geom_point(color="grey") +
  labs(x = "Number of images in an article", y = "Shares", 
               title = "Scatterplot of rate of negative words in an article vs shares") +
  theme_classic()
```

![](./images/graphsJW-3.png)<!-- -->

``` r
ggplot(filtered_channel, aes(x=global_sentiment_polarity, y=shares)) +
  geom_boxplot(color="grey") +
  labs(x = "Number of images in an article", y = "Shares", 
               title = "Scatterplot of global sentiment polarity in an article vs shares") +
  theme_classic()
```

    ## Warning: Continuous x aesthetic -- did you forget
    ## aes(group=...)?

![](./images/graphsJW-4.png)<!-- -->

``` r
# drop the weekday variable created for EDA (will get in the way for our models if we don't drop it)
filtered_channel <- subset(filtered_channel, select = -c(weekday))
```

# Modeling

## Splitting the Data

First, let’s split up the data into a testing set and a training set
using the proportions: 70% training and 30% testing.

``` r
set.seed(9876)
# Split the data into a training and test set (70/30 split)
# indices
train <- sample(1:nrow(filtered_channel), size = nrow(filtered_channel)*.70)
test <- setdiff(1:nrow(filtered_channel), train)

# training and testing subsets
Training <- filtered_channel[train, ]
Testing <- filtered_channel[test, ]
```

## Linear Models

Linear regression models allow us to look at relationships between one
response variable and several explanatory variables. A model can also
include interaction terms and even higher order terms. The general form
for a linear model is
![Y_i = \\beta_0 + \\beta_1 x_1 + \\beta_2 x_2 + ... + E_i](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;Y_i%20%3D%20%5Cbeta_0%20%2B%20%5Cbeta_1%20x_1%20%2B%20%5Cbeta_2%20x_2%20%2B%20...%20%2B%20E_i "Y_i = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + E_i"),
where each
![x_i](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;x_i "x_i")
represents a predictor variable and the “…” can include more predictors,
interactions and/or higher order terms. Since our goal is to predict
shares, we will be using these models to predict of a subset of the data
created for training, and then we will later test the models on the
other subsetted data set aside for testing.

Linear Model \#1: - Jordan

``` r
# linear model on training dataset with 5-fold cv
fit1 <- train(shares ~ . , data = Training, method = "lm",
              preProcess = c("center", "scale"), 
              trControl = trainControl(method = "cv", number = 5))
```

Linear Model \#2: - Jonathan

``` r
lm_fit <- train(
  shares ~ .^2,
  data=Training,
  method="lm",
  preProcess = c("center", "scale"), 
  trControl = trainControl(method = "cv", number = 5)
)
```

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

    ## Warning in predict.lm(modelFit, newdata):
    ## prediction from a rank-deficient fit may be
    ## misleading

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

    ## Warning in predict.lm(modelFit, newdata):
    ## prediction from a rank-deficient fit may be
    ## misleading

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

    ## Warning in predict.lm(modelFit, newdata):
    ## prediction from a rank-deficient fit may be
    ## misleading

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

    ## Warning in predict.lm(modelFit, newdata):
    ## prediction from a rank-deficient fit may be
    ## misleading

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

    ## Warning in predict.lm(modelFit, newdata):
    ## prediction from a rank-deficient fit may be
    ## misleading

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment,
    ## data_channel_is_bus, data_channel_is_socmed,
    ## data_channel_is_tech, data_channel_is_world,
    ## n_tokens_title:data_channel_is_entertainment,
    ## n_tokens_title:data_channel_is_bus,
    ## n_tokens_title:data_channel_is_socmed,
    ## n_tokens_title:data_channel_is_tech,
    ## n_tokens_title:data_channel_is_world,
    ## n_tokens_content:data_channel_is_entertainment,
    ## n_tokens_content:data_channel_is_bus,
    ## n_tokens_content:data_channel_is_socmed,
    ## n_tokens_content:data_channel_is_tech,
    ## n_tokens_content:data_channel_is_world,
    ## n_unique_tokens:data_channel_is_entertainment,
    ## n_unique_tokens:data_channel_is_bus,
    ## n_unique_tokens:data_channel_is_socmed,
    ## n_unique_tokens:data_channel_is_tech,
    ## n_unique_tokens:data_channel_is_world,
    ## n_non_stop_words:data_channel_is_entertainment,
    ## n_non_stop_words:data_channel_is_bus,
    ## n_non_stop_words:data_channel_is_socmed,
    ## n_non_stop_words:data_channel_is_tech,
    ## n_non_stop_words:data_channel_is_world,
    ## n_non_stop_unique_tokens:data_channel_is_entertainment,
    ## n_non_stop_unique_tokens:data_channel_is_bus,
    ## n_non_stop_unique_tokens:data_channel_is_socmed,
    ## n_non_stop_unique_tokens:data_channel_is_tech,
    ## n_non_stop_unique_tokens:data_channel_is_world,
    ## num_hrefs:data_channel_is_entertainment,
    ## num_hrefs:data_channel_is_bus,
    ## num_hrefs:data_channel_is_socmed,
    ## num_hrefs:data_channel_is_tech,
    ## num_hrefs:data_channel_is_world,
    ## num_self_hrefs:data_channel_is_entertainment,
    ## num_self_hrefs:data_channel_is_bus,
    ## num_self_hrefs:data_channel_is_socmed,
    ## num_self_hrefs:data_channel_is_tech,
    ## num_self_hrefs:data_channel_is_world,
    ## num_imgs:data_channel_is_entertainment,
    ## num_imgs:data_channel_is_bus,
    ## num_imgs:data_channel_is_socmed,
    ## num_imgs:data_channel_is_tech,
    ## num_imgs:data_channel_is_world,
    ## num_videos:data_channel_is_entertainment,
    ## num_videos:data_channel_is_bus,
    ## num_videos:data_channel_is_socmed,
    ## num_videos:data_channel_is_tech,
    ## num_videos:data_channel_is_world,
    ## average_token_length:data_channel_is_entertainment,
    ## average_token_length:data_channel_is_bus,
    ## average_token_length:data_channel_is_socmed,
    ## average_token_length:data_channel_is_tech,
    ## average_token_length:data_channel_is_world,
    ## num_keywords:data_channel_is_entertainment,
    ## num_keywords:data_channel_is_bus,
    ## num_keywords:data_channel_is_socmed,
    ## num_keywords:data_channel_is_tech,
    ## num_keywords:data_channel_is_world,
    ## data_channel_is_lifestyle:data_channel_is_entertainment,
    ## data_channel_is_lifestyle:data_channel_is_bus,
    ## data_channel_is_lifestyle:data_channel_is_socmed,
    ## data_channel_is_lifestyle:data_channel_is_tech,
    ## data_channel_is_lifestyle:data_channel_is_world,
    ## data_channel_is_entertainment:data_channel_is_bus,
    ## data_channel_is_entertainment:data_channel_is_socmed,
    ## data_channel_is_entertainment:data_channel_is_tech,
    ## data_channel_is_entertainment:data_channel_is_world,
    ## data_channel_is_entertainment:kw_min_min,
    ## data_channel_is_entertainment:kw_max_min,
    ## data_channel_is_entertainment:kw_avg_min,
    ## data_channel_is_entertainment:kw_min_max,
    ## data_channel_is_entertainment:kw_max_max,
    ## data_channel_is_entertainment:kw_avg_max,
    ## data_channel_is_entertainment:kw_min_avg,
    ## data_channel_is_entertainment:kw_max_avg,
    ## data_channel_is_entertainment:kw_avg_avg,
    ## data_channel_is_entertainment:self_reference_min_shares,
    ## data_channel_is_entertainment:self_reference_max_shares,
    ## data_channel_is_entertainment:self_reference_avg_sharess,
    ## data_channel_is_entertainment:weekday_is_monday,
    ## data_channel_is_entertainment:weekday_is_tuesday,
    ## data_channel_is_entertainment:weekday_is_wednesday,
    ## data_channel_is_entertainment:weekday_is_thursday,
    ## data_channel_is_entertainment:weekday_is_friday,
    ## data_channel_is_entertainment:weekday_is_saturday,
    ## data_channel_is_entertainment:weekday_is_sunday,
    ## data_channel_is_entertainment:is_weekend,
    ## data_channel_is_entertainment:LDA_00,
    ## data_channel_is_entertainment:LDA_01,
    ## data_channel_is_entertainment:LDA_02,
    ## data_channel_is_entertainment:LDA_03,
    ## data_channel_is_entertainment:LDA_04,
    ## data_channel_is_entertainment:global_subjectivity,
    ## data_channel_is_entertainment:global_sentiment_polarity,
    ## data_channel_is_entertainment:global_rate_positive_words,
    ## data_channel_is_entertainment:global_rate_negative_words,
    ## data_channel_is_entertainment:rate_positive_words,
    ## data_channel_is_entertainment:rate_negative_words,
    ## data_channel_is_entertainment:avg_positive_polarity,
    ## data_channel_is_entertainment:min_positive_polarity,
    ## data_channel_is_entertainment:max_positive_polarity,
    ## data_channel_is_entertainment:avg_negative_polarity,
    ## data_channel_is_entertainment:min_negative_polarity,
    ## data_channel_is_entertainment:max_negative_polarity,
    ## data_channel_is_entertainment:title_subjectivity,
    ## data_channel_is_entertainment:title_sentiment_polarity,
    ## data_channel_is_entertainment:abs_title_subjectivity,
    ## data_channel_is_entertainment:abs_title_sentiment_polarity,
    ## data_channel_is_bus:data_channel_is_socmed,
    ## data_channel_is_bus:data_channel_is_tech,
    ## data_channel_is_bus:data_channel_is_world,
    ## data_channel_is_bus:kw_min_min,
    ## data_channel_is_bus:kw_max_min,
    ## data_channel_is_bus:kw_avg_min,
    ## data_channel_is_bus:kw_min_max,
    ## data_channel_is_bus:kw_max_max,
    ## data_channel_is_bus:kw_avg_max,
    ## data_channel_is_bus:kw_min_avg,
    ## data_channel_is_bus:kw_max_avg,
    ## data_channel_is_bus:kw_avg_avg,
    ## data_channel_is_bus:self_reference_min_shares,
    ## data_channel_is_bus:self_reference_max_shares,
    ## data_channel_is_bus:self_reference_avg_sharess,
    ## data_channel_is_bus:weekday_is_monday,
    ## data_channel_is_bus:weekday_is_tuesday,
    ## data_channel_is_bus:weekday_is_wednesday,
    ## data_channel_is_bus:weekday_is_thursday,
    ## data_channel_is_bus:weekday_is_friday,
    ## data_channel_is_bus:weekday_is_saturday,
    ## data_channel_is_bus:weekday_is_sunday,
    ## data_channel_is_bus:is_weekend,
    ## data_channel_is_bus:LDA_00,
    ## data_channel_is_bus:LDA_01,
    ## data_channel_is_bus:LDA_02,
    ## data_channel_is_bus:LDA_03,
    ## data_channel_is_bus:LDA_04,
    ## data_channel_is_bus:global_subjectivity,
    ## data_channel_is_bus:global_sentiment_polarity,
    ## data_channel_is_bus:global_rate_positive_words,
    ## data_channel_is_bus:global_rate_negative_words,
    ## data_channel_is_bus:rate_positive_words,
    ## data_channel_is_bus:rate_negative_words,
    ## data_channel_is_bus:avg_positive_polarity,
    ## data_channel_is_bus:min_positive_polarity,
    ## data_channel_is_bus:max_positive_polarity,
    ## data_channel_is_bus:avg_negative_polarity,
    ## data_channel_is_bus:min_negative_polarity,
    ## data_channel_is_bus:max_negative_polarity,
    ## data_channel_is_bus:title_subjectivity,
    ## data_channel_is_bus:title_sentiment_polarity,
    ## data_channel_is_bus:abs_title_subjectivity,
    ## data_channel_is_bus:abs_title_sentiment_polarity,
    ## data_channel_is_socmed:data_channel_is_tech,
    ## data_channel_is_socmed:data_channel_is_world,
    ## data_channel_is_socmed:kw_min_min,
    ## data_channel_is_socmed:kw_max_min,
    ## data_channel_is_socmed:kw_avg_min,
    ## data_channel_is_socmed:kw_min_max,
    ## data_channel_is_socmed:kw_max_max,
    ## data_channel_is_socmed:kw_avg_max,
    ## data_channel_is_socmed:kw_min_avg,
    ## data_channel_is_socmed:kw_max_avg,
    ## data_channel_is_socmed:kw_avg_avg,
    ## data_channel_is_socmed:self_reference_min_shares,
    ## data_channel_is_socmed:self_reference_max_shares,
    ## data_channel_is_socmed:self_reference_avg_sharess,
    ## data_channel_is_socmed:weekday_is_monday,
    ## data_channel_is_socmed:weekday_is_tuesday,
    ## data_channel_is_socmed:weekday_is_wednesday,
    ## data_channel_is_socmed:weekday_is_thursday,
    ## data_channel_is_socmed:weekday_is_friday,
    ## data_channel_is_socmed:weekday_is_saturday,
    ## data_channel_is_socmed:weekday_is_sunday,
    ## data_channel_is_socmed:is_weekend,
    ## data_channel_is_socmed:LDA_00,
    ## data_channel_is_socmed:LDA_01,
    ## data_channel_is_socmed:LDA_02,
    ## data_channel_is_socmed:LDA_03,
    ## data_channel_is_socmed:LDA_04,
    ## data_channel_is_socmed:global_subjectivity,
    ## data_channel_is_socmed:global_sentiment_polarity,
    ## data_channel_is_socmed:global_rate_positive_words,
    ## data_channel_is_socmed:global_rate_negative_words,
    ## data_channel_is_socmed:rate_positive_words,
    ## data_channel_is_socmed:rate_negative_words,
    ## data_channel_is_socmed:avg_positive_polarity,
    ## data_channel_is_socmed:min_positive_polarity,
    ## data_channel_is_socmed:max_positive_polarity,
    ## data_channel_is_socmed:avg_negative_polarity,
    ## data_channel_is_socmed:min_negative_polarity,
    ## data_channel_is_socmed:max_negative_polarity,
    ## data_channel_is_socmed:title_subjectivity, dat

## Random Forest - Jordan

Random Forest is a tree based method for fitting predictive models, that
averages across all trees. One may choose to use a tree based methood
due to their prediction accuracy, the fact that predictors do not need
to be scaled, no statistical assumptions, and a built-in variable
selection process. Random forest, in particular, randomly selects a
subset of
![m = p / 3](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;m%20%3D%20p%20%2F%203 "m = p / 3")
predictors. This corrects the bagging issue where every bootstrap
contains a strong predictor for the first split.

``` r
# fandom forest model on training dataset with 5-fold cv
ranfor <- train(shares ~ ., data = Training, method = "rf", preProcess = c("center", "scale"),
                trControl = trainControl(method = "cv", number = 5), 
                tuneGrid = expand.grid(mtry = c(1:round(ncol(Training)/3))))
```

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

``` r
ranfor
```

    ## Random Forest 
    ## 
    ## 1469 samples
    ##   58 predictor
    ## 
    ## Pre-processing: centered (58), scaled (58) 
    ## Resampling: Cross-Validated (5 fold) 
    ## Summary of sample sizes: 1176, 1175, 1175, 1174, 1176 
    ## Resampling results across tuning parameters:
    ## 
    ##   mtry  RMSE      Rsquared     MAE     
    ##    1    8647.401  0.013806063  3339.815
    ##    2    8670.942  0.016487776  3416.647
    ##    3    8746.959  0.012640866  3453.693
    ##    4    8793.486  0.010810707  3490.870
    ##    5    8820.633  0.010464841  3515.555
    ##    6    8890.127  0.010299153  3534.856
    ##    7    8954.947  0.006461325  3577.440
    ##    8    8950.874  0.009611146  3564.044
    ##    9    9050.297  0.007999605  3592.140
    ##   10    9062.713  0.008020117  3603.272
    ##   11    9140.680  0.008393644  3629.264
    ##   12    9135.058  0.006962249  3631.516
    ##   13    9208.666  0.006922996  3654.880
    ##   14    9254.574  0.006711465  3659.443
    ##   15    9318.889  0.007400555  3685.630
    ##   16    9326.816  0.006721992  3683.519
    ##   17    9394.413  0.006586585  3708.637
    ##   18    9412.480  0.005168434  3712.670
    ##   19    9464.970  0.006418984  3694.991
    ##   20    9507.476  0.005502456  3729.256
    ## 
    ## RMSE was used to select the optimal model
    ##  using the smallest value.
    ## The final value used for the model was mtry = 1.

## Boosted Tree - Jonathan

``` r
tune_grid <- expand.grid(
  n.trees = c(5, 10, 50, 100),
  interaction.depth = c(1,2,3, 4),
  shrinkage = 0.1,
  n.minobsinnode = 10
)

bt_fit <- train(
  shares ~ .,
  data=Training,
  method="gbm",
  preProcess = c("center", "scale"), 
  trControl = trainControl(method = "cv", number = 5)
)
```

    ## Warning in preProcess.default(method = c("center",
    ## "scale"), x = structure(c(9, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 105186414.4306             nan     0.1000 -8190.9660
    ##      2 104635533.6093             nan     0.1000 -119296.8594
    ##      3 104211064.0922             nan     0.1000 -204272.3253
    ##      4 103885509.2736             nan     0.1000 -376674.2633
    ##      5 103703849.9430             nan     0.1000 -177405.1232
    ##      6 103626315.2741             nan     0.1000 -267489.9525
    ##      7 103007318.3224             nan     0.1000 -16195.9567
    ##      8 102922125.1071             nan     0.1000 -270800.8586
    ##      9 102573079.7617             nan     0.1000 -130064.6672
    ##     10 102157664.0377             nan     0.1000 -82946.5969
    ##     20 100203925.4856             nan     0.1000 -168592.0259
    ##     40 98750209.5697             nan     0.1000 -546520.8452
    ##     60 97314754.5000             nan     0.1000 65730.8357
    ##     80 95330791.3015             nan     0.1000 -1018941.3423
    ##    100 94586741.8314             nan     0.1000 -262842.8154
    ##    120 93410786.6903             nan     0.1000 1339.5170
    ##    140 91933538.7600             nan     0.1000 -460092.2919
    ##    150 91743279.2130             nan     0.1000 -236951.2229

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 105340130.3730             nan     0.1000 53235.3244
    ##      2 104485193.8217             nan     0.1000 -124474.1481
    ##      3 103627616.5294             nan     0.1000 -133949.3479
    ##      4 102793201.2806             nan     0.1000 -305504.6198
    ##      5 102254619.1289             nan     0.1000 -13277.7068
    ##      6 101848289.1047             nan     0.1000 -113815.1896
    ##      7 101476440.9914             nan     0.1000 -94793.8621
    ##      8 101087293.8282             nan     0.1000 -431026.8615
    ##      9 100945281.5334             nan     0.1000 -373295.1814
    ##     10 100805246.9067             nan     0.1000 -206026.9420
    ##     20 97163395.4192             nan     0.1000 -511558.1065
    ##     40 91233727.3673             nan     0.1000 -624421.5774
    ##     60 86568700.8920             nan     0.1000 -321063.1790
    ##     80 83918176.6155             nan     0.1000 -441631.8540
    ##    100 79928494.4332             nan     0.1000 -261104.3991
    ##    120 78334765.1544             nan     0.1000 -359513.4560
    ##    140 75298541.9623             nan     0.1000 -350654.8698
    ##    150 74826876.8244             nan     0.1000 -578777.3059

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 104398436.2349             nan     0.1000 64940.8695
    ##      2 103935092.4514             nan     0.1000 69670.5743
    ##      3 102758799.8749             nan     0.1000 -160706.4574
    ##      4 101819209.8987             nan     0.1000 -433361.2082
    ##      5 101378461.4125             nan     0.1000 -278735.8525
    ##      6 100967996.9766             nan     0.1000 -735710.1032
    ##      7 99855839.9426             nan     0.1000 162389.2990
    ##      8 99718987.1115             nan     0.1000 -236724.4173
    ##      9 99350210.5370             nan     0.1000 -80729.5711
    ##     10 98306067.1087             nan     0.1000 -177829.4290
    ##     20 94772646.6563             nan     0.1000 -258699.0329
    ##     40 87117817.9660             nan     0.1000 -927163.4356
    ##     60 82472883.0719             nan     0.1000 -403617.6339
    ##     80 77902477.8875             nan     0.1000 -113608.7509
    ##    100 74058089.0248             nan     0.1000 -634199.0223
    ##    120 69762657.2638             nan     0.1000 -194631.3905
    ##    140 67474473.9696             nan     0.1000 -326793.2642
    ##    150 65874247.1234             nan     0.1000 -597607.8498

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 37526602.6545             nan     0.1000 129510.9497
    ##      2 37141700.6158             nan     0.1000 -55000.7643
    ##      3 36762096.9653             nan     0.1000 232702.1676
    ##      4 36682977.2390             nan     0.1000 -15564.6875
    ##      5 36628952.4905             nan     0.1000 -59789.3244
    ##      6 36488492.7389             nan     0.1000 82479.5352
    ##      7 36405480.4282             nan     0.1000 -9374.4105
    ##      8 36297321.2241             nan     0.1000 -110818.9468
    ##      9 36068042.9604             nan     0.1000 -49012.5202
    ##     10 35901097.8328             nan     0.1000 -26579.4362
    ##     20 34474290.6870             nan     0.1000 26507.2437
    ##     40 33259590.4313             nan     0.1000 -72367.0977
    ##     60 32744607.5038             nan     0.1000 -234474.8986
    ##     80 32297530.8068             nan     0.1000 -131883.8967
    ##    100 32019974.2353             nan     0.1000 -62680.7419
    ##    120 31756752.9933             nan     0.1000 -330299.6088
    ##    140 31342245.8907             nan     0.1000 -27709.3862
    ##    150 31096602.5409             nan     0.1000 -156565.4855

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 37591690.5105             nan     0.1000 4937.7931
    ##      2 37000640.4531             nan     0.1000 107654.8940
    ##      3 36809192.5173             nan     0.1000 -10807.5387
    ##      4 36435471.2809             nan     0.1000 19475.0402
    ##      5 36014058.5174             nan     0.1000 -50993.1685
    ##      6 35768202.6400             nan     0.1000 -46077.8413
    ##      7 35647064.0638             nan     0.1000 27884.3614
    ##      8 35233597.2460             nan     0.1000 143442.1622
    ##      9 35129776.3275             nan     0.1000 -87491.7110
    ##     10 34908750.5686             nan     0.1000 -58430.3162
    ##     20 32685364.0931             nan     0.1000 -60007.7860
    ##     40 30491990.7273             nan     0.1000 -106001.2587
    ##     60 28597279.8956             nan     0.1000 -54883.6854
    ##     80 27113238.7793             nan     0.1000 -123882.1931
    ##    100 25896995.0269             nan     0.1000 -24706.0987
    ##    120 24755364.6921             nan     0.1000 -126580.6868
    ##    140 23775993.1728             nan     0.1000 -95484.2394
    ##    150 23286388.7966             nan     0.1000 -77277.8209

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 37726660.8890             nan     0.1000 67489.7939
    ##      2 37320252.5930             nan     0.1000 220609.0826
    ##      3 37039605.9140             nan     0.1000 96021.9597
    ##      4 36583682.7305             nan     0.1000 139178.6688
    ##      5 36050230.9353             nan     0.1000 26495.1424
    ##      6 35718437.6918             nan     0.1000 91919.5427
    ##      7 35264156.7924             nan     0.1000 30292.6385
    ##      8 34878528.4962             nan     0.1000 -104679.6459
    ##      9 34681886.7016             nan     0.1000 -100800.6812
    ##     10 34441069.7196             nan     0.1000 -90067.4245
    ##     20 32053514.6344             nan     0.1000 -318143.8860
    ##     40 28934754.6199             nan     0.1000 -14798.6497
    ##     60 26385451.2545             nan     0.1000 -138536.6838
    ##     80 24541617.3492             nan     0.1000 -110857.5489
    ##    100 22592683.2656             nan     0.1000 -15906.5180
    ##    120 21185166.1277             nan     0.1000 -85111.5781
    ##    140 20055715.2880             nan     0.1000 -61550.6375
    ##    150 19308188.5113             nan     0.1000 -74660.3993

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 103240858.0026             nan     0.1000 -117650.0557
    ##      2 102797529.6297             nan     0.1000 -123653.4763
    ##      3 102273184.7073             nan     0.1000 28304.7989
    ##      4 101884393.9775             nan     0.1000 -151617.2669
    ##      5 101484116.6421             nan     0.1000 -288262.7367
    ##      6 101123040.7221             nan     0.1000 -317618.7559
    ##      7 100913242.0067             nan     0.1000 -217546.2960
    ##      8 100704624.2057             nan     0.1000 -456018.1837
    ##      9 100570535.8687             nan     0.1000 -284678.8556
    ##     10 100296052.4762             nan     0.1000 -88951.6681
    ##     20 98967463.7010             nan     0.1000 82308.0163
    ##     40 97315576.3397             nan     0.1000 -251844.7926
    ##     60 95694618.5969             nan     0.1000 -417440.1165
    ##     80 94269700.8036             nan     0.1000 -268319.4022
    ##    100 92458733.0652             nan     0.1000 182726.5089
    ##    120 91768998.5979             nan     0.1000 -161460.4938
    ##    140 91479566.2591             nan     0.1000 -213484.2716
    ##    150 91154788.2911             nan     0.1000 -275478.6840

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 102625148.3850             nan     0.1000 -112771.4839
    ##      2 102355098.5102             nan     0.1000 22287.2783
    ##      3 102110323.5781             nan     0.1000 5711.5557
    ##      4 101434663.0968             nan     0.1000 -176436.7617
    ##      5 101183843.7010             nan     0.1000 -215866.4401
    ##      6 100866332.1673             nan     0.1000 -141757.1460
    ##      7 100437369.8202             nan     0.1000 -21937.2940
    ##      8 100025959.5246             nan     0.1000 -176499.2779
    ##      9 99886883.0557             nan     0.1000 -487558.0308
    ##     10 99542845.7536             nan     0.1000 -378322.0511
    ##     20 95690854.3869             nan     0.1000 -312070.3507
    ##     40 91321571.2186             nan     0.1000 -610163.9045
    ##     60 86777180.7471             nan     0.1000 -203674.5480
    ##     80 83936041.7602             nan     0.1000 -648383.5104
    ##    100 80359811.6927             nan     0.1000 -835943.3368
    ##    120 76419153.7672             nan     0.1000 -119485.3311
    ##    140 75162228.2127             nan     0.1000 -359772.9595
    ##    150 74746003.0007             nan     0.1000 -485793.0076

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 102793451.9367             nan     0.1000 78938.7450
    ##      2 102192851.6360             nan     0.1000 -148468.6996
    ##      3 101636703.0528             nan     0.1000 -109918.2258
    ##      4 101358631.1465             nan     0.1000 -36243.1188
    ##      5 101277252.8976             nan     0.1000 -292753.6288
    ##      6 100897646.6847             nan     0.1000 13908.1731
    ##      7 99989509.4205             nan     0.1000 -314163.6556
    ##      8 99562733.8986             nan     0.1000 -145645.6219
    ##      9 99056235.1016             nan     0.1000 -373919.9550
    ##     10 98886004.1156             nan     0.1000 -514753.9384
    ##     20 95711932.0447             nan     0.1000 -583616.7895
    ##     40 87821434.0349             nan     0.1000 -627250.9255
    ##     60 82851137.7143             nan     0.1000 -284341.1139
    ##     80 79662856.5130             nan     0.1000 -636813.6122
    ##    100 76719886.3483             nan     0.1000 -427873.7873
    ##    120 73962345.4101             nan     0.1000 -103505.2161
    ##    140 70146873.9778             nan     0.1000 -652305.4093
    ##    150 69946141.9570             nan     0.1000 -432151.1358

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 91420512.5632             nan     0.1000 -241208.2537
    ##      2 90751925.8867             nan     0.1000 -116446.2101
    ##      3 90223446.6799             nan     0.1000 -329220.7392
    ##      4 89843348.6852             nan     0.1000 -154597.9670
    ##      5 89498123.0737             nan     0.1000 -80694.1185
    ##      6 89266039.0733             nan     0.1000 -203120.2502
    ##      7 89082147.7846             nan     0.1000 -58676.4914
    ##      8 89018583.5444             nan     0.1000 -49123.7423
    ##      9 89081157.8191             nan     0.1000 -286110.6400
    ##     10 88839668.6213             nan     0.1000 -198768.4724
    ##     20 88062545.8886             nan     0.1000 -166686.3939
    ##     40 86718381.8828             nan     0.1000 70252.7944
    ##     60 85503399.9060             nan     0.1000 -170731.6672
    ##     80 84357160.8594             nan     0.1000 -883376.1379
    ##    100 83946816.9187             nan     0.1000 145005.9406
    ##    120 82877901.0159             nan     0.1000 -323838.2700
    ##    140 81289656.2806             nan     0.1000 67947.3998
    ##    150 81046212.6127             nan     0.1000 -503554.1253

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 91534908.4173             nan     0.1000 61898.0411
    ##      2 90380558.6930             nan     0.1000 -140831.4183
    ##      3 89713994.8886             nan     0.1000 -459345.7636
    ##      4 89445669.6985             nan     0.1000 -257028.4174
    ##      5 88877683.8734             nan     0.1000 -345388.9823
    ##      6 88846545.9660             nan     0.1000 -250701.7822
    ##      7 88611503.5608             nan     0.1000 -99572.6486
    ##      8 88446252.7577             nan     0.1000 -292838.0550
    ##      9 88188532.0679             nan     0.1000 -465947.1369
    ##     10 88022848.6211             nan     0.1000 -959591.7325
    ##     20 83710528.8218             nan     0.1000 -767374.5296
    ##     40 79182333.0314             nan     0.1000 -308791.7321
    ##     60 75398726.7082             nan     0.1000 -938190.6134
    ##     80 73427788.4314             nan     0.1000 -399489.9130
    ##    100 71122908.5624             nan     0.1000 -484197.0314
    ##    120 69499544.8591             nan     0.1000 -664435.4823
    ##    140 68064450.9582             nan     0.1000 -787503.5341
    ##    150 66779472.2170             nan     0.1000 -15934.7909

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 91542721.3978             nan     0.1000 -21830.7652
    ##      2 91280587.3949             nan     0.1000 -48317.5910
    ##      3 90113762.4176             nan     0.1000 24165.9356
    ##      4 90003286.4070             nan     0.1000 -140836.6577
    ##      5 89548221.5952             nan     0.1000 15694.7912
    ##      6 89410251.9973             nan     0.1000 -118015.2236
    ##      7 89198272.0898             nan     0.1000 9443.5508
    ##      8 89046354.1502             nan     0.1000 -102694.9502
    ##      9 88662221.1535             nan     0.1000 -369108.5652
    ##     10 88046633.1646             nan     0.1000 -272961.3033
    ##     20 85869450.5913             nan     0.1000 -674951.0393
    ##     40 82030392.1768             nan     0.1000 -697612.8247
    ##     60 76458364.9104             nan     0.1000 -333813.9016
    ##     80 74976708.1952             nan     0.1000 -450543.1368
    ##    100 71924408.2714             nan     0.1000 -545031.9695
    ##    120 68453500.6301             nan     0.1000 81161.5066
    ##    140 66078908.0080             nan     0.1000 -285824.3470
    ##    150 65650847.7329             nan     0.1000 -437953.0001

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 97139321.3094             nan     0.1000 59585.4944
    ##      2 96343398.3613             nan     0.1000 26938.0692
    ##      3 96309192.5150             nan     0.1000 -107297.4230
    ##      4 95818919.7410             nan     0.1000 -49143.1022
    ##      5 95310881.6604             nan     0.1000 -270037.0894
    ##      6 95020647.4227             nan     0.1000 -105147.1609
    ##      7 94684478.8091             nan     0.1000 -37307.3384
    ##      8 94424737.5997             nan     0.1000 -322820.6087
    ##      9 94125865.4391             nan     0.1000 -399053.9772
    ##     10 93996351.9128             nan     0.1000 -201993.7275
    ##     20 92692266.1708             nan     0.1000 -176545.0523
    ##     40 91113882.7204             nan     0.1000 -53908.5891
    ##     60 90024075.1715             nan     0.1000 -160823.0051
    ##     80 89276452.3542             nan     0.1000 -577134.8374
    ##    100 88008735.2074             nan     0.1000 -420272.6354
    ##    120 87731502.8640             nan     0.1000 -426125.6422
    ##    140 87001619.8474             nan     0.1000 -687299.2715
    ##    150 86665995.0813             nan     0.1000 -338042.2222

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 96224199.2356             nan     0.1000 -161838.8264
    ##      2 95839239.1247             nan     0.1000 49263.0827
    ##      3 94976947.9565             nan     0.1000 -641388.7861
    ##      4 94513505.3292             nan     0.1000 -227034.2729
    ##      5 94285646.2260             nan     0.1000 -407534.8413
    ##      6 94096796.1794             nan     0.1000 -283633.1813
    ##      7 94017349.1701             nan     0.1000 -183057.1587
    ##      8 92774082.0886             nan     0.1000 -118958.1206
    ##      9 92326762.9149             nan     0.1000 -124782.7999
    ##     10 92113163.2743             nan     0.1000 -394487.0660
    ##     20 90746831.8051             nan     0.1000 -241401.8365
    ##     40 86771603.9080             nan     0.1000 -840765.4307
    ##     60 82487443.9741             nan     0.1000 -357429.7348
    ##     80 79287271.8032             nan     0.1000 -635473.4076
    ##    100 76518170.0479             nan     0.1000 -285272.0545
    ##    120 74508151.2390             nan     0.1000 -306064.0705
    ##    140 72780049.2607             nan     0.1000 -701960.1420
    ##    150 72352217.4843             nan     0.1000 -434061.8822

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 96407795.6015             nan     0.1000 48959.6597
    ##      2 95730713.3043             nan     0.1000 9420.6832
    ##      3 95147422.3656             nan     0.1000 -50015.6431
    ##      4 94884163.9087             nan     0.1000 -151255.3474
    ##      5 94567378.5854             nan     0.1000 -57184.1169
    ##      6 93748100.3948             nan     0.1000 -163072.7114
    ##      7 93512344.2381             nan     0.1000 -411797.0019
    ##      8 93269800.4692             nan     0.1000 -453670.2090
    ##      9 93226242.9466             nan     0.1000 -523056.5378
    ##     10 92797072.7912             nan     0.1000 -553318.6307
    ##     20 88271152.9210             nan     0.1000 -347601.5563
    ##     40 80926698.8609             nan     0.1000 -314665.1800
    ##     60 77774270.4071             nan     0.1000 -235187.3562
    ##     80 72876747.1705             nan     0.1000 -555289.9201
    ##    100 69036988.3578             nan     0.1000 -382508.7588
    ##    120 67233528.2617             nan     0.1000 -259282.8003
    ##    140 63870896.7303             nan     0.1000 -645215.1265
    ##    150 61951067.9365             nan     0.1000 -84528.1207

    ## Warning in preProcess.default(thresh = 0.95, k =
    ## 5, freqCut = 19, uniqueCut = 10, : These variables
    ## have zero variances: data_channel_is_lifestyle,
    ## data_channel_is_entertainment, data_channel_is_bus,
    ## data_channel_is_socmed, data_channel_is_tech,
    ## data_channel_is_world

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 12:
    ## data_channel_is_lifestyle has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 13:
    ## data_channel_is_entertainment has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 14:
    ## data_channel_is_bus has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 15:
    ## data_channel_is_socmed has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 16:
    ## data_channel_is_tech has no variation.

    ## Warning in (function (x, y, offset = NULL, misc =
    ## NULL, distribution = "bernoulli", : variable 17:
    ## data_channel_is_world has no variation.

    ## Iter   TrainDeviance   ValidDeviance   StepSize   Improve
    ##      1 86474684.2752             nan     0.1000 -109769.4695
    ##      2 85979781.2285             nan     0.1000 275838.0140
    ##      3 85646087.7628             nan     0.1000 -113923.9261
    ##      4 85331100.7485             nan     0.1000 -81814.1525
    ##      5 85238159.8986             nan     0.1000 -250355.8792
    ##      6 84420137.0265             nan     0.1000 50659.6375
    ##      7 83806718.4974             nan     0.1000 186892.9733
    ##      8 83492337.8741             nan     0.1000 -110052.8013
    ##      9 83312197.1762             nan     0.1000 -409117.8120
    ##     10 82931869.7776             nan     0.1000 -257687.5926
    ##     20 81024248.4564             nan     0.1000 -418875.3219
    ##     40 78318756.6332             nan     0.1000 -343711.8743
    ##     50 75704653.5223             nan     0.1000 -574475.5153

``` r
bt_fit
```

    ## Stochastic Gradient Boosting 
    ## 
    ## 1469 samples
    ##   58 predictor
    ## 
    ## Pre-processing: centered (58), scaled (58) 
    ## Resampling: Cross-Validated (5 fold) 
    ## Summary of sample sizes: 1174, 1175, 1175, 1176, 1176 
    ## Resampling results across tuning parameters:
    ## 
    ##   interaction.depth  n.trees  RMSE    
    ##   1                   50      8349.487
    ##   1                  100      8416.981
    ##   1                  150      8445.566
    ##   2                   50      8364.222
    ##   2                  100      8524.384
    ##   2                  150      8598.466
    ##   3                   50      8325.348
    ##   3                  100      8479.079
    ##   3                  150      8569.094
    ##   Rsquared     MAE     
    ##   0.010888812  3385.248
    ##   0.010134066  3437.888
    ##   0.009662024  3408.131
    ##   0.016105060  3427.359
    ##   0.013940457  3531.638
    ##   0.014352657  3583.487
    ##   0.023478418  3390.699
    ##   0.017824212  3518.664
    ##   0.019975661  3597.147
    ## 
    ## Tuning parameter 'shrinkage' was held constant
    ##  parameter 'n.minobsinnode' was held constant at
    ##  a value of 10
    ## RMSE was used to select the optimal model
    ##  using the smallest value.
    ## The final values used for the model were n.trees
    ##  = 50, interaction.depth = 3, shrinkage = 0.1
    ##  and n.minobsinnode = 10.

# Comparison - Jordan

Finally, let’s compare our four models: 2 linear models, 1 random forest
model, and 1 boosted tree model.

``` r
# random forest prediction on testing model and its performance
predRF <- predict(ranfor, newdata = Testing)
RF <- postResample(predRF, Testing$shares)

# linear model 1 prediction on testing model and its performance
predlm1 <- predict(fit1, newdata = Testing)
LM <- postResample(predlm1, Testing$shares)

# linear model 2 prediction on testing model and its performance

# boosted tree prediction on testing model and its performance


# NEEDS TO BE REPEATED FOR OTHER TWO MODELS - I'll do this later!

# combine each of the performance stats for the models and add a column with the model names
dat <- data.frame(rbind(t(data.frame(LM)), t(data.frame(RF))))
df <- as_tibble(rownames_to_column(dat, "models"))

# find the model with the lowesr RMSE
best <- df %>% filter(RMSE == min(RMSE)) %>% select(models)

# print "The Best fitting model according to RMSE is [insert model name for lowest RMSE here]"
paste("The Best fitting model according to RMSE is", best$models, sep = " ")
```

    ## [1] "The Best fitting model according to RMSE is RF"

# Automation - Jonathan

``` r
#rmarkdown::render(
#  "Tanley-Wood-Project2.Rmd",
#  output_format="github_document",
#  output_dir="./Analysis",
#  output_options = list(
#    html_preview = FALSE
#  )
#)
```
