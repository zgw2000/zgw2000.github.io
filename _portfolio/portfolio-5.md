---
title: "Apply machine learning techniques to a finance problem."
excerpt: "This project involves the application of machine learning techniques to a finance problem. The goal is to determine the characteristics that have the most impact on outstanding bonds. By using Boosted Regression Techniques and Random Forests, we analyze firm and country characteristics to identify the factors influencing bond performance. Through data preprocessing, model training, and evaluation, we aim to provide valuable insights into predicting firms' outstanding bonds. The chosen model demonstrates superior performance and interpretability, making it a valuable tool for decision-making in the finance industry.<br/><img src='/images/RISC_evo1.drawio.png'>"
collection: portfolio
---
<a id="top"></a>

***Miro Confalone, Jacopo Giannetti, Federico Astolfi, Guowang Zeng***

The dataset represents a random subsample of firm-year observations and the number of bonds issued by these firms, based on Fama-French 48 industry classification, the datasets contain indicators based on World Bank databanks and other sources, the entries in the dataset relate to specific kinds of industries in specific countries, and there are data about law code type, how much tax evasion in a certain country, information about GDP per capita, inflation, risk indicators, and frequency of bankruptcy among many others.
<br/><br/>
We used different models for our analysis, the Random Forest model and Boosting techniques. When it comes to the random forest, we have to mention Bagging. Bagging can be simply understood as: replacement sampling, majority voting (classification), or simple average (regression), and the base learners of Bagging belong to a parallel generation, and there is no strong dependence. relationship. Random Forest is an extended variant of Bagging. It builds Bagging integration based on the decision tree as the base learner and further introduces random feature selection in the training process of the decision tree.
<br/><br/>
Random forest algorithm can solve two types of problems of classification and regression, and it performs well. Because it is integrated learning, the variance and deviation are relatively low, and the generalization performance is superior.
<br/><br/>
Random forest has good processing power for high-dimensional data sets. It can process thousands of input variables and determine the most important variables, so it is considered a good dimensionality reduction method. In addition, the model can output the importance of features, which is a very practical function.
Can deal with missing data.
<br/><br/>
Random forest has also some downsides, for instance, Random forest does not perform as well as it does in classification when solving regression problems because it does not give a continuous output. When performing regression, the random forest cannot make predictions that exceed the range of the training set data, which may lead to overfitting when modeling some data with specific noise and tends to ignore correlations between attributes.
<br/><br/>
The other model we selected is represented by Gradient boosting regressor, this model is a member of the Boosting family of integrated learning, which is an improvement on the boosting algorithm based on GBDT (Gradient Boosted Decision Trees). GBDT uses the negative gradient of the model on the data as an approximation of the residuals to fit the residuals. These models offer some advantages like predictive accuracy that cannot be beaten. More flexibility optimizing on different loss functions and provides several hyperparameter tuning options that make the function fit very flexible. No data pre-processing required - often works great with categorical and numerical values as is. But this technique can overemphasize outliers and cause overfitting. Must use cross-validation to neutralize.
<br/><br/>
Computationally expensive - GBMs often require many trees (>1000) which can be time and memory exhaustive. The high flexibility results in many parameters that interact and influence heavily the behavior of the approach (number of iterations, tree depth, regularization parameters, etc.). This requires a large grid search during tuning.
Less interpretable although this is easily addressed with various tools (variable importance, partial dependence plots, LIME, etc.).
So we moved in this direction:
<br/><br/>
As we will explain later, we have implemented an algorithm that uses two models, random forest classifier and random forest regressor, which can significantly reduce the MSE on the predictions. The idea here is that the models obviously cannot make an accurate prediction when the value of the outstanding bond is 0, since it will try to come up with a predicted number that usually is non-null. For this specific reason, we thought of firstly understanding if the outstanding bond was zero or no zero, and then, only on the predictions that produced a non zero value, also applying the regressor.
<br/><br/>
We so proceeded by training the first model, the classifier, on the whole dataset. For doing so, we changed the outstanding bond column so that it was composed by zeros if the value of the observations' outstanding bond was zero, and 1s if they were nonzero.
<br/><br/>
In this way, we can accurately predict whether the outstanding of a particular observation is zero or if it has any non-zero value. For training the second model, on the other hand, we dropped from the dataset all the observations with outstanding bonds equal to zero, and we generated a model capable of predicting the precise number of outstanding bonds. We then came up with two lists for each observation, the first which, through the presence of a zero or a one, explains whether that observation has a null or non-zero outstanding bond value and the second that instead tries to predict the precise number. To conclude the algorithm we simply multiply the two lists. Where the first model had predicted 0 the result will therefore remain zero while where it had predicted 1 the result will be that of the second model.
<br/><br/>
The main problems encountered in carrying out the task are mainly multicollinearity, the need to apply feature engineering to the dataset, and the high number of outstanding bonds which values ​​were null (0). For multicollinearity, we used VIF (Variance Inflation Factor), which is a measure of the amount of multicollinearity, which deals with finding which variables are excessively correlated with each other and produce misleading results. For example, both the "Country" and "region" variables were already explained by the "Developed country" variable and were therefore dropped from the set. For the feature engineering, it was the N / A values ​​and the fact that the "Industry" feature was categorical and in the form of a string. To solve these problems we first identified the columns with N / A values ​​and replaced them in some cases with the average of the other observations for continuous variables, and with zero for categorical variables. As for the Industry column, the One hot encoder algorithm was used to generate a dummy variable for each category of the feature by inserting these dummies inside the data frame instead of the original column. The last problem that is the most limiting for the performance of the models is the presence of too many null values ​​at the predictor that none of the three models was initially able to manage and predict. To overcome this problem we have created an algorithm that initially uses a classification model in the case of the random forest to identify non-null values ​​and later apply the regression model only to those values ​​through a process of multiplying the results.

