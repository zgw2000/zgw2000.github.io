---
title: "Judicial decision on ECtHR Dataset"
excerpt: "The goal of this project is building a system able to predict judicial decision with the intent of showing to what extent these are predictable. The dataset and the starting point are taken from Judicial decisions of the European Court of Human Rights: Looking into the crystal ball, a paper written by Medvedeva et al. The dataset contains all english cases available on 09/2017 from both the Chamber and the Grand Chamber. The task consists in classifying violation or non-violation for 9 different articles of ECHR. Their approach was basically to train 9 different binary classifiers using Support Vector Machines, one per article to assess if there was a violation or non-violation. The training set was built using the same number of random cases for the two classes. Many information were removed from the text of the articles, for example specific sections like the part containing the final decision that sometimes could be explicit or other informations not available until the trial ended. The test sets contain only one class. Notwithstanding this the task is still to build a model able to classify between the two classes rather than just identifying if it belong to that single class<br/><img src='/images/ecthr.png'>"
collection: portfolio
---
<a id="top"></a>

***Alessio Barboni, Miro Confalone, Guowang Zeng***

The goal of this project is building a system able to predict judicial decisions with the intent of showing to what extent these are predictable. The dataset and the starting point are taken from Judicial decisions of the European Court of Human Rights: Looking into the crystal ball, a paper written by Medvedeva et al. The paper is available [`here`](http://martijnwieling.nl/files/Medvedeva-submitted.pdf).

The dataset contains all english cases available on 09/2017 from both the Chamber and the Grand Chamber, without making a distinction between these two. Being closed cases labels are present, thus it is a supervised learning problem. The task consists in classifying violation or non-violation for 14 different articles of ECHR.

Their approach was basically to train 9 different binary classifiers using Support Vector Machines, one per article (only 9 because some of them did not have enough instances for training), to assess if there was a violation or non-violation. The training set was built using the same number of random cases for the two classes (a sort of stratified sampling with 50%-50%). Many information were removed from the text of the articles, for example specific sections like the part containing the final decision (that could be explicit) or other informations not available until the trial ended. The test sets contain only one class, i.e. violation for all articles except the 14th (because there are not enough violation cases available). Notwithstanding this the task is still to build a model able to classify between the two classes rather than just identifying if it belong to that single class or not.

First we showed performances of the "model 0" that is downloadable from [`here`](https://github.com/masha-medvedeva/ECtHR_crystal_ball/blob/master/src/Experiment_1/pipeline_exp1.ipynb), then we applied some additional preprocessing techniques showing improvements in the accuracy, we tried a Naive Bayes approach (i.e. MultinomialNB) and at the end an Embedding approach (i.e. Doc2Vec).

```python
#Model 0
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline, FeatureUnion
import glob, re
from sklearn.model_selection import cross_val_predict
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix, precision_recall_fscore_support
from random import shuffle
from sklearn.svm import LinearSVC
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import os

def plot_coefficients(article, classifier, feature_names, top_features=15): #this function was added for making plots
    coef = classifier.coef_.ravel()
    top_positive_coefficients = np.argsort(coef)[-top_features:]
    top_negative_coefficients = np.argsort(coef)[:top_features]
    top_coefficients = np.hstack([top_negative_coefficients, top_positive_coefficients])
    # create plot
    plt.figure(figsize=(15, 5))
    colors = ['red' if c < 0 else 'blue' for c in coef[top_coefficients]]
    plt.bar(np.arange(2 * top_features), coef[top_coefficients], color=colors)
    feature_names = np.array(feature_names)
    plt.xticks(np.arange(1, 1 + 2 * top_features), feature_names[top_coefficients], rotation=60, ha='right')
    plt.tight_layout()
    plt.savefig(article+"-top_coefficients.png")
    plt.show()

def dump_csv(article, classifier, feature_names, Ypredict, Ytest_v, Xtest_v, top_features=15):
    # coef = classifier.coef_.ravel()
    # top_positive_coefficients = np.argsort(coef)[-top_features:]
    # top_negative_coefficients = np.argsort(coef)[:top_features]
    # top_coefficients = np.hstack([top_negative_coefficients, top_positive_coefficients])
    # feature_names = np.array(feature_names)
    df = pd.DataFrame({'prediction': Ypredict,
                              'true_label': Ytest_v,
                              'case_facts': Xtest_v})
    #df.to_csv(article + "-predictions-"+".csv")
    subset = df[df['true_label'] == df['prediction']]
    subset.to_csv(article + "-predictions-" + ".csv")
    #subset = df[df['case_facts'].str.contains(feature_names[top_coefficients][0])]
    #subset.to_csv(article + "-predictions-"+feature_names[top_coefficients][0]+".csv")


def extract_text(starts, ends, cases, violation):
    facts = []
    D = []
    years = []
    for case in cases:
        contline = ''
        year = 0
        with open(case, 'r', encoding="utf8") as f:
            for line in f:
                dat = re.search('^([0-9]{1,2}\s\w+\s([0-9]{4}))', line)
                if dat != None:
                    year = int(dat.group(2))
                    break
            if year>0:
                years.append(year)
                wr = 0
                for line in f:
                    if wr == 0:
                        if re.search(starts, line) != None:
                            wr = 1
                    if wr == 1 and re.search(ends, line) == None:
                        contline += line
                        contline += '\n'
                    elif re.search(ends, line) != None:
                        break
                facts.append(contline)
    for i in range(len(facts)):
        D.append((facts[i], violation, years[i]))
    return D


def extract_parts(train_path, violation, part):  # extract text from different parts
    cases = glob.glob(train_path)
    facts = []
    D = []
    years = []
    if part == 'relevant_law':  # seprarte extraction for relevant law
        for case in cases:
            year = 0
            contline = ''
            with open(case, 'r',encoding="utf8") as f:
                for line in f:
                    dat = re.search('^([0-9]{1,2}\s\w+\s([0-9]{4}))', line)
                    if dat != None:
                        year = int(dat.group(2))
                        break
                if year > 0:
                    years.append(year)
                    wr = 0
                    for line in f:
                        if wr == 0:
                            if re.search('RELEVANT', line) != None:
                                wr = 1
                        if wr == 1 and re.search('THE LAW', line) == None and re.search('PROCEEDINGS', line) == None:
                            contline += line
                            contline += '\n'
                        elif re.search('THE LAW', line) != None or re.search('PROCEEDINGS', line) != None:
                            break
                    facts.append(contline)
        for i in range(len(facts)):
            D.append((facts[i], violation, years[i]))

    if part == 'facts':
        starts = 'THE FACTS'
        ends = 'THE LAW'
        D = extract_text(starts, ends, cases, violation)
    if part == 'circumstances':
        starts = 'CIRCUMSTANCES'
        ends = 'RELEVANT'
        D = extract_text(starts, ends, cases, violation)
    if part == 'procedure':
        starts = 'PROCEDURE'
        ends = 'THE FACTS'
        D = extract_text(starts, ends, cases, violation)
    if part == 'procedure+facts':
        starts = 'PROCEDURE'
        ends = 'THE LAW'
        D = extract_text(starts, ends, cases, violation)
    return D

def evaluate(Ytest, Ypredict):  # evaluate the model (accuracy, precision, recall, f-score, confusion matrix)
    print('Accuracy:', accuracy_score(Ytest, Ypredict))
    print('\nClassification report:\n', classification_report(Ytest, Ypredict))
    print('\nCR:', precision_recall_fscore_support(Ytest, Ypredict, average='macro'))
    print('\nConfusion matrix:\n', confusion_matrix(Ytest, Ypredict), '\n\n_______________________\n\n')

def train_model_cross_val(Xtrain, Ytrain, vec, c):  # Linear SVC model cross-validation
    print('***10-fold cross-validation***')
    pipeline = Pipeline([
        ('features', FeatureUnion(
            [vec],
        )),
        ('classifier', LinearSVC(C=c))
    ])
    Ypredict = cross_val_predict(pipeline, Xtrain, Ytrain, cv=10)  # 10-fold cross-validation
    evaluate(Ytrain, Ypredict)

def train_model_test(Xtrain, Ytrain, Xtest_v, Ytest_v, vec, c, article):  # test on 'violations' test set
    pipeline = Pipeline([
        ('features', FeatureUnion([vec]
                                  )),
        ('classifier', LinearSVC(C=c))
    ])
    pipeline.fit(Xtrain, Ytrain)

    print('***testing on violation testset***')
    Ypredict = pipeline.predict(Xtest_v)
    evaluate(Ytest_v, Ypredict)

    # plot
    plot_coefficients(article, pipeline.named_steps['classifier'],
                      vec[1].get_feature_names())
    # dump
    dump_csv(article, pipeline.named_steps['classifier'],
             vec[1].get_feature_names(), Ypredict, Ytest_v, Xtest_v)


def run_pipeline(article, part, vec, c):  # run tests
    print('Trained on *' + part + '* part of the cases')

    v = extract_parts(path + 'train/' + article + '/violation/*.txt', 'violation', part)
    nv = extract_parts(path + 'train/' + article + '/non-violation/*.txt', 'non-violation', part)
    trainset = v + nv
    shuffle(trainset)

    Xtrain = [i[0] for i in trainset]
    Ytrain = [i[1] for i in trainset]

    # test set with violations only
    if article == 'Article14':
        test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'non-violation', part)
    else:
        test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'violation', part)
    Xtest_v = [i[0] for i in test]
    Ytest_v = [i[1] for i in test]

    print('Training on', Ytrain.count('violation'), '+', Ytrain.count('non-violation'), '=',
          Ytrain.count('violation') + Ytrain.count('non-violation'), 'cases',
          '\nCases available for testing(violation):', Ytest_v.count('violation'))

    #train_model_cross_val(Xtrain, Ytrain, vec, c)  # use for cross-validation
    train_model_test(Xtrain, Ytrain, Xtest_v, Ytest_v, vec, c, article)

if __name__ == "__main__":
    # Path to the Documents
    path = '/home/alessio/Desktop/aiii/crystal_ball_data/'
    os.chdir("/home/alessio/Desktop/aiii/crystal_ball_data/")
    articles = ['Article2', 'Article3', 'Article5', 'Article6', 'Article8', 'Article10', 'Article11', 'Article13', 'Article14']
    for article in articles: 
            print (article)
            vec = ('wordvec', TfidfVectorizer(analyzer = 'word', binary = True,  lowercase = True,  min_df = 4,  ngram_range =(4, 6),  norm = 'l2',  stop_words = 'english',  use_idf = True))
            c = 1
            run_pipeline(article, 'facts', vec, c)
```

In this case setting a seed is not necessary to get reproducible results since, despite the shuffle function applied on the train, we are training the model on the whole train set, not a subset of it.

The results are the following:

+ Article 2: Accuracy = 0.7236
+ Article 3: Accuracy = 0.7121
+ Article 5: Accuracy = 0.6967
+ Article 6: Accuracy = 0.7013
+ Article 8: Accuracy = 0.6612
+ Article 10: Accuracy = 0.6111
+ Article 11: Accuracy = 0.6067
+ Article 13: Accuracy = 0.6518
+ Article 14: Accuracy = 0.6363
  
Overall accuracy for these articles is approx 66.67%. We could not adopt the ROC AUC as performance metric because the test set is composed of just one class. The plot below represents the best set of predictors (n-grams in range 4-6) for violations (in blue) and non-violations (in red) for Article 2, the one that achived the highest accuracy in this setting.

![result](/images/nlp1.png )

Our goal now is to increase this accuracy.

## Text Preprocessing

We will use the NLTK package that is made ad hoc for Natural Language Processing, the steps will be the following:

1) Choosing the Scope of the Documents --> in our case the whole document would be appropriate for classification (usually subsets of it are preferred in sentiment analysis or document summarization tasks)

2) Tokenization --> breaking the document into tokens (i.e. words)

