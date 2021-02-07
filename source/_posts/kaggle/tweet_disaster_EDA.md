---
title: kaggle-Real or Not? NLP with Disaster Tweets ②-Basic EDA
date: 2020-11-28 17:42:36
tags: [데이터 사이언스, 자연어 처리, kaggle, NLP, disater_tweets]
category: kaggle
mathjax: true
---

# Real or Not? NLP with Disaster Tweets②-Basic EDA


### Importing required Libraries.

가장 먼저 필요한 라이브러리들을 import 합니다.


```
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from nltk.corpus import stopwords
from nltk.util import ngrams
from sklearn.feature_extraction.text import CountVectorizer
from collections import defaultdict
from collections import Counter
plt.style.use('ggplot')
stop=set(stopwords.words('english'))
import re
from nltk.tokenize import word_tokenize
import gensim
import string
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences
from tqdm import tqdm
from keras.models import Sequential
from keras.layers import Embedding, LSTM, Dense, SpatialDropout1D
from keras.initializers import Constant
from sklearn.model_selection import train_test_split
from keras.optimizers import Adam
```


```
import nltk
nltk.download("stopwords")
```

    [nltk_data] Downloading package stopwords to /root/nltk_data...
    [nltk_data]   Package stopwords is already up-to-date!





    True




```
import os
```


```
tweet = pd.read_csv('/content/drive/MyDrive/kaggle/tweetDisater/data/train.csv')
test = pd.read_csv('/content/drive/MyDrive/kaggle/tweetDisater/data/test.csv')
tweet.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }


    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>keyword</th>
      <th>location</th>
      <th>text</th>
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Our Deeds are the Reason of this #earthquake M...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Forest fire near La Ronge Sask. Canada</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>All residents asked to 'shelter in place' are ...</td>
      <td>1</td>
    </tr>
  </tbody>
</table>

</div>




```
x = tweet.target.value_counts()
sns.barplot(x.index, x)
plt.gca().set_ylabel('samples')
```

    /usr/local/lib/python3.6/dist-packages/seaborn/_decorators.py:43: FutureWarning: Pass the following variables as keyword args: x, y. From version 0.12, the only valid positional argument will be `data`, and passing other arguments without an explicit keyword will result in an error or misinterpretation.
      FutureWarning





    Text(0, 0.5, 'samples')




​    
![input_7_2](https://user-images.githubusercontent.com/33755241/100497720-ac4a2200-31a0-11eb-8225-5b3b9809b4bc.png)
​    


결과를 보니까 class 0(No Disaster)이 class 1(disaster tweets)보다 더 많네요.

### Exploratory Data Analysis of tweets

  #### Number of characters in **tweets**


```
fig, (ax1, ax2) = plt.subplots(1,2,figsize=(10,5)) #figsize = 그림의 가로 세로 인치 단위
tweet_len = tweet[tweet['target'] == 1]['text'].str.len()
ax1.hist(tweet_len, color='red')
ax1.set_title('disater tweets')
tweet_len = tweet[tweet['target']==0]['text'].str.len()
ax2.hist(tweet_len, color='green')
ax2.set_title('Not disaster tweets')
fig.suptitle('Character in tweets')
plt.show()
```


​    
![input_11_0](https://user-images.githubusercontent.com/33755241/100497723-ace2b880-31a0-11eb-9aff-86067e0e9f17.png)
​    


#### Number of words in a tweet


```
fig, (ax1, ax2) = plt.subplots(1,2, figsize=(10,5)) #두개 그래프 설정을 한번에 subplots
tweet_len = tweet[tweet['target']==1]['text'].str.split().map(lambda x : len(x))
ax1.hist(tweet_len, color='red')
ax1.set_title('disater tweets')
tweet_len = tweet[tweet['target']==0]['text'].str.split().map(lambda x: len(x))
ax2.hist(tweet_len,color='green')
ax1.set_title('Not disater tweets')
fig.suptitle('Words in a tweet')
plt.show()
```


​    
![input_13_0](https://user-images.githubusercontent.com/33755241/100497724-ae13e580-31a0-11eb-8129-33c732664371.png)
​    


#### Average word length in a tweet


```
fig, (ax1, ax2) = plt.subplots(1,2,figsize=(10,5))
word=tweet[tweet['target']==1]['text'].str.split().apply(lambda x : [len(i) for i in x])
sns.distplot(word.map(lambda x : np.mean(x)), ax = ax1, color='red') # 막대 그래프 + mean값으로 선
ax1.set_title('disaster')
word = tweet[tweet['target']==0]['text'].str.split().apply(lambda x : [len(i) for i in x])
sns.distplot(word.map(lambda x : np.mean(x)), ax = ax2, color = 'green')
ax2.set_title('Not disaster')
fig.suptitle('Average word lentgh in a tweet')
```

    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2551: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2551: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)





    Text(0.5, 0.98, 'Average word lentgh in a tweet')




​    
![input_15_2](https://user-images.githubusercontent.com/33755241/100497726-af451280-31a0-11eb-99e5-e79d129ebaed.png)
​    



```
def create_corpus(target):
    corpus=[]
    
    for x in tweet[tweet['target']==target]['text'].str.split():
        for i in x:
            corpus.append(i)
    return corpus #자연언어 연구를 위해 특정한 목적을 가지고 언어의 표본을 추출한 집합

