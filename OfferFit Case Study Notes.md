`data.head()` Inspect first few rows of data
`data.shape` Observations + feature count `data.shape`
`data.info()` Data types of columns and non-null counts
- Make notes of any columns with many missing values, consider dropping
`data.describe()` summary stats for numerical cols, is anything weird?
`data.nunique()` could tell if numerical cols are categorical, if categorical has too high # of categories
`data["col"].value_counts()` get counts of vals in spec column
`data.duplicated()` check if any duplicate rows

`sns.pairplot(data, hue)` numeric only works well if not too many features
`sns.heatmap(data, annot=True)`