3) Stopping --> removing stopwords that are not meaningful for classification. Punctuation could be included.

4) Lemmatization --> reducing inflection of words and ensuring that the remaining root word (s.c. lemma) is a proper word in that language, using Part-of-Speech Tagging and a language dictionary. This is generally preferred over the Stemming approach that does not rely on pos-tags but simply uses some algorithms to shrink it, without checking whether the resulting word belongs to that language.

5) Case Normalization --> converting everything to either lower or upper case.

Another relevant step would be Spelling Normalization that has several pros and cons. Even the exact step in which it should be applied represents a sort of tradeoff. For example we could use it before stopping so that a typo like "aand" would be converted into "and" and thus it make the Pos-tagging part correct ("aand" would have been tagged based on context, even as a verb, "and" instead is always correclty tagged as conjunction and thus removed with stopping). A sentence like "I hte you" on the other hand, tagged in this way "('I', 'n'), ('hate', 'v'), ('you', 'n')", would become "('I', 'n'), ('the', 'v'), ('you', 'n')" thus it would be miscorrected and the verb would be even removed in the stepping phase (without additional modifications). It could be performed as the last step to correct avoidable errors (that would improve the counter effectiveness), e.g. "allapogize"="apologize". A minor drawback is that it would try to correct also Proper Nouns (without additional modifications) e.g. "Alessio"-->"Cessio", but these words are not expected to have relevance for the classification task.

