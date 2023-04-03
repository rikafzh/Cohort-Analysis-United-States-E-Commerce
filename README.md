# Use United State E-Commerce Records 2020 data from Kaggle
# https://www.kaggle.com/datasets/ammaraahmad/us-ecommerce-record-2020

Load Data
# Import library
import pandas as pd

# Load data
data = pd.read_csv('us_2020.csv', engine='python', encoding='ISO-8859-1')

# View data
data.head()

Reformat Timestamp
import dateutil
from datetime import datetime as dt
from pytz import utc

# Add new column datetime, month, and year
data['datetime'] = data['Order Date'].apply(lambda x: dateutil.parser.parse(x).timestamp())
data['month'] = data['datetime'].apply(lambda x: dt.fromtimestamp(x, utc).month)
data['year'] = data['datetime'].apply(lambda x: dt.fromtimestamp(x, utc).year)

# View data
data.head()

Create Cohort
# Create format column first_cohort
data['cohort'] = data.apply(lambda row: (row['year'] * 100) + (row['month']), axis=1)

# Merge data
cohorts = data.groupby('Customer ID')['cohort'].min().reset_index()
cohorts.columns = ['Customer ID', 'first_cohort']

data = data.merge(cohorts, on='Customer ID', how='left')

# View data merge
cohorts.head()

# View data with column first_cohort
data.head()

Header for Every Cohort
# Create header for every cohort
headers = data['cohort'].value_counts().reset_index()
headers.columns = ['Cohorts', 'Count']
headers.head()
headers = headers.sort_values(['Cohorts'])['Cohorts'].to_list()

# View data headers
headers

Pivot Data by Cohort
# View data
data.head()

# Remove null data
data.dropna(inplace=True)

# Make new column 'cohort_distance'
data['cohort_distance'] = data.apply(lambda row: (headers.index(row['cohort']) - headers.index(row['first_cohort'])) if (row['first_cohort'] != 0 and row['cohort'] != 0) else np.nan, axis=1)

# View data
data.head()

# Make pivot cohort
cohort_pivot = pd.pivot_table(data,
                              index='first_cohort',
                              columns='cohort_distance',
                              values='Customer ID',
                              aggfunc=pd.Series.nunique)

# View pivot Cohort
cohort_pivot

# Divide each value by the first row
cohort_pivot = cohort_pivot.div(cohort_pivot[0],axis=0)

# View data
cohort_pivot

View Pivot using Heatmap
# Import library
import seaborn as sns
import matplotlib.pyplot as plt

# Create heatmap
fig, ax = plt.subplots(figsize=(12, 8))
sns.heatmap(cohort_pivot, annot=True, fmt='.0%', mask=cohort_pivot.isnull(), ax=ax, square=True, linewidths=.5, cmap=sns.cubehelix_palette(8))
plt.title('Customer Retention Rate in Percentage: Monthly Cohort 2020')
plt.xlabel('Cohort Index')
plt.ylabel('Cohort Month')

plt.show()
