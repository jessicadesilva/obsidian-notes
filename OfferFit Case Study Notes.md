# EDA notes
`data.head()` Inspect first few rows of data
`data.shape` Observations + feature count `data.shape`
`data.info()` Data types of columns and non-null counts
- Make notes of any columns with many missing values, consider dropping
`data.describe()` summary stats for numerical cols, is anything weird?
`data.nunique()` could tell if numerical cols are categorical, if categorical has too high # of categories
`data["col"].value_counts()` get counts of vals in spec column
`data.duplicated()` check if any duplicate rows

`sns.pairplot(data, hue)` numeric only works well if not too many features
`sns.heatmap(data.select_dtypes("number").corr(), annot=True)`
correlation only for numerics
`sns.countplot(data, x, hue)` countplot for categorical data
`sns.barplot(data, x, y, hue)` bar plots for comparing categorical data
`sns.boxplot(data, x, hue)` numeric
`sns.histplot(data, x, hue)` both numeric and cat

# Pipeline
```
numeric_features = [...]
numeric_transformer = Pipeline(steps=[
("imputer", SimpleImputer(strategy="median")),
("scaler", StandardScaler()),
])

categorical_features = [...]
categorical_transformer = Pipeline(steps=[
("imputer", SimpleImputer(strategy="most_frequent")),
("onehot", OneHotEncoder(handle_unknown="ignore"))
])

ordinal_features = [...]
ordinal_features = Pipeline(steps=[
("imputer", SimpleImputer(strategy="most_frequent")),
("label", LabelEncoder(handle_unknown="ignore"))
])
```