The following code has been used for assesing these decisions:

```python
import re
def reduce_lengthening(text):
    pattern = re.compile(r"(.)\1{2,}")
    return pattern.sub(r"\1\1", text)
from autocorrect import Speller
def spell_checker(tex):
        spell = Speller(lang='en')
        return [spell(reduce_lengthening((x))) for x in tex.split()]
```

Anyway being the corpus in a highly codified format we expect a relatively small number of typos that would not influence the overall word counter in a relevant way.

The code below represents all the preprocessing steps:

```python
import nltk
from nltk.corpus import wordnet
from os.path import expanduser
from nltk.parse import CoreNLPParser 
pos_tagger = CoreNLPParser(url='http://localhost:9000', tagtype='pos')

#Converter from Stanfort CoreNLP to wordnet format
def wordnet_pos_gen(lista): 
    pos_tags = list(pos_tagger.tag(lista))
    tag_dict = {"J": wordnet.ADJ, "N": wordnet.NOUN, "V": wordnet.VERB, "R": wordnet.ADV}
    t = []
    for x in pos_tags:
        try:
            t.append((x[0], tag_dict[x[1][0]]))
        except Exception:
            t.append(
                (x[0], "n"))  # a get around to assign irrelevant tags to one of wordnet classes, noun in this case
    return t

# Stopping
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
def stopping(ls):
    processed = []
    for case in ls:
        tok = word_tokenize(case)
        stop_words = set(stopwords.words("english"))
        stop_words.add(","), stop_words.add("."), stop_words.add(";"), stop_words.add(":")
        res = [x for x in tok if not x in stop_words]
        result = ""
        for x in res:
            result = result + " " + x
        processed.append(result)
    return processed

# Lemmatizer
from nltk.stem import WordNetLemmatizer
def lemma(lista):
    out = []
    t = wordnet_pos_gen(lista)
    lemmatizer = WordNetLemmatizer()
    res = [lemmatizer.lemmatize(w[0], w[1]) for w in t]
    result = ""
    for x in res:
        result = result + " " + x
    out.append(result.lower())  # case normalization step here
    return out

# Spelling Normalization
import re
from autocorrect import Speller
def reduce_lengthening(text):
    pattern = re.compile(r"(.)\1{2,}")
    return pattern.sub(r"\1\1", text)

def spell_checker(lista):
    spell = Speller(lang='en')
    r = []
    for x in lista:
        r.append((spell(reduce_lengthening(x))))
    return r
```