```python
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv) import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split, cross_validate
from sklearn.ensemble import RandomForestRegressor
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, r2_score, mean_squared_error from sklearn.linear_model import LinearRegression
from sklearn.ensemble import GradientBoostingRegressor
from statsmodels.stats.outliers_influence import variance_inflation_factor from sklearn.model_selection import GridSearchCV

#UPLOAD THE DATASET UNDER THE BOND VARIABLE
bond= pd.read_csv("/Users/zengguowang/Desktop/Casestudy3data.csv")
bond.head()

#VERIFY IF SOME COLUMNS HAVE NA VALUES, AND PRINT WHICH COLUMNS HAVE IT AND HOW␣ ,→MANY
print("SUM OF NA VALUES FOR EACH COLUMN ##################################" +␣ ,→"\n")
for i in bond.columns.array: print(bond[i].isna().sum(), i)

#FIND THE MEAN OF NON CATEGORICAL FEATURES TAHT HAVE NA VALUES mean =␣
,→bond[["efficientsystem","riskexpropriation","antidirectors","lawenforcement","creditorrights "EconomicRisk","inflation","Taxevasion"]].mean()
mean

#FILL THE NA VALUES OF THE CATEGORICAL FEATURES WITH 0 bond.update(bond[["developedcountry","civillaw","marketbased"]].fillna(0)) #FILL THE NA VALUES OF NON CATEGORICAL FEATURES WITH THE MEAN
bond.
,→update(bond[["efficientsystem","riskexpropriation","antidirectors","lawenforcement","credito ␣
,→"bankruptcy","liquidation","EconomicRisk","inflation","Taxevasion"]]. ,→fillna(mean))

DROP THE FEATURES THET WE DON'T NEED
bond = bond.drop(columns=["year","idfirm"])
bond.head()

#CREATE 3 DATAFREME CONTAINING THE DUMMIES VARIABLE OF CATEGORICAL FETURES THAT␣ ,→HAVE MORE THAN 2 CATEGORY
dummies = pd.get_dummies(bond.industryff48)
dummies2 = pd.get_dummies(bond.country)
dummies3 = pd.get_dummies(bond.region)

# MULTICOLLINEAIRITY WITH VIF
print("\n"*5 + "############################## MULTICOLLINEAIRITY WITH VIF ") def calc_vif(X):
    vif = pd.DataFrame()
vif["VIF Factor"] = [variance_inflation_factor(X.values, i) for i in␣ ,→range(X.shape[1])]
vif["features"] = X.columns return vif
vif = bond.drop(columns=["bonds", "country", "region","industryff48"]) vif = calc_vif(vif)
for index, row in vif.iterrows():
    print(row["VIF Factor"], row["features"])

#WE REPLACE THE COLUMNS OF THIS CATEGORICAL FETURES WITH ONLY DUMMIES␣ ,→"INDUSTRY" OF THE 3 DATAFRAME THAT WE HAVE CREATED
bond = bond.drop(columns=["industryff48","country","region"]) bond = pd.merge(bond, dummies, left_index=True, right_index=True)
list = []
for index, row in bond.iterrows():
if row['bonds'] == 0: list.append(0)
else: list.append(1)

#CREATE THE DATASET WITH ALL THE PREDICTOR AND THE DATASET WITH THE COLUMN␣ ,→"BOND" THAT IS OUR PREDICTION
X = bond.drop(columns=["bonds"])
y = bond["bonds"]
y_2 = list
X.head()

#DIVIDE THE DATASET IN TEST AND TRAIN
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)

#MODIFY THE Y TO CREATE OUR CLASSIFICATION MODEL BETWEEN 0 AND 1
y_train_2 = []
y_test_2 = []
for x in range(len(y_train)):
if y_train.tolist()[x] == 0: y_train_2.append(0)
else: y_train_2.append(1)
for x in range(len(y_test)): if y_test.tolist()[x] == 0:
y_test_2.append(0) else:
        y_test_2.append(1)

#CREATE AND FIT THE RANDOM FOREST CLASSIFIER
rf= RandomForestClassifier()
rfModel = rf.fit(X_train,y_train_2)
y_pred_test = rf.predict(X_test)
print("\n"*3 + "############################") print("SCORE OF THE RANDOM FOREST CLASSIFIER: {}".
,→format(accuracy_score(y_test_2, y_pred_test))+ "\n") #THIS IS THE SCORE OF THE MODEL

#TO DO THE RANDOM FOREST REGRESSOR WE TRAIN OUR MODEL ON THE DATASET WITHOUT␣ ,→THE 0 IN THE BONDS COLUMN
bond_2 = bond[bond.bonds != 0]
x_3 = bond_2.drop(columns=["bonds"])
y_3 = bond_2["bonds"]

#CREATE AND FIT THE RANDOM FOREST REGRESSOR
X_train_3, X_test_3, y_train_3, y_test_3 = train_test_split(x_3, y_3,␣ ,→test_size=0.3)
rf_2= RandomForestRegressor()
rfModel_2 = rf_2.fit(X_train_3,y_train_3)
rf_score = rf_2.score(X_test_3, y_test_3)
print("SCORE OF THE RANDOM FOREST REGRESSOR: {}".format(rf_score)+ "\n")

#FEATURE IMPORTANCES (REGRESSOR)
importances2 = rf_2.feature_importances_
std2 = np.std([
tree.feature_importances_ for tree in rf_2.estimators_], axis=0)
(pd.Series(rf_2.feature_importances_, index=X.columns)
   .nlargest(10)
   .plot(kind='barh'))
plt.show() #SHOW FEATURE IMPORTANCE

y_pred_test_2 = rf_2.predict(X_test) print("################################"+ "\n"*2)
print("list of prediction whit the random forest regressor:") print(y_pred_test_2.tolist())
print("list of prediction whit the random forest classifier:") print(y_pred_test.tolist())

#print(r2_score(y_test.tolist(), final_pred))
print("\n"+ "MSE of the final prediction: ") print(mean_squared_error(y_test.tolist(), final_pred)) print("\n"*4)

#CROSS VALIDATION MODEL CV WITH RANDOM FOREST

# Create the parameter grid based on the results of random search
param_grid = {
'bootstrap': [True], 'max_depth': [80, 90], 'max_features': [2, 3], 'min_samples_leaf': [3, 4], 'min_samples_split': [8, 10], 'n_estimators': [100, 200]
}
# Create a based model
rf_cv = RandomForestRegressor()
# Instantiate the grid search model
grid_search = GridSearchCV(estimator = rf_cv, param_grid = param_grid,
                          cv = 10, n_jobs = -1, verbose = 2)
grid_search.fit(X,y)
print("\n"*2 + "############ SCORE OF CROSS VALIDATION ############" + "\n") print('Best Score for RF Regression: %s' % grid_search.best_score_) print('Best Hyperparameters for RF Regression: %s' % grid_search.best_params_)

#SHOW FEATURE IMPORTANCE OPTIMIZED RF
optimised_random_forest = grid_search.best_estimator_
importances_optimized_rf = optimised_random_forest.feature_importances_
std_1 = np.std([
tree.feature_importances_ for tree in optimised_random_forest.estimators_],␣ ,→axis=0)
(pd.Series(optimised_random_forest.feature_importances_, index=X.columns)
   .nlargest(10)
   .plot(kind='barh', title= "Optimized RF Features"))
plt.show()

#BOOSTING REGRESSION TECHNIQUE
params = {
          'max_depth': [5,6,7],
          'min_samples_split': [5],
          'learning_rate': [0.01],
          'loss': ['ls'],
          'n_estimators':[500]}
xgb= GradientBoostingRegressor()
grid_search_gb = GridSearchCV(xgb,
params,
cv = 10, n_jobs = 5, verbose=True)
grid_search_gb.fit(X,y)
print("\n" + "############ SCORE OF XGBoost ############" + "\n") print('Best Score for XGBoost: %s' % grid_search_gb.best_score_) print('Best Hyperparameters for XGBoost: %s' % grid_search_gb.best_params_) grid_search_gb_best_estimator = grid_search_gb.best_estimator_

#SHOW FEATURE IMPORTANCE BOOSTING REGRESSION TECHNIQUE
feature_importance = grid_search_gb_best_estimator.feature_importances_
sorted_idx = np.argsort(feature_importance)
pos = np.arange(sorted_idx.shape[0]) + .5
fig = plt.figure(figsize=(12, 6))
plt.subplot(1, 2, 1)
plt.barh(pos, feature_importance[sorted_idx], align='center')
plt.yticks(pos, np.array(X.columns)[sorted_idx])
plt.title('Feature Importance (MDI)')
plt.show()
```

## Results

### 123

#### 78

##### 34

###### 376




















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
