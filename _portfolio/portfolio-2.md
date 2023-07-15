---
title: "Judicial decision on ECtHR Dataset"
excerpt: "The goal of this project is building a system able to predict judicial decision with the intent of showing to what extent these are predictable. The dataset and the starting point are taken from Judicial decisions of the European Court of Human Rights: Looking into the crystal ball, a paper written by Medvedeva et al. The dataset contains all english cases available on 09/2017 from both the Chamber and the Grand Chamber. The task consists in classifying violation or non-violation for 9 different articles of ECHR. Their approach was basically to train 9 different binary classifiers using Support Vector Machines, one per article to assess if there was a violation or non-violation. The training set was built using the same number of random cases for the two classes. Many information were removed from the text of the articles, for example specific sections like the part containing the final decision that sometimes could be explicit or other informations not available until the trial ended. The test sets contain only one class. Notwithstanding this the task is still to build a model able to classify between the two classes rather than just identifying if it belong to that single class<br/><img src='/images/ecthr.png'>"
collection: portfolio
---
<a id="top"></a>

**Alessio Barboni, Miro Confalone, Guowang Zeng**

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

![result](/images/nlp.png )

Our goal now is to increase this accuracy.

## Text Preprocessing
-------







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
