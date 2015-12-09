---
title    : Linguistic Analysis and Data Science
subtitle : lecture 09
author   : 謝舒凱 Graduate Institute of Linguistics, NTU
mode     : selfcontained # {standalone, draft}
url      : {lib: "../../libraries", assets: "../../assets"}
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
lecnum   : "03"
widgets     : [bootstrap, quiz, interactive, mathjax]  # {mathjax, shiny, bootstrap}
ext_widgets: {rCharts: [libraries/widgets/nvd3]}
knit     : slidify::knit2slides
bibliography: /Users/shukai/LOPE_LexInfo/BIB/corpus.bib
github      : {user: loperntu, repo: lads}


--- bg:#FFFAF0
## 大綱

1. More on text classification
2. Topic modeling
3. Web data collection 




---
## Discussion on your approach in Kaggle



---bg:#FFFAF0
## Recap: 機器學習程式實作基本流程

- [`create_matrix`] Import your hand-coded data into R
- [`create_corpus`] 把「不相關」的資料移除，建立訓練語料 (training dataset) 與測試語料 (test data) (with `dtm`)
- [`train model(s)`] Choose machine learning algorithm(s) to train a model
- [`build predictive model(s)`] Test on the (out-of-sample) test data; establish accuracy criteria 了解成效。
- [`apply predictive model(s)`] Use model to classify novel data
- [`create analytics`] 把自動分錯的資料找出來 Manually label data that do not meet accuracy criteria



---
## 再做一個例子 (Mayor, 2015)


```r
URL = "http://www.cs.cornell.edu/people/pabo/movie-review-data/review_polarity.tar.gz"
download.file(URL,destfile = "./data/reviews.tar.gz")
#untar("./data/reviews.tar.gz")
setwd("./data/reviews/txt_sentoken")

library(tm)
SourcePos <- DirSource(file.path(".", "pos"), pattern="cv")
SourceNeg <- DirSource(file.path(".", "neg"), pattern="cv")
pos <- Corpus(SourcePos)
neg <- Corpus(SourceNeg)
reviews <- c(pos, neg)
```

---
## Preprocessing data


```r
preprocess = function(
  corpus, stopwrds = stopwords("english")){
  library(SnowballC)
  corpus <- tm_map(corpus, content_transformer(tolower))
  corpus <- tm_map(corpus, removePunctuation)
  corpus <- tm_map(corpus, content_transformer(removeNumbers))
  corpus <- tm_map(corpus, removeWords, stopwrds)
  corpus <- tm_map(corpus, stripWhitespace)
  corpus <- tm_map(corpus, stemDocument)
  corpus
}
```

---
## Exploring data


```r
processed <- preprocess(reviews)
term_documentFreq <- TermDocumentMatrix(processed)

asMatrix <- t(as.matrix(term_documentFreq))
Frequencies <- colSums(asMatrix)
head(Frequencies[order(Frequencies, decreasing=T)], 5)

Frequencies[Frequencies > 3000]

#Present <- data.frame(asMatrix)
#Present[Present>0] = 1

DocFrequencies <- colSums(Present)
head(DocFrequencies[order(DocFrequencies, decreasing=T)], 5)
DocFrequencies[DocFrequencies > 1400]



# total <- ncol(asMatrix)
# moreThanOnce <- sum(DocFrequencies != Frequencies)
# prop <- moreThanOnce / total
# moreThanOnce
# total
# prop

term_documentTfIdf <- TermDocumentMatrix(processed,
                                         control = list(weighting = function(x) weightTfIdf(x, normalize = TRUE)))


SparseRemoved <- as.matrix(t(removeSparseTerms(
  term_documentTfIdf, sparse = 0.8)))

ncol(SparseRemoved)
sum(rowSums(as.matrix(SparseRemoved)) == 0)

colnames(SparseRemoved)
```

---
## Computing new attributes


```r
quality <- c(rep(1,1000),rep(0,1000))
lengths <- colSums(as.matrix(TermDocumentMatrix(processed)))
```


---
## Creating the training and testing data frames


```r
DF <- as.data.frame(cbind(quality, lengths, SparseRemoved))
set.seed(123)
train = sample(1:2000,1000)
TrainDF = DF[train,]
TestDF = DF[-train,]
```


---
## Text Classification of the reviews
- **Naïve Bayes** 


