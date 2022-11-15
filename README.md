# bigdatafinalproject
## About:
- [Bhaskharm](https://github.com/Bhaskar2909)
## Text:
- [The Project Gutenberg EBook of Under the Law, by Edwina Stanton Babcock](https://www.gutenberg.org/files/48009/48009-0.txt)

## Tools & Languages:
- Databricks Cloud Environment.
- Spark Processing Engine.
- PySpark.
- Python Programming Language.
## Process:
```
# fetching the text data from url of my choice
import urllib.request
stringInURL = "https://www.gutenberg.org/files/48009/48009-0.txt"
urllib.request.urlretrieve(stringInURL, "/tmp/bhaskar.txt")

# moving file from tmp folder to data folder of dbfs.
dbutils.fs.mv("file:/tmp/bhaskar.txt", "dbfs:/data/bhaskar.txt")

# Tranfering the file to the Spark job.
bhaskarfinal_RDD = sc.textFile("dbfs:/data/bhaskar.txt")

# Splitting the words with the " " and conevrting them into the lower case.
bhaskartextRDD = bhaskarfinal_RDD.flatMap(lambda line : line.lower().strip().split(" "))

# map() words to (words,1) intermediate key-value pairs.
import re
# removing punctutations.
bhaskarcleanToken_RDD = bhaskartextRDD.map(lambda words: re.sub(r'[^a-zA-Z]','',words))
#prepare to clean stopwords
from pyspark.ml.feature import StopWordsRemover
remove =StopWordsRemover()
stopWords = remove.getStopWords()
bhaskarcleanData_RDD=bhaskarcleanToken_RDD.filter(lambda wrds: wrds not in stopWords)
# Removing the spaces/emptywords
bhaskarFinal_clean_DataRDD = bhaskarcleanData_RDD.filter(lambda x: x != "")
#maps the words to key value pairs
bhaskarIKVPairsRDD= bhaskarFinal_clean_DataRDD.map(lambda word: (word,1))

# Converting the keyvalue pairs to word count.
bhaskarFinal_word_count_RDD = bhaskarIKVPairsRDD.reduceByKey(lambda acc, value: acc+value)

# Sorting the words in the descending order and printing the results to check the first 25 results in descending order.
bhaskarfinal_results = bhaskarFinal_word_count_RDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(25)
print(bhaskarfinal_results)

# collect() action to get back to python
results = bhaskarFinal_word_count_RDD.collect()
print(results)
```
## Charting the results:
- We use pandas, matplotlib seaborn to chart the results.
```

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from collections import Counter

source = 'The Project Gutenberg EBook of Under the Law, by Edwina Stanton Babcock'
title = 'Top Words in ' + source
xlabel = 'Count'
ylabel = 'Words'

# create Pandas dataframe from list of tuples
df = pd.DataFrame.from_records(bhaskarfinal_results, columns =[xlabel, ylabel]) 
print(df)

# create plot (using matplotlib)
plt.figure(figsize=(15,15))
sns.barplot(xlabel, ylabel, data=df, palette="Paired").set_title(title)

```

![PHOTO](BARGRAPH.PNG)


## References:
- https://pypi.org/project/pyspark/
- https://docs.python.org/3/tutorial/
