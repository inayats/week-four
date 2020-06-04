# Using R for data analysis/charts

We used RCurl to import data from the web

* Making a table

`x <- getURL("URL OF CSV", .opts = list(ssl.verifypeer = FALSE))`

This creates a variable 'x' in my Environment pane. From there we create the table with columns.
`documents <- read.csv(text = x, col.names=c("Column1", "Column2 etc"), colClasses=rep("character", 3), sep=",", quote="")

* Analyzing variables

We have to create a variable before we can count it
```
counts <- table(documents$Newspaper.City)
counts
```

* Counting other things

I wanted to plot the frequency of certain newspapers.

Making a new variable

`papertitle <- table(documents$Newspaper.Title)`

Plotting the new variable

`barplot(papertitle, main="Newspapers", xlab="Number of Articles")`

The Glasgow Advertiser and Scotsman turn up as the main newspaper titles.

# Voyant Tools for text analysis

* Web based tool available at (link)[https://voyant-tools.org/]
* Add or upload an excel file for analysis
* In options, change Documents to "from cells in each row"
* Add the correct columns for Content, Author and Title (and you can combine columns with a +)
* For some reason pasting the link didn't work, so I downloaded the sample excel file and uploaded it
Sample corpus ID = 9cc66a9fd10eb25e5622157c51b91b02

* Using ther Terms tab, I can select certain words. Double clicking opens up the term in the Trends tab.

* I played around with the tab to see the mentions of the word "Indian" in the data. Its main collocate was "killed" showing how Indigenous people were closely associated with violence in the text.

* Exporting the view - hit the embed button, choose "an HTML snippet for embedding..." to get the iFrame code.

<iframe style='width: 474px; height: 337px;' src='https://voyant-tools.org/tool/CorpusTerms/?corpus=9cc66a9fd10eb25e5622157c51b91b02'></iframe>

# Building my own corpus

The exercise had us downloading articles from the Chronicling America database. I used the suggestion to search for articles with "Canada" in them.

* Use the Python script from week2 for accessing the Chronicling America API (link)[https://craftingdh.netlify.app/week/2/apis/]

* Use the keyword "Canada"

`'proxtext': 'archeology' # Search for this keyword  `

# AntConc

* Followed the (tutorial)[https://programminghistorian.org/en/lessons/corpus-analysis-with-antconc]   
* Downloaded and installed AntConc
* Downloaded the sample file for the exercise

*Use Open Dir to open the whole folder in AntConc


# Topic Modeling

* Downloaded and installed the Topic Modeling Tool from the link shared in the discord chat [link](https://github.com/shawngraham/topic-modeling-tool-1/blob/master/TopicModelingTool.zip)

* Made a new folder called tmt, with input and output folders inside.
* Added the chapbooks file to the input folder

* Open Topic Modeling and set the input and output folders. I chose 10 for the number of topics.

* Took 23 minutes to process

* The output csv files contain several files. Using the guide from [Andy Wallace](http://miriamposner.com/blog/very-basic-strategies-for-interpreting-results-from-the-topic-modeling-tool/) I started with the TopicsinDocs file, and analysed one row. The cells work like this - the number of the topic, followed by the percentage contribution. The topics were number 0 to 9.

* In the output HTML files I opened all_topics

* The topics are just a series of words at this point. Opening one topic shows the topic txt files associated with it.

* The tool does seem to require a fair amount of manual work to categorize the topics. By looking at the txt files, I had these guesses as to the topics (could be wildly off):

Religious texts
Obituaries
Love letters ??
Accounts of battle ????
Book summaries
Religious leader/priests

* Perhaps lowering the number of topics for the tool to model would help here.

# Topic modeling in R

* Downloaded the csv of the chapbooks and added it to a new folder chapbooks-r
* Opened Rstudio 
* Opened a new project in the directory

* Installed the required packages
```
install.packages('tidyverse')
install.packages('tidytext')
```

* Open a new R script (not in the Console)

Pasted the code - for cleaning the digits from the text field of the data and loading up the data.

```
# slightly modified version of
# https://tm4ss.github.io/docs/Tutorial_6_Topic_Models.html
# by Andreas Niekler, Gregor Wiedemann

# libraries
library(tidyverse)
library(tidytext)

# load, clean, and get data into shape

# cb = chapbooks
cb  <- read_csv("chapbooks-text.csv")

# put the data into a tibble (data structure for tidytext)
# we are also telling R what kind of data is in the 'text',
# 'line', and 'data' columns in our original csv.
# we are also stripping out all the digits from the text column

cb_df <- tibble(id = cb$line, text = (str_remove_all(cb$text, "[0-9]")), date = cb$date)

#turn cb_df into tidy format
# use `View(cb_df)` to see the difference
# from the previous table

tidy_cb <- cb_df %>%
  unnest_tokens(word, text)

# the only time filtering happens
# load up the default list of stop_words that comes
# with the tidyverse

data(stop_words)

# delete stopwords from our data
tidy_cb <- tidy_cb %>%
  anti_join(stop_words)
In the console, take a look at how your data has been transformed. Type cb and hit enter - you’ll see a table very much like how your data would look if you opened it in excel. Type cb_df and it’s still table like, but columns and digits are removed. Type tidy_cb and you’ll see it’s a list of words, with metadata indicating which document the word is a part of, and which year the document was written! Transforming your data like this makes it more amenable to text analysis. Let’s continue.

We can now transform that list into a matrix, which will enable us to do the topic model. Add, and then run, the following lines to your script:

# this line might take a few moments to run btw
cb_words <- tidy_cb %>%
  count(id, word, sort = TRUE)

# take a look at what you've just done
# by examining the first few lines of `cb_words`

head(cb_words)

# already, you start to get a sense of what's in this dataset...

# turn that into a matrix
dtm <- cb_words %>%
  cast_dtm(id, word, n)

  ```

 * REMEMBER to run each line separately!

 * Watch the Console tab to see if things are running

 * Now the topic model code: (for 15 topics)

 ```
 require(topicmodels)
# number of topics
K <- 15
# set random number generator seed
# for purposes of reproducibility
set.seed(9161)
# compute the LDA model, inference via 1000 iterations of Gibbs sampling
topicModel <- LDA(dtm, K, method="Gibbs", control=list(iter = 500, verbose = 25))
```

* The code gave an error for the first line - require(topicmodels). You need to first install it:

`install.packages('topicmodels')`

* Got an error that dtm was not found. Went back in the code to find where dtm is defined and ran it. Very important to run every line of code.









