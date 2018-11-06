# Sentiment Analysis of Amazon Reviews

### overview:
A review text usually contains emotional information, which is very useful for evaluating the quality of a product. In this Project, we will train a model to classify a sentence to 3 sentiment: positive, negative and neutral. 



### dataset:
We will use the Amazon Customer Reviews Dataset, which is provided from Amazon. This dataset consists of many classes, and we will use [book review data]( http://snap.stanford.edu/data/amazon/ ) from them, which is about 4.4GB in size.This is a [link](https://s3.amazonaws.com/amazon-reviews-pds/readme.html) including all the information of those data. There are many attributes in this data set, however we will use mainly the review text as training data.

```
amazon_reviews(
product/productId int,
product/title string, 
product/price double,
review/userId string,
review/profileName string,
review/helpfulness int, 
review/score float,
review/time: int,
review/summary string,
review/text string)
```

Here is an example:
```
product/productId: 0826414346
product/title: Dr. Seuss: American Icon
product/price: unknown
review/userId: A30TK6U7DNS82R
review/profileName: Kevin Killian
review/helpfulness: 10/10
review/score: 5.0
review/time: 1095724800
review/summary: Really Enjoyed It
review/text: I don't care much for Dr. Seuss but after reading Philip Nel's book I changed my mind--that's a good testimonial to the power of Rel's writing and thinking. Rel plays Dr. Seuss the ultimate compliment of treating him as a serious poet as well as one of the 20th century's most interesting visual artists, and after reading his book I decided that a trip to the Mandeville Collections of the library at University of California in San Diego was in order, so I could visit some of the incredible Seuss/Geisel holdings they have there.There's almost too much to take in, for, like William Butler Yeats, Seuss led a career that constantly shifted and metamoprhized itself to meet new historical and political cirsumstances, so he seems to have been both a leftist and a conservative at different junctures of his career, both in politics and in art. As Nel shows us, he was once a cartoonist for the fabled PM magazine and, like Andy Warhol, he served his time slaving in the ad business too. All was in the service of amusing and broadening the minds of US children. Nel doesn't hesitate to administer a sound spanking to the Seuss industry that, since his death, has seen fit to license all kinds of awful products including the recent CAT IN THE HAT film with Mike Myers. Oh, what a cat-astrophe!The book is great and I can especially recommend the work of the picture editor who has given us a bounty of good illustrations.
```



### method:
We will use [CNN]( https://www.aclweb.org/anthology/D14-1181 ) as main model for classification, which is proved as a powerful model to solve sentiment analysis problem. And we use [Glove](https://nlp.stanford.edu/projects/glove/) and [SSWE](https://www.aclweb.org/anthology/P/P14/P14-1146.xhtml) as word embedding, which is also a important part of NLP problem. In order to compare the result, we will also implement a Naive Bayes model and evaluate it.

### plan:
We divided the whole project into 3 part: data process, training and evaluating Naive Bayes model, training and evaluating CNN model.

| Name | Work |
|:----:|:------:|
|Xi  | data process |
|Martin | Naive Bayes model |
|Ziyuan | CNN model|

### part1. data process:
reference: [How to Clean Text for Machine Learning with Python](https://machinelearningmastery.com/clean-text-machine-learning-python/?spm=5176.100239.blogcont225721.32.cAebmf)


In this part, we choose the `review/text` from data set as the text we use to classify; and use `review/score` as label following below 
condition: 
```
if score >= 4.0 label the text as "positive"
if score == 3.0 label the text as "neutral"
if score <= 2.0 label the text as "negative"
```

In order to use a text to do a classification, the first step is to clear up the text, and it consists of 4 main part:
```
1. read text;
2. spilt the text paragrap to sentence;
3. tokenize the sentence to words;
4. extend the abbreviations (eg. "isn't -> is not");
5. convert all words to lower-case, remove stop words and punctuation;
6. write the cleared text to file
```

Besides, according to different classifier(NB or CNN), there is also small difference:
in CNN, we should keep the original form of words, because different forms such as "-ing" or "-s" have different means, and with word embedding these information can be used in CNN model. But for Naive Bayes model, we can't use these information, so usually we will stem them (eg. liking -> like, words -> word ).

Here is an example :

original sentence:
```
["This is only for Julie Strain fans. It's a collection of her photos -- about 80 pages worth with a nice section of paintings by Olivia.If you're looking for heavy literary content, this isn't the place to find it -- there's only about 2 pages with text and everything else is photos.Bottom line: if you only want one book, the Six Foot One ... is probably a better choice, however, if you like Julie like I like Julie, you won't go wrong on this one either."]
```
cleared-up sentence ( for CNN ):
```
[[['julie', 'strain', 'fans'], ['collection', 'photos', 'pages', 'worth', 'nice', 'section', 'paintings', 'looking', 'heavy', 'literary', 'content', 'place', 'find', 'pages', 'text', 'everything', 'else', 'line', 'want', 'one', 'book', 'six', 'foot', 'one', 'probably', 'better', 'choice', 'however', 'like', 'julie', 'like', 'like', 'julie', 'go', 'wrong', 'one', 'either']]]
```
cleared-up sentence ( for Naive Bayes ):
```
[['julie', 'strain', 'fan', 'collection', 'photo', 'page', 'worth', 'nice', 'section', 'painting', 'look', 'heavy', 'literary', 'content', 'place', 'find', 'page', 'text', 'everything', 'else', 'line', 'want', 'one', 'book', 'six', 'foot', 'one', 'probably', 'good', 'choice', 'however', 'like', 'julie', 'like', 'like', 'julie', 'go', 'wrong', 'one', 'either']]
```

Second step is to map those words to a indexes: we make a dictionary, including every word which appears in text and assign a index for every word. We also created 2 dictionaries, one of them, i.e. id2word, map indexes to words, another,i.e. word2id, map words to indexes. We can use them later on building classifiers. The structure and description of result after processing can be shown as following:

CNN:

| file name | type | description |
|:----:|:----:|:------:|
|CNN_id2word.txt | dict() | dictionary, use id to find word |
|CNN_word2id.txt | dict() | dictionary, use word to find id |
|CNN_sentence_id_list.txt | list() | 3d list, every word in sentence convert to index  |
|CNN_sentence_list.txt | list() | 3d list, see example "cleared-up sentence for CNN" above|

Naive Bayes:

| file name | type | description |
|:----:|:----:|:------:|
|NB_id2word.txt | dict() | dictionary, use id to find word |
|NB_word2id.txt | dict() | dictionary, use word to find id |
|NB_sentence_id_list.txt | list() | 2d list, every word in sentence convert to index  |
|NB_sentence_list.txt | list() | 2d list, see example "cleared-up sentence for Naive Bayes" above|

all the data can be loaded by following python code:
``` python
import json
def ReadListAndDictFromFile(readFileName):
    with open(readFileName, "r",encoding='UTF-8') as f:
        readedList =  json.loads(f.read())
        return readedList
        
#for example
ReadListAndDictFromFile("CNN_sentence_id_list.txt")
```

Then we will encode those word list to numerical type. based on different classification algorithm, the final text will look a little different. For Naive Bayes, using index as encoder is enough. For CNN, we need a more complex method called "word embedding" to encode. In this project, we will use [Glove](https://pan.baidu.com/s/1qX9uVTE) and [SSWE](https://pan.baidu.com/s/1jIoOFRK) as word embedding.



 
 