```r
library(e1071)
library(caret) # confusionMatrix is in the caret package
set.seed(345)
model <- naiveBayes(TrainDF[-1], as.factor(TrainDF[[1]]))
classifNB <- predict(model, TrainDF[,-1])
confusionMatrix(as.factor(TrainDF$quality),classifNB)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   0   1
##          0 353 139
##          1  74 434
##                                          
##                Accuracy : 0.787          
##                  95% CI : (0.7603, 0.812)
##     No Information Rate : 0.573          
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.573          
##  Mcnemar's Test P-Value : 1.159e-05      
##                                          
##             Sensitivity : 0.8267         
##             Specificity : 0.7574         
##          Pos Pred Value : 0.7175         
##          Neg Pred Value : 0.8543         
##              Prevalence : 0.4270         
##          Detection Rate : 0.3530         
##    Detection Prevalence : 0.4920         
##       Balanced Accuracy : 0.7921         
##                                          
##        'Positive' Class : 0              
## 
```


---
## Classification of the reviews
- **Support Vector Machines**


```r
library(e1071)
modelSVM <- svm(quality ~ ., data = TrainDF)
probSVMtrain <- predict(modelSVM, TrainDF[,-1])
classifSVMtrain <- probSVMtrain
classifSVMtrain[classifSVMtrain>0.5] = 1
classifSVMtrain[classifSVMtrain<=0.5] = 0
confusionMatrix(TrainDF$quality, classifSVMtrain)
```


```r
probSVMtest <- predict(modelSVM, TestDF[,-1])
classifSVMtest <- probSVMtest
classifSVMtest[classifSVMtest>0.5] = 1
classifSVMtest[classifSVMtest<=0.5] = 0
confusionMatrix(TestDF$quality, classifSVMtest)
```


---
## Classification of the reviews
- **Deep Learning** 

[Practical Deep Text Learning tutorial](https://dato.com/learn/gallery/notebooks/deep_text_learning.html?_ga=1.58953197.192178516.1447602577)



---
## More on extraction of *meaning*

- Topic modeling [tutorial](https://eight2late.wordpress.com/2015/09/29/a-gentle-introduction-to-topic-modeling-using-r/)
- Event detection



---
## Microposts: 當前文本挖掘熱門議題
Check [this CFP](http://microposts2016.seas.upenn.edu/topics.html)

<img style = 'border: 1px solid;' width = 60%; src='./assets/img/message.png'></img>



---
## 如何抓資料

- crawler 好像很厲害，但是遊走法律邊緣 (**trespass to chattels**)
- web scaping 還可以。
- 利用 `api` 則是課堂上應該教的 XD



---
## Easy web scraping with `rvest`

>  a new package that makes it easy to scrape (or harvest) data from html web pages, inspired by libraries like beautiful soup. It is designed to work with magrittr so that you can express complex operations as elegant pipelines composed of simple, easily understood pieces.

- 需要有網頁設計概念: `rvest` + CSS Selector 很厲害。[`Selectorgadget`](http://selectorgadget.com))

---
## 舉例
- scrape some information about **Ex Machina** from IMDB.


```r
library(rvest)
machina <- html("http://www.imdb.com/title/tt0470752/")

# extract the rating 
machina %>% 
  html_node("strong span") %>%
  html_text() %>%
  as.numeric()

# extract the cast
machina %>%
  html_nodes("#titleCast .itemprop span") %>%
  html_text()

# The titles and authors of recent message board postings are stored in a the third table on the page. We can use html_node() and [[ to find it, then coerce it to a data frame with html_table():

machina %>%
  html_nodes("table") %>%
  .[[3]] %>%
  html_table()
```

---
## Collecting NYT news articles via API

[`rtimes`](https://github.com/ropengov/rtimes): R client for NYTimes API for government data, including the Congress, Article Search, Campaign Finance, and Geographic APIs. The focus is on those that deal with political data, but throwing in Article Search and Geographic for good measure（另外也可用 `tm` 自家的 plugin `tm.plugin.webmining`）。


- Get your own API keys at <http://developer.nytimes.com/apps/register>, you'll need a different key for each API.
- Put these entries in your `.Rprofile` file for re-use.

```
options(nytimes_cg_key = "e63b6f8917f30c79521ad7ddba7b9255:11:66687269") 
options(nytimes_as_key = "017ecf6cafb56e24947086cc1778ea30:1:66687269")
options(nytimes_cf_key = "YOURKEYHERE")
options(nytimes_geo_key = "YOURKEYHERE")
```



---
## 使用很容易


```r
library(rtimes)

# Search for bailout between two dates, Oct 1 2008 and Dec 1 2008
out <- as_search(q = "bailout", begin_date = "20081001", end_date = '20081201')
out$data[1:2]

# Search for keyword money, within the Sports and Foreign news desks
res <- as_search(q = "money", fq = 'news_desk:("Sports" "Foreign")')
res$data[1:3]
```

---




---
## Lab: Collecting data via API

各組選一個實作看看 [同一共筆]

-`rtimes` 

-`Rfacebook`

-`Rweibo`

-`twitteR`


