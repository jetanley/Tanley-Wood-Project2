# ST558 Project 2
By Jordan Tanley and Jonathan Wood

## Purpose

The purpose of this repo is to store the necessary files for the ST558 Project 2 and to create a space for collaboration between assigned partners, Jonathan and Jordan. The focus of this project is to investigate shares for news sites using the [Online News Popularity Data Set](https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity) from the UCI Machine Learning Repository.  Within the Tanley-Wood-Project2.Rmd file, you will find all of the code for creating the material found in the analysis links below. This includes a brief outline of the project, reading in the data, EDA using both numerical and graphical summaries, two linear models, two ensemble tree-based models (random forest and boosted tree), and automated comparisons of the four models. This is all automated to produce the six .html links found below, one for each data channel. 

## Required R Packages

The required packages include:

* [`tidyverse`](https://www.tidyverse.org/): This package loads in several packages, 3 of which we will use:  
  - [`readr`](https://readr.tidyverse.org/): Used to read in the data  
  - [`ggplot2`](https://ggplot2.tidyverse.org/): Used for plotting and making figures
  - [`dplyr`](https://dplyr.tidyverse.org/): Coding grammar
* [`knitr`](https://yihui.org/knitr/): This allows us to use `kable` for producing nice tables  
* [`caret`](https://topepo.github.io/caret/): Used for building our models  



## Analysis Links

Here are the links to each of the rendered documents:

* [Lifestyle](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-lifestyle)  
* [Entertainment](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-entertainment)  
* [Business](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-bus)  
* [Social Media](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-socmed)
* [Tech](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-tech)
* [World](https://jetanley.github.io/Tanley-Wood-Project2/Analysis/Tanley-Wood-Project2-world)

## Render Code

```r
data_channels = c("lifestyle", "entertainment", "bus", "socmed", "tech", "world")

for (channel in data_channels) {
  rmarkdown::render(
    "Tanley-Wood-Project2.Rmd",
    output_format = "github_document",
    output_dir = "./Analysis",
    output_options = list(
      html_preview = FALSE
    ),
    params = list(
      channel = channel
    ),
    output_file = paste0("Tanley-Wood-Project2-", channel, ".md")
  )
}
```