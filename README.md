# MASH-Peformance-Analytics
This project was for my Senior Project/Portfolio class. The project consisted of working with MASH Performance, a sports facility in Savage, MN. In the past 10 years, athletes's performance and strenth testing had been tracked to measure progreess in player development. Much of the data had not been cleaned and analyzed for overall trends in player development.

# Goals
- Consolidate Excel sheets into one dataset for more efficient analysis
- Create visualizations that provide clear insights into player progress
- Conduct advanced analyses on current testing metrics
- Learn new skills!

## Step 1: Clean Data, Create Unique PlayerIDs, and Merge Datasets
- Initial data was housed in three main Excel sheets: player metadata, performance history, and strength history
- In order to keep player data anonymous, a unique player ID was generated in Excel for each player within the Player sheet
- Then the index/match function was used to match these player IDs to other player occurrences in both the performance and strenght history sheets based on player name
- Data was cleaned (trimming spaces, correcting spelling errors in names) until all player IDs were matched to the rows in the performance and strength history sheets.

## Step 2: Analysis in Python Using Visual Studio Code
1. Import Packages.
```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
from scipy import stats
from scipy.stats import ks_2samp
import plotly.express as px
import plotly.io as pio
pio.renderers.default = "browser"  
import seaborn as sns
```
2. Read in Data from CSV.

### Basic Analysis
1. How many rows are in the dataset? How many unique members make up these rows?
```
num_rows = len(df) #number of rows in dataset
print(num_rows)
num_unique_members = df['MemberID'].nunique() #number of unique members
print(num_unique_members)
```
There were 2,730 records of athletic tests within the dataset, which was a combination of both performance tests (speed and agility) as well as strength tests, and 801 unique member IDs.

2. MASH has been open for a little over 10 years. How has the amount of recorded tests changed over time?
```
# Convert 'Date' column to datetime
df['Date'] = pd.to_datetime(df['Date'], errors='coerce')

# Extract year
df['Year'] = df['Date'].dt.year

# Get value counts per year
year_counts = df['Year'].value_counts().sort_index()

# Plot bar chart
plt.figure(figsize=(8, 5))
plt.bar(year_counts.index, year_counts.values, color='#001D54')

# Labels and title
plt.xlabel('Year')
plt.ylabel('Count')
plt.title('Tests per Year')
plt.xticks(year_counts.index)  # Ensure all years are shown on the x-axis

# Show the graph
plt.show()
```
![image](https://github.com/user-attachments/assets/da6763af-c304-46c1-9c18-3fb0822a3186)

The graph shows an increase in recorded tests for athletes, with the highest number of recorded tests occurring in the most recent year of testing 2024. This upward trend underscores the importance of analysis for player performance as the program grows.

3. How many tests does each athlete have?
```
# Assuming 'member_id' is the column identifying athletes in your dataset
athlete_test_counts = df['MemberID'].value_counts().reset_index()

# Rename columns for clarity
athlete_test_counts.columns = ['Member ID', 'Test Count']

# Group athletes by how many tests they had (1, 2, 3, etc.)
test_distribution = athlete_test_counts['Test Count'].value_counts().sort_index().reset_index()

# Rename columns to 'Number of Tests' and 'Number of Athletes'
test_distribution.columns = ['Number of Tests', 'Number of Athletes']
```
Plot this distribution using seaborn.
```
# Assuming 'member_id' is the column identifying athletes in your dataset
athlete_test_counts = df['MemberID'].value_counts()

# Create a DataFrame from the counts (no need to reset index)
test_distribution = athlete_test_counts.value_counts().sort_index()

# Plot the distribution using Seaborn's barplot
plt.figure(figsize=(10, 6))
sns.barplot(x=test_distribution.index, y=test_distribution.values, color = 'steelblue')

# Customize the plot
plt.title('Distribution of Tests per Athlete')
plt.xlabel('Number of Tests')
plt.ylabel('Number of Athletes')
plt.xticks(rotation=0)
plt.grid(True, axis='y', linestyle='--', alpha=0.7)

# Show the plot
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/42270958-027c-45d0-a2b8-b2cd67131e09)

The plot shows that athletes are most likely to have one test record. Less than 50 athletes have 6 testing records in their player history, and the number of athletes with more than 6 testing records declines as the number of records increases. There are many reasons why this trend may be happening, but two potential options could be that there are many new athletes becoming members at the sports facility who have just started training. Additionally, based on the Tests Per Year graph, tests seem to be have been recorded more often in recent years, so early data may not be recorded for some athletes.

## Step 3: Creating Dashboard in PowerBI


## Step 4: Creating Dashboard in Tableau