```

#### Common stopwords in tweets

- stopword(불용어)
  : 갖고 있는 데이터에서 유의미한 단어 토큰만을 선별하기 위해서는 큰 의미가 없는 단어 토큰을 제거하는 작업이 필요함. 여기서 큰 의미가 없다라는 것은 자주 등장하지만 분석을 하는 것에 있어서는 큰 도움이 되지 않는 단어들을 말함.

class 0 부터 분석 


```
corpus = create_corpus(0)

dic = defaultdict(int) # 디폴트 값이 int인 딕셔너리.
for word in corpus:
  if word in stop:
    dic[word]+=1
    

top = sorted(dic.items(), key=lambda x :x [1], reverse = True) [:10]
```


```
x,y = zip(*top)
plt.bar(x,y)
```




    <BarContainer object of 10 artists>




​    
![input_20_1](https://user-images.githubusercontent.com/33755241/100497727-afdda900-31a0-11eb-8fe0-a7ff5d5d8020.png)
​    



```
corpus = create_corpus(1)

dic = defaultdict(int) # 디폴트 값이 int인 딕셔너리.
for word in corpus:
  if word in stop:
    dic[word]+=1
    

top = sorted(dic.items(), key=lambda x :x [1], reverse = True) [:10]

x,y = zip(*top)
plt.bar(x,y)
```




    <BarContainer object of 10 artists>








### Analyzing punctuations


```
plt.figure(figsize=(10,5))
corpus = create_corpus(1)

dic = defaultdict(int)
import string
special = string.punctuation
for i in (corpus) : 
  if i in special : 
    dic[i]+=1

x,y = zip(*dic.items())
plt.bar(x,y)
```




    <BarContainer object of 18 artists>




​    
![input_24_1](https://user-images.githubusercontent.com/33755241/100497728-b1a76c80-31a0-11eb-9dfa-9a975187b505.png)
​    



```
plt.figure(figsize=(10,5))
corpus = create_corpus(0)

dic = defaultdict(int)
import string
special = string.punctuation
for i in (corpus) : 
  if i in special : 
    dic[i]+=1

x,y = zip(*dic.items())
plt.bar(x,y, color = 'green')
```




    <BarContainer object of 20 artists>




​    
![input_25_1](https://user-images.githubusercontent.com/33755241/100497731-b3713000-31a0-11eb-97a5-c6dcf1d3d29a.png)
​    


### Common words?


```
counter = Counter(corpus)
most = counter.most_common()
x = []
y = []
for word, count in most [:40]:
  if (word not in stop) :
    x.append(word)
    y.append(count)
```


```
sns.barplot(x=y, y=x)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f8bb4990cc0>




