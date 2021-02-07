---
title: kaggle-Real or Not? NLP with Disaster Tweets ①
date: 2020-11-23 23:05:36
tags: [데이터 사이언스, 자연어 처리, kaggle, NLP, disater_tweets]
category: kaggle
mathjax: true
---

# Real or Not? NLP with Disaster Tweets

## NLP Getting Stated Tutorial




```
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)
from sklearn import feature_extraction, linear_model, model_selection, preprocessing
```


```
train_df = pd.read_csv('/content/drive/MyDrive/kaggle/tweetDisater/data/train.csv')
test_df=pd.read_csv('/content/drive/MyDrive/kaggle/tweetDisater/data/test.csv')

```

an example of what is not a disater tweet


```
train_df[train_df["target"]==0]["text"].values[1]
```




    'I love fruits'



an example of what is a disater tweet


```
train_df[train_df["target"]==1]["text"].values[1]
```




    'Forest fire near La Ronge Sask. Canada'



CountVectorizer
: 단어들의 카운트(출현 빈도(frequency))로 여러 문서들을 벡터화
카운트 행렬, 단어 문서 행렬 (Term-Document Matrix, TDM))
모두 소문자로 변환시키기 때문에 me 와 Me 는 모두 같은 특성이 된다.


```
count_vectorizer = feature_extraction.text.CountVectorizer()

#처음 다섯개 트위터 count
example_train_vectors = count_vectorizer.fit_transform(train_df["text"][0:5])
```


```
print(example_train_vectors[0].todense().shape)
print(example_train_vectors[0].todense())
```

    (1, 54)
    [[0 0 0 1 1 1 0 0 0 0 0 0 1 1 0 0 0 0 1 0 0 0 0 0 0 1 0 0 0 1 0 0 0 0 1 0
      0 0 0 1 0 0 0 0 0 0 0 0 0 1 1 0 1 0]]


1. 처음 5개 트윗에 54개의 단어(tokens)들이 있음
2. 첫번째 트윗은 54개의 unique words 중 몇개만 포함함. 0이 아닌것은 첫번째 트위터에 존재하지 않는 단어(token)

모든 트윗에 대해 vetors 만듬 


```
train_vectors = count_vectorizer.fit_transform(train_df["text"])

test_vectors = count_vectorizer.transform(test_df["text"])
```


```

clf = linear_model.RidgeClassifier()
```


```
scores = model_selection.cross_val_score(clf, train_vectors, train_df["target"], cv =3, scoring="f1")
scores
```




    array([0.59453669, 0.56498283, 0.64082434])




```
clf.fit(train_vectors, train_df["target"])
```




    RidgeClassifier(alpha=1.0, class_weight=None, copy_X=True, fit_intercept=True,
                    max_iter=None, normalize=False, random_state=None,
                    solver='auto', tol=0.001)




```
sample_submisstion = pd.read_csv("/content/drive/MyDrive/kaggle/tweetDisater/data/sample_submission.csv")
```


```
sample_submisstion.head()
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
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```
sample_submisstion["target"] = clf.predict(test_vectors)
```


```
sample_submisstion.head()
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
      <th>target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```
sample_submisstion.to_csv("submission.csv", index=False)
```



```

```
