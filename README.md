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

4. How many records exist for performance tests compared to strength tests?

```
## Member Test Type
performance_tests = df[df['MemberTestType'] == 'Performance']
print(len(performance_tests))

strength_tests = df[df['MemberTestType'] == 'Strength']
len(strength_tests)
```
Of the reecords, 2,197 were performance tests while 533 were strength tests. Due to the larger number of performance test records, I decided to focus analysis on performance tests.

5. Within each test group type, multiple tests are chosen from depending on athlete health. For example, a change of direction test is administered to most athletes, but the specific drill to test change of direction can change depending on their health/sport: some may do a pro-agility sprint while others perform another test. After talking with the Director of Sports Performance, not all athletes undergo all test types on testing day, so I created a loop to help me understand which tests seem to be administered the most frequently and would have the smallest amount of missing data to work with.
```
# Loop through each column and get value counts
for column in performance_tests.columns:
    print(f"Value counts for column '{column}':")
    print(performance_tests[column].value_counts())  # Get value counts for each column
    print("\n")   Add a newline for readability 
```
Notes from this loop:
* 2198 records of performance testing
* 3 biggest categories of athletes: baseball (1292), multisport (377), softball (155)
* Male (1810) vs Female (337)
* Most common testing months: March, August, November, February
* Foot Speed Type: SL Dynamic (973 records)
* COD: Pro-Agility Sprint (1845 records)
* Vertical Jump Type: CMJ (2097 records)
* Horizontal Jump Type: Broad Jump (1958 records)
* SL Broad Jump Type (91 records)
* Localized Endurance Type: Pushups (198 records)
* Conditioning Test: 1 Min Row Machine (203 records)
* Custom Test Type: Vert M/S (651 records)

6. To further understand the amout of missing data, I used Plotly to create interactive visualizations that shows the amount of data present for each column, allowing a user to hover over each bar (representing a column) to see the amount of records.
```
# Total rows
total_rows = len(p_tests)

# Count non-missing (data present) and percent present
data_present = p_tests.notnull().sum().reset_index()
data_present.columns = ['Column', 'Present Count']
data_present['Percent Present'] = (data_present['Present Count'] / total_rows) * 100

# Create bar chart
fig = px.bar(data_present, x='Column', y='Percent Present',
             title='Data Present by Column (% and Count)',
             color='Percent Present',
             color_continuous_scale='Blues',
             hover_data={
                 'Column': True,
                 'Percent Present': ':.2f',
                 'Present Count': True
             })

fig.update_layout(yaxis_title='% Present',
                  hoverlabel=dict(bgcolor="white", font_size=12))

fig.show()
```



7.  

3. 



- 
- 


## Step 3: Creating Dashboard in PowerBI


## Step 4: Creating Dashboard in Tableau