​    
![input_28_1](https://user-images.githubusercontent.com/33755241/100497733-b66c2080-31a0-11eb-8a6f-da32b9f47f60.png)
​    


### Ngram analysis

bigram (n=2)


```
def get_top_tweet_bigrams(corpus, n = None):
  vec = CountVectorizer(ngram_range = (2,2)).fit(corpus)
  bag_of_words = vec.transform(corpus)
  sum_words = bag_of_words.sum(axis=0)
  words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
  words_freq = sorted(words_freq, key = lambda x: x[1], reverse = True)
  return words_freq[:n]
```


```
plt.figure(figsize=(10,5))
top_tweet_bigrams = get_top_tweet_bigrams(tweet['text'])[:10]
x,y = map(list, zip(*top_tweet_bigrams))
sns.barplot(x=y, y=x)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f8bb3f4e9b0>




​    
![input_32_1](https://user-images.githubusercontent.com/33755241/100497734-b79d4d80-31a0-11eb-8ee3-30645ee98563.png)
​    


### Data Cleaning




```
df = pd.concat([tweet, test])
df.shape
```




    (10876, 5)



#### Removing urls


```
example="New competition launched :https://www.kaggle.com/c/nlp-getting-started"
```


```
def remove_URL(text):
    url = re.compile(r'https?://\S+|www\.\S+')
    return url.sub(r'',text)

remove_URL(example)
```




    'New competition launched :'




```
df['text'] = df['text'].apply(lambda x : remove_URL(x))
```


```
example = """<div>
<h1>Real or Fake</h1>
<p>Kaggle </p>
<a href="https://www.kaggle.com/c/nlp-getting-started">getting started</a>
</div>"""
```


```
def remove_html(text):
    html=re.compile(r'<.*?>')
    return html.sub(r'',text)
print(remove_html(example))
```


    Real or Fake
    Kaggle 
    getting started


​    


```
df['text']=df['text'].apply(lambda x : remove_html(x))
```

#### Removing Emojis


```
# Reference : https://gist.github.com/slowkow/7a7f61f495e3dbb7e3d767f97bd7304b

def remove_emoji(text):
    emoji_pattern = re.compile("["
                           u"\U0001F600-\U0001F64F"  # emoticons
                           u"\U0001F300-\U0001F5FF"  # symbols & pictographs
                           u"\U0001F680-\U0001F6FF"  # transport & map symbols
                           u"\U0001F1E0-\U0001F1FF"  # flags (iOS)
                           u"\U00002702-\U000027B0"
                           u"\U000024C2-\U0001F251"
                           "]+", flags=re.UNICODE)
    return emoji_pattern.sub(r'', text)

remove_emoji("Omg another Earthquake 😔😔")
```




    'Omg another Earthquake '




```
df['text']=df['text'].apply(lambda x: remove_emoji(x))
```

#### Removing punctuations


```
def remove_punct(text):
    table=str.maketrans('','',string.punctuation)
    return text.translate(table)

example="I am a #king"
print(remove_punct(example))
```

    I am a king



```
df['text']=df['text'].apply(lambda x : remove_punct(x))
```

#### Spelling Correction


```
!pip install pyspellchecker
```

    Collecting pyspellchecker
    [?25l  Downloading https://files.pythonhosted.org/packages/f1/96/827c132397d0eb5731c1eda05dbfb019ede064ca8c7d0f329160ce0a4acd/pyspellchecker-0.5.5-py2.py3-none-any.whl (1.9MB)
    [K     |████████████████████████████████| 1.9MB 7.1MB/s 
    [?25hInstalling collected packages: pyspellchecker
    Successfully installed pyspellchecker-0.5.5



```
from spellchecker import SpellChecker

spell = SpellChecker()
def correct_spellings(text):
    corrected_text = []
    misspelled_words = spell.unknown(text.split())
    for word in text.split():
        if word in misspelled_words:
            corrected_text.append(spell.correction(word))
        else:
            corrected_text.append(word)
    return " ".join(corrected_text)
        
text = "corect me plese"
correct_spellings(text)
```




    'correct me please'




```
df['text']=df['text'].apply(lambda x : correct_spellings(x))
```

