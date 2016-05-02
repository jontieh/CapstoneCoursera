# Capstone Project - Milestone Report
Jonathan Tieh  

# Introduction

This milestone report will be applying data science in the area of natural language processing. The following lines addressing the data extraction, cleaning and text mining of the so called [HC Copora](http://www.corpora.heliohost.org). This report is part of the data science capstone project of [Coursera](https://www.coursera.org) and [Swiftkey](http://swiftkey.com/). The plots, code chunks and remarks will explain the reader the first steps to build a prediction application.



# Data Processing

The data set consists of three files in US English.

### Loading The Dataset 

```r
fileURL <- "http://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip"
download.file(fileURL, destfile = "Dataset.zip", method = "curl")
unlink(fileURL)
unzip("Dataset.zip")
```



### Aggreagating A Data Sample

In order to enable faster data processing, a data sample from all three sources was generated.


```r
sampleTwitter <- twitter[sample(1:length(twitter),10000)]
sampleNews <- news[sample(1:length(news),10000)]
sampleBlogs <- blogs[sample(1:length(blogs),10000)]
textSample <- c(sampleTwitter,sampleNews,sampleBlogs)
```





# Summary Statistics









The following table provides an overview of the imported data. In addition to the size of each data set, the number of lines and words are displayed. 


File Name            File Size in Megabyte   Line Count   Word Count
------------------  ----------------------  -----------  -----------
Blogs                               200.42       899288     37334147
News                                196.28      1010242     34372530
Twitter                             159.36      2360148     30373603
Aggregated Sample                     2.42        15000        15000



A word cloud usually provides a first overview of the word frequencies. The word cloud displays the data of the aggregated sample file.


```r
trigramTDM <- TermDocumentMatrix(finalCorpus)
wcloud <- as.matrix(trigramTDM)
v <- sort(rowSums(wcloud),decreasing=TRUE)
d <- data.frame(word = names(v),freq=v)
wordcloud(d$word,d$freq,
          c(5,.3),50,
          random.order=FALSE,
          colors=brewer.pal(8, "Dark2"))
```

![](MilestoneReport_files/figure-html/unnamed-chunk-13-1.png)


# Building A Clean Text Corpus

By using the [tm package](http://tm.r-forge.r-project.org/index.html) the sample data gets *cleaned*. With cleaning it is meant that the text data is converted into lower case, further punction, numbers and URLs are getting removed. Next to that stop and profanity words are erased from the text sample. At the end we are getting a clean text corpus which enables an easy subsequent processing.

The used profanity words can be inspected [in this Github Repository](https://github.com/mhnierhoff/CapstoneCoursera/blob/master/MilestoneReport/profanityfilter.txt).


```r
## Make it work with the new tm package
cleanSample <- tm_map(cleanSample, content_transformer(function(x) iconv(x, to="UTF-8", sub="byte")), 
                      mc.cores=2)
cleanSample <- tm_map(cleanSample, content_transformer(tolower), lazy = TRUE)
cleanSample <- tm_map(cleanSample, content_transformer(removePunctuation))
cleanSample <- tm_map(cleanSample, content_transformer(removeNumbers))
removeURL <- function(x) gsub("http[[:alnum:]]*", "", x) 
cleanSample <- tm_map(cleanSample, content_transformer(removeURL))
cleanSample <- tm_map(cleanSample, stripWhitespace)
cleanSample <- tm_map(cleanSample, removeWords, stopwords("english"))
cleanSample <- tm_map(cleanSample, removeWords, profanityWords)
cleanSample <- tm_map(cleanSample, stemDocument)
cleanSample <- tm_map(cleanSample, stripWhitespace)
```





## The N-Gram Tokenization

In Natural Language Processing (NLP) an *n*-gram is a contiguous sequence of n items from a given sequence of text or speech.

The following function is used to extract 1-grams, 2-grams and 2-grams from the cleaned text corpus.


```r
ngramTokenizer <- function(theCorpus, ngramCount) {
        ngramFunction <- NGramTokenizer(theCorpus, 
                                Weka_control(min = ngramCount, max = ngramCount, 
                                delimiters = " \\r\\n\\t.,;:\"()?!"))
        ngramFunction <- data.frame(table(ngramFunction))
        ngramFunction <- ngramFunction[order(ngramFunction$Freq, 
                                             decreasing = TRUE),][1:10,]
        colnames(ngramFunction) <- c("String","Count")
        ngramFunction
}
```

By the usage of the tokenizer function for the *n*-grams a distribution of the following top 10 words and word combinations can be inspected. Unigrams are single words, while bigrams are two word combinations and trigrams are three word combinations.

### Top Unigrams

```r
unigram <- readRDS("./unigram.RDS")
unigramPlot <- gvisColumnChart(unigram, "String", "Count",                  
                            options=list(legend="none"))

print(unigramPlot, "chart")
```

<!-- ColumnChart generated in R 3.2.3 by googleVis 0.5.10 package -->
<!-- Mon May  2 14:38:22 2016 -->


<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataColumnChartID259e75566963 () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "said",
1531 
],
[
 "one",
1409 
],
[
 "will",
1390 
],
[
 "like",
1192 
],
[
 "just",
1168 
],
[
 "get",
1129 
],
[
 "time",
1054 
],
[
 "year",
1045 
],
[
 "go",
1005 
],
[
 "can",
990 
] 
];
data.addColumn('string','String');
data.addColumn('number','Count');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartColumnChartID259e75566963() {
var data = gvisDataColumnChartID259e75566963();
var options = {};
options["allowHtml"] = true;
options["legend"] = "none";

    var chart = new google.visualization.ColumnChart(
    document.getElementById('ColumnChartID259e75566963')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "corechart";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartColumnChartID259e75566963);
})();
function displayChartColumnChartID259e75566963() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartColumnChartID259e75566963"></script>
 
