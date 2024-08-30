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
# Preprocessing
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
categories = [[ordering1...], [ordering2...]]
ordinal_transformer = Pipeline(steps=[
("imputer", SimpleImputer(strategy="most_frequent")),
("ordinal", OrdinalEncoder(categories=categories))
])

preprocessor = ColumnTransformer(
transformers = [
("num", numeric_transformer, numeric_features),
("cat", categorical_transformer, categorical_features),
("ord", ordinal_transformer, ordinal_features)
]
)
```

# Pipeline

```
pipeline = Pipeline(steps=[('preprocessor', preprocessor),
('classifier', RandomForestClassifier())])
```

# Splitting Data
`X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=12)`

# Grid Search via Cross Validation
```
parameters = { 'classifier__n_estimators': [100, 200], 'classifier__max_depth': [None, 10, 20], 'classifier__min_samples_split': [2, 5, 10] }

grid_search = GridSearchCV(pipeline, parameters, cv=5, scoring="recall")

grid_search.fit(X_train, y_train)

print("Best parameters found: ", grid_search.best_params_)
print("Average recall: ", grid_search.best_scores_.mean())
```

# Test
```
y_pred = grid_search.predict(X_test)

print("Test Set Recall: ", recall_score(y_test, y_pred)) print(classification_report(y_test, y_pred))
```

# Feature Importance
```
importances = grid_search.best_estimator_.feature_importances_

output_features = list(data.select_dtypes("number")) + list(
final_pipeline["preprocessing"]["cat"]["encoding"].get_feature_names_out(["cat"])

)
```