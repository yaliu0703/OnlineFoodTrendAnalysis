# 

**Project description:** 

Early detection of emerging food trends can translate into great business opportunities. Today, a lot of food-related discussions occur on social media platforms such as Twitter and Facebook. Thus, such social media content presents a potentially valuable and real-time source of intelligence that can be leveraged by retailers to better serve its customers. The purpose of this project is to explore this possibility using techniques discussed in the social media analytics course to help retailers see the rise and fall of certain categories of food before competitors do. 

The project is based on 4 million Facebook posts from 2011 to 2015. Here I will validate my method with the case of Cauliflower Rice whose demand has been growing steadily from 2011 to 2015.

## Potential approaches

1. Topic modelling. Analyze topic relevant with food to learn about recent trends. However, this may take a long time and explored topic is ambiguous.

2. Syntactic analysis. Firstly, use document term matrix to learn about changes of different ingredients. Then we may choose interested ingredients to develop insights by conducting co-occurrence analysis of chosen ingredients. This method is more direct and easier to carry out. I will adopt this approach in the following analysis.

## 1. Explore changes of different terms from 2011 to 2015 with document term matrix

As each document represents a month, I conduct document term matrix to express the number of terms used in every month from 2011 to 2015:

```javascript
#construct dtm
docs<-Corpus(DirSource( c("data/fb2011","data/fb2012","data/fb2013","data/fb2014","data/fb2015") ))
mystopwords <- c("http","www","com","https","can","will","html","just","know","now","one","day","time","year","got","made","don","today","will","really","ever","may","make","use","also","get","week","minutes")
dtm <- DocumentTermMatrix(docs, control=list(tolower=T, removePunctuation=T, removeNumbers=T, stripWhitespace=T, stopwords=c(mystopwords,stopwords("english"), stopwords("spanish"))))
dtm <- removeSparseTerms(dtm,0.97)
dtm <- as.matrix(dtm)
```

As our manager is interested in emerging trends of food consumption, I keep terms relevant with ingredients:

```javascript
library(text2vec)
ingredients <- tolower(readLines("ingredients.txt", warn = FALSE))
freq_matrix <- dtm[,which(colnames(dtm) %in% ingredients)]
```
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/dtm1.png?raw=true"/>

Now we can calculate the increase/decrease rate of frequency each term used every year to see general trend. We may more likely to interest those terms with great increase in times and frequency. After filtering out, we are interested in Cauliflower. We may use the number of times term Cauliflower used and the frequency of term Cauliflower used to detect the trend of Cauliflower:

```javascript
##validation use cauliflower

freq_cauliflower <- freq_matrix[,"cauliflower"]/rowSums(freq_matrix)
barplot(freq_cauliflower,xlab = "Time", main = "the freqency of Term Cauliflower used")
barplot(freq_matrix[,"cauliflower"], xlab = "Time", main = "the number of Term Cauliflower used")
```
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/the%20frequency%20of%20cauliflower%20used.png?raw=true"/>

<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/the%20number%20of%20Term%20Cauliflower%20used.png?raw=true"/>

From the graphs above, despite the seasonal fluctuation, we can see that there is an increase in both frequency and times of term Cauliflower being used. Is there any emerging trend of cauliflower consumption that relevant with this change? I will explore this by conducting cooccurrence analysis.

## 2. Explore changes of different terms from 2011 to 2015 with document term matrix

In reality, to detect the emerging trend as early as possible, I suggest that we use weekly or monthly data. Considering the limit of my computer memory, however, here I can only construct co-occurrence matrix by year.

### 2.1 Load data

```javascript
files <- c()
filename_year <- "data/fb2011/fpost-2011-"
for (j in 1:12){
  filename <- paste(filename_year,j,".csv", sep='')
  files <- c(files, filename)
}
# readLines for each file and put them in a list
lineList <- lapply(files, readLines)
# create a character vector that contains all lines from all files
lineVector <- unlist(lineList)
# collapse the character vector into a single string
mytxt <- paste(lineVector , collapse = '\n')
```

### 2.2 Construct TCM

```javascript
# construct TCM
it = itoken(mytxt, preprocessor=tolower, tokenizer=word_tokenizer)
stop_words = c("and", "the", "with", "for", "is", "week","like","love","make","recipes","cook","good","making","cup","http","on","a","of","this","one","2011","sweet","easy","just","add","1","2","3","4","5","6","7","8","9","0","01","02","03","04","05","06","07","08","09","one","just","made","red","will","new","day","best")  
vocab <- create_vocabulary(it,stopwords = c(stop_words,stopwords("english")))
vocab <- prune_vocabulary(vocab, term_count_min = 10L)
vectorizer <- vocab_vectorizer(vocab)
tcm <- create_tcm(it, vectorizer, skip_grams_window = 10L) 
tcm <- as.matrix(tcm) # convert from the sparse matrix format to a regular one
```
### 2.3 Check co-occurrence matrix and visualize the result

```javascript
# syntagmatic association (first-order co-occurrence)
target <- "cauliflower" 
result_2011 <- sort(tcm[target,], decreasing=TRUE)[1:50]

#final result
library(wordcloud)
wordcloud(names(result_2011), result_2011, max.words=50,colors=brewer.pal(8, "Dark2"))
```
#### 2.3.1 visualization of 2011
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/result+wordcloud_2011.jpg?raw=true"/>

#### 2.3.2 visualization of 2012
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/result+wordcloud_2012.jpg?raw=true"/>

#### 2.3.3 visualization of 2013
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/result+wordcloud_2013.jpg?raw=true"/>

#### 2.3.4 visualization of 2014
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/result+wordcloud_2014.jpg?raw=true"/>

#### 2.3.3 visualization of 2015
<img src="https://github.com/yaliu0703/OnlineFoodTrendAnalysis/blob/master/img/result+wordcloud_2015.jpg?raw=true"/>

From results above, we can easily notice that:

a. The number of times Cauliflower and the Rice co-occur within the window is increasing as time goes by.

b. Compared with other words, the likelihood of rice co-occurring with Cauliflower is increasing as time goes by.

The analysis implies that from 2011 to 2015 the times of consumers mentioning rice when they mention Cauliflower has increased a lot and the likelihood of consumers mentioning rice when they mention Cauliflower has also increased. It is possible that people are more interested in cauliflower rice. From words like, love and delicious which appear in cooccurrence matrix, we can tell that people may express positive feelings for cauliflower. We may infer that people like consuming cauliflower more in recent years.

Now let’s adjust gram window to 1 and only calculate context right to the cauliflower:















