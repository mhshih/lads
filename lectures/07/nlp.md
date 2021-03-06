
---
title: "R 自然語言處理"
author: "Shu-Kai Hsieh"
date: "2015 年 11 月 4 日"
output: html_document
---

## 安裝前置作業

- 電腦中裝好了 `Java Developer's Kit (JDK)`


```r
install.packages(c("rJava", "NLP", "openNLP", "RWeka", "qdap"))
```
- 要處理什麼語言通常要下載該語言的 language models


```r
install.packages("openNLPmodels.en",
                 repos = "http://datacube.wu.ac.at/",
                 type = "source")
```

## 前處理




```r
dickens <- readLines("../../../data/week8/dickens-clean.txt")
#print(dickens)
```

We can combine all of these character vectors into a single character vector using the paste() function, adding a space between each of them.


```r
dickens.1 <- paste(dickens, collapse = " ")
```


## 詞與句子的自動標示



```r
library(NLP)
library(openNLP)
library(RWeka)
```

- `NLP` 套件要求所處理的文本先轉成 `String` 類別。


```r
dickens.1 <- as.String(dickens.1)
```

- 之後就可以建立套件中的各種 **tagger** (許多套件的函式稱之為 **annotator**)，並且用管線概念串結起來。


```r
word_ann <- Maxent_Word_Token_Annotator()
sent_ann <- Maxent_Sent_Token_Annotator()
dickens_tagged <- annotate(dickens.1, list(sent_ann, word_ann))
```

```
## Error in as.data.frame.default(x[[i]], optional = TRUE): cannot coerce class "c("Simple_Sent_Token_Annotator", "Annotator")" to a data.frame
```

```r
#要等一下子，結果是 annotation object
#class(dickens_tagged) 
head(dickens_tagged)
```

```
##  id type     start end  features
##   1 sentence     1  522 constituents=<<integer,110>>
##   2 sentence   525 1038 constituents=<<integer,100>>
##   3 sentence  1041 1343 constituents=<<integer,58>>
##   4 sentence  1346 1726 constituents=<<integer,68>>
##   5 sentence  1729 1951 constituents=<<integer,42>>
##   6 sentence  1954 2180 constituents=<<integer,46>>
```

- `NLP` 的 `AnnotatedPlainTextDocument()` 施用後，就可用 `sents()`, `words()` 來抽取句子和字。


```r
dickens_doc <- AnnotatedPlainTextDocument(dickens.1, dickens_tagged)
#sents(dickens_doc)
head(words(dickens_doc),10)
```

```
##  [1] "CHAPT"     "R I  "     "REATS "    "F THE PLA" "E "       
##  [6] "H"         "RE OLIV"   "R TW"      "I"         "T WAS"
```

## 自動詞類標記


```r
# tagPOS()
```

## 專用名詞自動標示

- 在自然語言處理中叫做**命名實體辨識** (Named Entity Recognition (NER))
- `OpenNLP` 套件可以提供辨識 `date`, `location`, `money`, `organization`, `percentage`, `person`, and `time` (`misc`). 
- 建立 tagger 的方式類似


```r
person_ann <- Maxent_Entity_Annotator(kind = "person")
location_ann <- Maxent_Entity_Annotator(kind = "location")
organization_ann <- Maxent_Entity_Annotator(kind = "organization")
```

- 重新建立一個管線作業


```r
dickens.pipe <- list(sent_ann,
                     word_ann,
                     person_ann,
                     location_ann,
                     organization_ann)
dickens_tagged.2 <- annotate(dickens.1, dickens.pipe)
```

```
## Error in as.data.frame.default(x[[i]], optional = TRUE): cannot coerce class "c("Simple_Sent_Token_Annotator", "Annotator")" to a data.frame
```

```r
dickens_doc.2 <- AnnotatedPlainTextDocument(dickens.1, dickens_tagged.2)
```

在抽取部分必須自己寫函式


```r
# Extract entities from an AnnotatedPlainTextDocument
entities <- function(doc, kind) {
  s <- doc$content
  a <- annotations(doc)[[1]]
  if(hasArg(kind)) {
    k <- sapply(a$features, `[[`, "kind")
    s[a[k == kind]]
  } else {
    s[a[a$type == "entity"]]
  }
}
```

這樣我們可以用 `entities()` 來抽取專名，類型則由 `kind=` 來指定。

```r
entities(dickens_doc.2, kind = "person")
```

```
## [1] "sibly befall" "d and "       "ed rat"       "ract; "
```


## 句法信息自動剖析與標示 (Parser)

## Chunker


# 萬一要處理的是語料庫