To set up the Pos tagger we executed this in the Linux Terminal:

```shell
$ cd stanford-corenlp-full-2018-02-27
$ java -mx4g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLPServer -preload tokenize,ssplit,pos,lemma,ner,parse,depparse -status_port 9000 -port 9000 -timeout 30000 &
```

We used CoreNLPParser instead of the default nltk.pos_tag for its improved performances.

With the pickle library we saved the preprocessed cases into files

```python
import os
path = '/home/alessio/Desktop/aiii/crystal_ball_data/'
os.chdir("/home/alessio/Desktop/aiii/crystal_ball_data/")
articles = ['Article2', 'Article3', 'Article5', 'Article6', 'Article8', 'Article10', 'Article11', 'Article13', 'Article14']
article="Article14"
v = extract_parts(path + 'train/' + article + '/violation/*.txt', 'violation', part)
nv = extract_parts(path + 'train/' + article + '/non-violation/*.txt', 'non-violation', part)
trainset = v + nv
shuffle(trainset)
Xtrain = [i[0].lower() for i in trainset]
Ytrain = [i[1].lower() for i in trainset]

# test set with violations only
if article == 'Article14':
    test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'non-violation', part)
else:
    test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'violation', part)
Xtest_v = [i[0].lower() for i in test]
Ytest_v = [i[1].lower() for i in test]

print('Training on', Ytrain.count('violation'), '+', Ytrain.count('non-violation'), '=',
      Ytrain.count('violation') + Ytrain.count('non-violation'), 'cases',
      '\nCases available for testing(violation):', Ytest_v.count('violation'))

out = (stopping(Xtrain)) #first step is stopping
out1 = (stopping(Xtest_v))
temp = [] #this list will contain lemmatized cases
temp1 = []
error = [] #this list will contain cases that cannot be lemmatized 
error1 = []
i = 0
while i != len(out):
    try:
        temp.append((lemma([out[i]])))  #Spellchecker could be added here --> temp.append(spell_checker(lemma([out[i]]))) 
    except Exception as e:
        print(e)
        error.append(i)
    finally:
        i += 1
i = 0
while i != len(out1):
    try:
        temp1.append(
            (lemma([out1[i]])))
    except Exception as e:
        print(e)
        error1.append(i)
    finally:
        i += 1

qq = [] #this list will substitute Xtrain and Xtest_v
qq1 = []
for x in temp:
    result = ""
    for y in x:
        result = result + " " + y
    qq.append(result)
for x in temp1:
    result = ""
    for y in x:
        result = result + " " + y
    qq1.append(result)

for el in error:  # to remove labels from not included cases
    del Ytrain[el]
for el in error1:
    del Ytest_v[el]

Xtrain = qq
Xtest_v = qq1

pickle.dump(Ytest_v, open("Ytest14.p", "wb")) #this create files with preprocessed cases
pickle.dump(Ytrain, open("Ytrain14.p", "wb"))
pickle.dump(Xtrain, open("Xtrain14.p", "wb"))
pickle.dump(Xtest_v, open("Xtest14.p", "wb"))
```

