## Machine Learning with Spark ##
Leveraging distributed nature of Spark to massive data machine learning problems.

### Spark's MLLib ###
1. Some algos supported by MLLib are:  
- Feature extraction - TF-IDF (Term Frequency / Inverse Document Frequency) --> for searches
- Basic statistics - chi-squared test, peasrson
- Linear regression, logistics regression
- Support Vector Machines
- Naive Bayes classifier
- Decision trees
- K-Means clustering --> for unsupervised learning
- Principal component analysis, singular value decomposition
- Alternating Least Squares (ALS) --> for recommendations

2. The newer MLLib uses dataframes and the old uses RDD which is already deprecated in Spark 3.

1. Some learnings from the supported algos, e.g. ALS, don't seemed to work as expected. So be aware of what the algo is doing on Spark to make an informed decision. Data quality and hyperparameters values are sensitive influence on the outcomes too (as with any ML problems).

1. MLLib's linear regression can take in multi-features (x-axis) to try to predict a value (y-axis); if you have a multi-dimensional data, use Spark Streaming SGD (Stochastic Gradient Descent) for that purpose instead.

1. Use a DecisonTreeRegressor instead of a LinearRegression if you need to handle input data with features of different scales; use the default hyperparameters when starting.

1. Use a `VectorAssembler` to process your input data columns / features data:
```
val assembler = new VectorAssembler().
      setInputCols(Array("features_1", "feature_2", "feature_3")).
      setOutputCol("features")

 // some raw data read in from file
val df = assembler.transform(dsRaw)
        .select("label","features")
```
---