<!-- divChart -->
  
<div id="ColumnChartID259e75566963" 
  style="width: 500; height: automatic;">
</div>

### Top Bigrams

```r
bigram <- readRDS("./bigram.RDS")
bigramPlot <- gvisColumnChart(bigram, "String", "Count",                  
                            options=list(legend="none"))

print(bigramPlot, "chart")
```

<!-- ColumnChart generated in R 3.2.3 by googleVis 0.5.10 package -->
<!-- Mon May  2 14:38:22 2016 -->


<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataColumnChartID259e626ad3df () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "last year",
97 
],
[
 "new york",
90 
],
[
 "right now",
81 
],
[
 "look like",
80 
],
[
 "year ago",
80 
],
[
 "dont know",
69 
],
[
 "last week",
67 
],
[
 "high school",
59 
],
[
 "feel like",
57 
],
[
 "first time",
55 
] 
];
data.addColumn('string','String');
data.addColumn('number','Count');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartColumnChartID259e626ad3df() {
var data = gvisDataColumnChartID259e626ad3df();
var options = {};
options["allowHtml"] = true;
options["legend"] = "none";

    var chart = new google.visualization.ColumnChart(
    document.getElementById('ColumnChartID259e626ad3df')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "corechart";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartColumnChartID259e626ad3df);
})();
function displayChartColumnChartID259e626ad3df() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartColumnChartID259e626ad3df"></script>
 
<!-- divChart -->
  
<div id="ColumnChartID259e626ad3df" 
  style="width: 500; height: automatic;">
</div>

### Top Trigrams

```r
trigram <- readRDS("./trigram.RDS")
trigramPlot <- gvisColumnChart(trigram, "String", "Count",                  
                            options=list(legend="none"))

print(trigramPlot, "chart")
```

<!-- ColumnChart generated in R 3.2.3 by googleVis 0.5.10 package -->
<!-- Mon May  2 14:38:23 2016 -->


<!-- jsHeader -->
<script type="text/javascript">
 
// jsData 
function gvisDataColumnChartID259e73ee0d56 () {
var data = new google.visualization.DataTable();
var datajson =
[
 [
 "let us know",
10 
],
[
 "presid barack obama",
10 
],
[
 "cant wait see",
8 
],
[
 "new york citi",
8 
],
[
 "happi mother day",
7 
],
[
 "osama bin laden",
7 
],
[
 "two year ago",
7 
],
[
 "dont even know",
6 
],
[
 "execut order issu",
6 
],
[
 "ive ever seen",
6 
] 
];
data.addColumn('string','String');
data.addColumn('number','Count');
data.addRows(datajson);
return(data);
}
 