Actually we were not able to implement spell checking in reasonable amount of time due to our machine limitations, but that function can be easily added in the lemmatization step.

An idea of the actual transformation is given by the following text taken from article 2:

> "According to the Government, the population of Akhkinchu-Barzoy was regularly warned by the military that mines had been laid by armed gangs in the forest. They said that the minefield around the military unit was marked"

> "accord government population akhkinchu-barzoy regularly warn military mine lay armed gang forest say minefield around military unit marked"

Run_pipeline function becomes:

```python
def run_pipeline(article, part, vec, c):    
    os.chdir("/home/alessio/Desktop/aiii/preprocessed")
    Xtrain = pickle.load(open(f"Xtrain{article[article.find('e')+1:]}.p", "rb")) #to open pickle files for each article
    Xtest_v = pickle.load(open(f"Xtest{article[article.find('e')+1:]}.p", "rb"))
    Ytrain = pickle.load(open(f"Ytrain{article[article.find('e')+1:]}.p", "rb"))
    Ytest_v = pickle.load(open(f"Ytest{article[article.find('e')+1:]}.p", "rb"))
    os.chdir("/home/alessio/Desktop/aiii/crystal_ball_data")

    train_model_test(Xtrain, Ytrain, Xtest_v, Ytest_v, vec, c, article)
```

