# Bigdata-Project-Rohitha
## Author
- Rohitha Reddy Meda
## Text data
Source of text data: [Homecoming Horde](https://www.gutenberg.org/files/65119/65119-0.txt)
## Tools and Languages
Language: Python  
Tools: Pyspark, Databricks Notebook, Regex, Seaborn, Pandas, MatPlotLib, Urllib
## Process
## Gathering Data
1. To gather the data, first we will use urllib.request library to request or pull data from the data's url. After the data is pulled from the site i.e, homecoming horde, it is stored in a temporary file name 'rohitha.txt'
```Bash
# get text data from the url selected
import urllib.request
stringInURL = "https://www.gutenberg.org/files/65119/65119-0.txt"
urllib.request.urlretrieve(stringInURL, "/tmp/rohitha.txt")
```
2. After gathering the data, the data gathered need to be saved. To do that we will be using dbutils.fs.mv to transfer data from temporary file to a new site. The command returns a boolean value.
```Bash
# save the gathered data
dbutils.fs.mv("file:/tmp/rohitha.txt", "dbfs:/data/rohitha.txt")
```
3. The data need to be transferred from datafile to Spark, to do that we will be using sc.textfile into rohithaRDD(Resilent distributed datasets) is a fundamental data structure of spark. An RDD is an abstraction of a collection of data distributed on various node.
```Bash
rohithaRDD = sc.textFile("dbfs:/data/rohitha.txt")
```
## Cleaning the data
4. The data provided above has capital words, punctuations, stop words. To clean the data, first we need to split each line by he spaces, changing the capital words to lower case. Filtering out empty spaces and lines. 
```Bash
#flatMap each line to words 
wordsRDD=rohithaRDD.flatMap(lambda line : line.lower().strip().split(" "))
```
5. Removing punctuations. Here regular expression is used, to do that we need to import re library first. It views other that letters in the text.
```Bash
#remove punctuation
import re
cleanTokensRDD = wordsRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```
6. Next step is to remove stopwords. To do that we need to import StopWordsRemover library. 
```Bash
#remove stop words
from pyspark.ml.feature import StopWordsRemover
remove =StopWordsRemover()
stopwords = remove.getStopWords()
cleanword_RDD=cleanTokensRDD.filter(lambda word: word not in stopwords
```
## Processing the data
7. After cleaning the data, we can start processing the data. Mapping our words into intermediate key-value pairs using lambda function.
```Bash
#maps the words to key value pairs
IKVPairsRDD= cleanword_RDD.map(lambda word: (word,1))
```
8. Here, words are converted to word count using word and count as keys.
```Bash
# reduceByKey() to get count
wordcount_RDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc+value)
```
9. Here, we can retrieve the elements using collect() function. 
```Bash
results = wordcount_RDD.collect()
```
## Charting the results
The commands used for charting the results is provided below
```Bash
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from collections import Counter
source = 'The Project Gutenberg eBook of Homecoming Horde, by Robert Silverberg'
xlabel = 'Words'
ylabel = 'Count'
mostCommon = results[1:5]
xlabel,ylabel = zip(*mostCommon)
# create plot (using matplotlib)
plt.figure(figsize=(5,5))
plt.bar(xlabel, ylabel, color="pink")
plt.xlabel("Words")
plt.ylabel("Count")
plt.title("Commonly used words in novel")
```
![Image](https://github.com/Rohitha12/bigdata-project/blob/main/barchart.PNG)
## References
- https://www.edureka.co/blog/spark-with-python-pyspark
- https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/4574377819293972/2246755934805346/3186223000943570/latest.html