// jsDrawChart
function drawChartColumnChartID259e73ee0d56() {
var data = gvisDataColumnChartID259e73ee0d56();
var options = {};
options["allowHtml"] = true;
options["legend"] = "none";

    var chart = new google.visualization.ColumnChart(
    document.getElementById('ColumnChartID259e73ee0d56')
    );
    chart.draw(data,options);
    

}
  
 
// jsDisplayChart
(function() {
var pkgs = window.__gvisPackages = window.__gvisPackages || [];
var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
var chartid = "corechart";
  
// Manually see if chartid is in pkgs (not all browsers support Array.indexOf)
var i, newPackage = true;
for (i = 0; newPackage && i < pkgs.length; i++) {
if (pkgs[i] === chartid)
newPackage = false;
}
if (newPackage)
  pkgs.push(chartid);
  
// Add the drawChart function to the global list of callbacks
callbacks.push(drawChartColumnChartID259e73ee0d56);
})();
function displayChartColumnChartID259e73ee0d56() {
  var pkgs = window.__gvisPackages = window.__gvisPackages || [];
  var callbacks = window.__gvisCallbacks = window.__gvisCallbacks || [];
  window.clearTimeout(window.__gvisLoad);
  // The timeout is set to 100 because otherwise the container div we are
  // targeting might not be part of the document yet
  window.__gvisLoad = setTimeout(function() {
  var pkgCount = pkgs.length;
  google.load("visualization", "1", { packages:pkgs, callback: function() {
  if (pkgCount != pkgs.length) {
  // Race condition where another setTimeout call snuck in after us; if
  // that call added a package, we must not shift its callback
  return;
}
while (callbacks.length > 0)
callbacks.shift()();
} });
}, 100);
}
 
// jsFooter
</script>
 
<!-- jsChart -->  
<script type="text/javascript" src="https://www.google.com/jsapi?callback=displayChartColumnChartID259e73ee0d56"></script>
 
<!-- divChart -->
  
<div id="ColumnChartID259e73ee0d56" 
  style="width: 500; height: automatic;">
</div>


# Interesting Findings

+ Loading the dataset costs a lot of time. The processing is time consuming because of the huge file size of the dataset. By avoiding endless runtimes of the code, it was indispensable to create a data sample for text mining and tokenization. Nedless to say, this workaround decreases the accuracy for the subsequent predictions.

+ Removing all stopwords from the corpus is recommended, but, of course, stopwords are a fundamental part of languages. Therefore, consideration should be given to include these stop words in the prediction application again.

+ The text mining algorithm needs to be adjusted, so to speak a kind of fine-tuning. As seen in the chart of the top trigrams some words severely curtailed. For example, the second most common trigram is *presid barack obama* instead of *president barack obama*.

# Next Steps For The Prediction Application

As already noted, the next step of the capstone project will be to create a prediction application. 
To create a smooth and fast application it is absolutely necessary to build a fast prediction algorithm. This is also means, I need to find ways for a faster processing of larger datasets. Next to that,  increasing the value of n for n-gram tokenization will improve the prediction accuracy. All in all a shiny application will be created which will be able to predict the next word a user wants to write.

# All Used Code Scripts

All used code snippets to generate this report can be viewed in this [repository](https://github.com/jontieh/CapstoneCoursera/tree/master/MilestoneReport).

# Session Informations

```r
sessionInfo()
```

```
## R version 3.2.3 (2015-12-10)
## Platform: x86_64-apple-darwin15.3.0 (64-bit)
## Running under: OS X 10.11.4 (El Capitan)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
##  [1] googleVis_0.5.10       stringi_1.0-1          DT_0.1                
##  [4] stringr_1.0.0          wordcloud_2.5          rJava_0.9-8           
##  [7] RWeka_0.4-26           slam_0.1-32            SnowballC_0.5.1       
## [10] tm_0.6-2               NLP_0.1-9              qdap_2.2.4            
## [13] RColorBrewer_1.1-2     qdapTools_1.3.1        qdapRegex_0.6.0       
## [16] qdapDictionaries_1.0.6 RWekajars_3.7.13-1    
## 
## loaded via a namespace (and not attached):
##  [1] gtools_3.5.0        venneuler_1.1-0     reshape2_1.4.1     
##  [4] reports_0.1.4       colorspace_1.2-6    htmltools_0.3.5    
##  [7] yaml_2.1.13         chron_2.3-47        XML_3.98-1.4       
## [10] DBI_0.3.1           plyr_1.8.3          munsell_0.4.3      
## [13] gtable_0.2.0        htmlwidgets_0.6     evaluate_0.8.3     
## [16] knitr_1.12.3        gender_0.5.1        parallel_3.2.3     
## [19] xlsxjars_0.6.1      highr_0.5.1         Rcpp_0.12.4        
## [22] scales_0.4.0        formatR_1.3         gdata_2.17.0       
## [25] plotrix_3.6-1       xlsx_0.5.7          openNLPdata_1.5.3-2
## [28] gridExtra_2.2.1     ggplot2_2.1.0       digest_0.6.9       
## [31] dplyr_0.4.3         RJSONIO_1.3-0       grid_3.2.3         
## [34] tools_3.2.3         bitops_1.0-6        magrittr_1.5       
## [37] RCurl_1.95-4.8      data.table_1.9.6    assertthat_0.1     
## [40] rmarkdown_0.9.5     openNLP_0.2-6       R6_2.1.2           
## [43] igraph_1.0.1
```