Results from this approach are the following: 

+ Article 2: Accuracy = 0.7569
+ Article 3: Accuracy = 0.7166
+ Article 5: Accuracy = 0.7026
+ Article 6: Accuracy = 0.7023
+ Article 8: Accuracy = 0.6262
+ Article 10: Accuracy = 0.6480
+ Article 11: Accuracy = 0.7191
+ Article 13: Accuracy = 0.7033
+ Article 14: Accuracy = 0.7045
  
Accuracy now is approximately 69.77% thus it is increased for each article using the same set of parameters. Again we represented below the best predictors for Article 2.

![result](/images/nlp2.png )

Now before trying other models let's tune hyperparameters using **GridSearchCV**. We tried a bunch of values for the main hyperparameters, namely c, min_df, norm, ngram_range, binary.

```python
def grid_search(Xtrain, Ytrain, Xtest_v, Ytest_v, vec, c, article): 
    random.seed(225) #set a seed to get reproducible results from cv
    pipeline = Pipeline([
        ('tvec', TfidfVectorizer()),
        ('lsvc', LinearSVC(C=c))
    ])
    print(pipeline.get_params().keys()) #to get parameters list
    tf_params = {
    "tvec__binary":[True, False],
    "tvec__min_df": [1,2,3],
    "tvec__norm": ["l1", "l2"],
    "tvec__ngram_range": [(1,1),(2,2),(3,3),(4, 6)],
    "lsvc__C":[0.1,1,5]
    }
    tvc_gs = GridSearchCV(pipeline, param_grid=tf_params, cv=5, verbose=1, n_jobs=3)
    tvc_gs.fit(Xtrain, Ytrain)
    print(tvc_gs.best_params_)
    print(tvc_gs.best_score_)
    Ypredict = tvc_gs.predict(Xtest_v)
    evaluate(Ytest_v, Ypredict)
    return tvc_gs.best_params_
    #pickle.dump(tvc_gs.best_params_, open(f"bestpar_{article}.p", "wb")) #save selected values into a file
```

The choice of the actual values could be questioned and it is linked to our machine limitations. GridSearchCV yielded the following values:

+ Article 2: binary=False, ngram_range=(1, 1), norm='l2',c=5, min_df=2, Accuracy = 0.8151
+ Article 3: binary=True, ngram_range=(1, 1), norm='l2',c=1, min_df=2, Accuracy = 0.7922
+ Article 5: binary=True, ngram_range=(1, 1), norm='l2',c=1, min_df=2, Accuracy = 0.7547
+ Article 6: binary=True, ngram_range=(2, 2), norm='l2',c=1, min_df=3, Accuracy = 0.7369
+ Article 8: binary=True, ngram_range=(2, 2), norm='l2',c=1, min_df=1, Accuracy = 0.6888
+ Article 10: binary=False, ngram_range=(1, 1), norm='l2',c=1, min_df=1, Accuracy = 0.5680
+ Article 11: binary=False, ngram_range=(1, 1), norm='l2',c=5, min_df=1, Accuracy = 0.7415
+ Article 13: binary=True, ngram_range=(3,3), norm='l2',c=0.1, min_df=1, Accuracy = 0.8767
+ Article 14: binary=False, ngram_range=(1, 1), norm='l2',c=5, min_df=1, Accuracy = 0.7954

Overall accuracy is 75.21%. Variance is 0.0076.

These are the best predictiors for Article 2, in this case unigrams were chosen.

![result](/images/nlp3.png )

## Naive Bayes Approach

We used MultinomialNB from sklearn, tuning parameters with GridSearchCV once again.

+ Article 2: binary=True, ngram_range=(2, 2), norm='l2',alpha=0.1, min_df=1, Accuracy = 0.9367
+ Article 3: binary=False, ngram_range=(1, 1), norm='l2',alpha=0.1, min_df=1, Accuracy = 0.8382
+ Article 5: binary=True, ngram_range=(3, 3), norm='l2',alpha=0.1, min_df=1, Accuracy = 0.8535
+ Article 6: binary=True, ngram_range=(2, 2), norm='l2',alpha=0.1, min_df=1, Accuracy = 0.7452
+ Article 8: binary=False, ngram_range=(2, 2), norm='l2',alpha=0.1, min_df=2, Accuracy = 0.7070
+ Article 10: binary=False, ngram_range=(1, 1), norm='l2',alpha=1, min_df=2, Accuracy = 0.4000
+ Article 11: binary=False, ngram_range=(1, 1), norm='l2',alpha=1, min_df=1, Accuracy = 0.7865
+ Article 13: binary=False, ngram_range=(1,1), norm='l2',alpha=0.1, min_df=2, Accuracy = 0.8464
+ Article 14: binary=True, ngram_range=(2, 2), norm='l2',alpha=0.1, min_df=2, Accuracy = 0.7045

Overall accuracy is 75.75%. Variance is 0.0237.

## Word Embedding

We tried Doc2Vec approach that creates a numeric representation of text documents that can be used for classification tasks, in this case we used LinearSVC on them. For this we used the original dataset without applying any preprocessing transformation. We started with hyperparameters tuning using GridSearch. Given that Doc2Vec approach belongs to Gensim model and not to sklearn we could not include it into our Pipeline. Thus for tuning all parameters (vector_size and min_count from Doc2Vec, C from LinearSVC) together we recreated GridSearchCV approach manually.

```python
#Manual GridSearch
from sklearn.model_selection import KFold
random.seed(225)
ls=[]
for x in [100, 200, 300]: #vec_size
    for y in [1, 2, 3]: #min_count
        for z in [0.1,1,5]: #c
            model = Doc2Vec(vector_size=x, min_count=y ,epochs=5)
            model.build_vocab(training_set)
            model.train(training_set, total_examples=model.corpus_count, epochs=model.epochs)
            predictors_train = []
            for sentence in Xtrain:
                sentence = sentence.split()
                predictor = model.infer_vector(doc_words=sentence, epochs=5, alpha=0.01)
                predictors_train.append(predictor.tolist())
            predictors_test = []
            for sentence in Xtest_v:
                sentence = sentence.split()
                predictor = model.infer_vector(doc_words=sentence, epochs=5, alpha=0.01)
                predictors_test.append(predictor.tolist())

            sv_classifier = SVC(kernel='linear', class_weight='balanced', decision_function_shape='ovr',
                                random_state=0, C=z)

            predictors_train = np.array(predictors_train)
            Y_train = np.array(Ytrain)
            kf = KFold(n_splits=5) #to make 5-fold CV
            l=[]
            for train_index, test_index in kf.split(range(len(predictors_train))):
                X_train, X_test = predictors_train[train_index], predictors_train[test_index]
                y_train, y_test = Y_train[train_index], Y_train[test_index]
                sv_classifier.fit(X_train, y_train)
                Ypredict = sv_classifier.predict(X_test)
                l.append(accuracy_score(y_test, Ypredict))
            ls.append(([x,y,z],sum(l)/len(l)))

best_par=[x[0] for x in ls if x[1]==max([x[1] for x in ls])][0]
best_x,best_y,best_z=best_par[0],best_par[1],best_par[2]
```

Using Doc2Vec there is no possibility of getting the same result in a deterministic way (Q11 from gensim FAQ). We cannot set a seed to fix the way in which numeric vectors are made. Thus we can only approximate the optimal outcome of this GridSearch, relying on the fact that results obtained should not differ that much. We could repeat the process n times and take the average out of it but in this case due to our machine limitation we will stick to the single execution. Once we have found the approximation for the best hyperparameters we can train a LinearSVC on the model with these and here repeat the process e.g. 10 times taking the average of those results.

```python
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
from sklearn.svm import SVC

def run_pipeline(article, part, vec, c):  
    print('Trained on *' + part + '* part of the cases')
    v = extract_parts(path + 'train/' + article + '/violation/*.txt', 'violation', part)
    nv = extract_parts(path + 'train/' + article + '/non-violation/*.txt', 'non-violation', part)
    trainset = v + nv
    shuffle(trainset)
    Xtrain = [i[0] for i in trainset]
    Ytrain = [i[1] for i in trainset]
    # test set with violations only
    if article == 'Article14':
        test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'non-violation', part)
    else:
        test = extract_parts(path + 'test_violations/' + article + '/*.txt', 'violation', part)
    Xtest_v = [i[0] for i in test]
    Ytest_v = [i[1] for i in test]

    training_set = [TaggedDocument(sentence, tag) for sentence, tag in zip(Xtrain, Ytrain)]
    ls=[]
    for x in range(10):
      model = Doc2Vec(vector_size=best_x, min_count=best_y, epochs=5)
      model.build_vocab(training_set)
      model.train(training_set, total_examples=model.corpus_count, epochs=model.epochs)

      predictors_train = []
      for sentence in Xtrain:
          sentence = sentence.split()
          predictor = model.infer_vector(doc_words=sentence, epochs=5, alpha=0.01)
          predictors_train.append(predictor.tolist())

      predictors_test = []
      for sentence in Xtest_v:
          sentence = sentence.split()
          predictor = model.infer_vector(doc_words=sentence, epochs=5, alpha=0.01)
          predictors_test.append(predictor.tolist())

      sv_classifier = SVC(C=best_z, kernel='linear', class_weight='balanced', decision_function_shape='ovr', random_state=0)
      sv_classifier.fit(predictors_train, Ytrain)

      Ypredict = sv_classifier.predict(predictors_test)
      ls.append(accuracy_score(Ytest_v, Ypredict))
    print(sum(ls)/len(ls))
```

+ Article 2: vector_size=200, min_count=3, c=5, Accuracy is 0.7489
+ Article 3: vector_size=300, min_count=3, c=5, Accuracy is 0.7654
+ Article 5: vector_size=300, min_count=2, c=5, Accuracy is 0.8748
+ Article 6: vector_size=200, min_count=2, c=5, Accuracy is 0.8094
+ Article 8: vector_size=200, min_count=2, c=5, Accuracy is 0.7989
+ Article 10: vector_size=300, min_count=3, c=5, Accuracy is 0.7055
+ Article 11: vector_size=300, min_count=2, c=5, Accuracy is 0.7033
+ Article 13: vector_size=100, min_count=2, c=5, Accuracy is 0.8447
+ Article 14: vector_size=200, min_count=2, c=1, Accuracy is 0.4340
  
Average accuracy is 74.28%, variance is 0.0167.

## Conclusion

> We tried several approaches after having performed additional text preprocessing techniques (e.g. lemmatization) but with a looser tuning of hyperparameters (smaller grids due to our machine limitations). We managed to get similar results to those from Medvedeva et al. (they also got about 75% accuracy on average but tuning an additional hyperparamer, i.e. "parts" thus training on larger datasets, that we have not considered). We reached that level of accuracy also using the embedding approach, notwithstanding the limited number of available training cases for some articles. Further research could count on a broader number of both violation and non-violations cases that is now available and try even more instance-wise approaches as Convolutional Neural Networks.

[<center>↑ BACK TO TOP ↑</center>](#top)

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(document).ready(function() {
      $('a[href="#top"]').click(function() {
        $('html, body').animate({ scrollTop: 0 }, 'slow');
        return false;
      });
    });
  </script>
