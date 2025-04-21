# MASH-Peformance-Analytics
This project was for my Senior Project/Portfolio class. The project consisted of working with MASH Performance, a sports facility in Savage, MN. In the past 10 years, athletes's performance and strenth testing had been tracked to measure progreess in player development. Much of the data had not been cleaned and analyzed for overall trends in player development.

# Goals
- Consolidate Excel sheets into one dataset for more efficient analysis
- Create visualizations that provide clear insights into player progress
- Conduct advanced analyses on current testing metrics
- Learn new skills!

<details>
<summary><strong>📌 Table of Contents</strong></summary>

- [Part I: Clean Data, Create Unique PlayerIDs, and Merge Datasets](#part-i-clean-data-create-unique-playerids-and-merge-datasets)
- [Part II: Exploratory Data Analysis](#part-ii-exploratory-data-analysis)
  - [Descriptive Stats](#descriptive-stats)
  - [Distributions](#distributions)
- [Part III: Modeling and Evaluation](#part-iii-modeling-and-evaluation)
  - [Model Selection](#model-selection)
  - [Performance Metrics](#performance-metrics)

</details>


## Part I: Clean Data, Create Unique PlayerIDs, and Merge Datasets
- Initial data was housed in three main Excel sheets: player metadata, performance history, and strength history
- In order to keep player data anonymous, a unique player ID was generated in Excel for each player within the Player sheet
- Then the index/match function was used to match these player IDs to other player occurrences in both the performance and strenght history sheets based on player name
- Data was cleaned (trimming spaces, correcting spelling errors in names) until all player IDs were matched to the rows in the performance and strength history sheets.

## Part II: Analysis in Python Using Visual Studio Code
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
Plot this distribution using the seaborn package.
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
![newplot](https://github.com/user-attachments/assets/c6d28c93-56e1-4bde-88d4-d85e8deb7994)

The plot shows that most missing data occurs within some of the testing types (eg., Food Speed Type, SL Broad Jump Type). The Change of Direction test, which is tied to columns titled 'Agility Time R' and 'Agility Time L', had the least amout of missing data, so I chose to work with it to conduct some deeper analysis.

## Bootstrapping: Understanding the Pro-Agility Sprint Test
How different are sprint time between athletes' left and right sides?
```
p_tests = performance_tests
COD_tests = p_tests[p_tests['COD'] == 'Pro-Agility Sprint']
len(COD_tests)
```
There were 1, 844 initial records of the Pro-Agility Sprint test.
```
#dropping NAs from the Change of Direction Tests
COD_tests.dropna(subset=['Agility Time L', 'Agility Time R'], inplace=True)
```
After dropping missing values, there were 1,758 complete records. 

```
COD_tests['CODTimeDifference'] = abs(COD_tests['Agility Time R'] - COD_tests['Agility Time L'])  # Absolute difference
COD_tests['CODPercentageDifference'] = (COD_tests['CODTimeDifference'] / COD_tests[['Agility Time R', 'Agility Time L']].mean(axis=1)) * 100
print(COD_tests['CODPercentageDifference'].describe())

```
```
count    1758.000000
mean        2.072706
std         3.139420
min         0.000000
25%         0.633580
50%         1.497727
75%         2.577882
max        83.889695
Name: CODPercentageDifference, dtype: float64
```

Results: On average, athletes tended to have a sprint time on their left side that was 2% greater than the spring on their right side. Using boostrapping, are the distributions of these sprint times between left and right sides similar, and if so, how significant is 2% difference in sprint times?

```
# Sample Data
np.random.seed(42)  # For reproducibility

# Bootstrapping Function
def bootstrap_ci(COD_tests, num_bootstrap=10000, ci=95):
    means = np.random.choice(COD_tests, (num_bootstrap, len(COD_tests)), replace=True).mean(axis=1)
    lower = np.percentile(means, (100 - ci) / 2)
    upper = np.percentile(means, 100 - (100 - ci) / 2)
    return means, (lower, upper)

# Perform Bootstrapping
left_means, left_ci = bootstrap_ci(COD_tests['Agility Time L'].values)
right_means, right_ci = bootstrap_ci(COD_tests['Agility Time R'].values)

```
Plotting the distribution
```
plt.style.use('seaborn-whitegrid')
plt.rcParams['axes.facecolor'] = '#F5F5F5'
plt.rcParams['font.family'] = 'Arial'

plt.figure(figsize=(8, 5))

# Histograms
plt.hist(left_means, bins=30, alpha=0.85, color='#2874AF', label='Left Side Mean Times')
plt.hist(right_means, bins=30, alpha=0.75, color='#6AB3E7', label='Right Side Mean Times')

# axvlines (Mean indicators)
plt.axvline(COD_tests['Agility Time L'].mean(), color='#B21029', linestyle='dashed', linewidth=2.5)
plt.axvline(COD_tests['Agility Time R'].mean(), color='#B21029', linestyle='dashed', linewidth=2.5)

# Labels & Styling
plt.legend()
plt.title("Bootstrapped Mean Distributions of Pro-Agility Sprint", fontsize=14)
plt.xlabel("Mean Time (s)")
plt.ylabel("Frequency")
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/3e787ac9-d79a-43f5-8da9-8dbc76e6b903)

Distributions appear normal and are similiar. To further ensure that the bootstapped distributions were similar and validate the two simulated distributions against the real data, I ran a Kolmogorov-Smirnov, or a KS, test.
```
# Perform KS Test
ks_stat, p_value = ks_2samp(COD_tests['Agility Time L'].values, COD_tests['Agility Time R'].values)

# Print Results
print(f"KS Statistic: {ks_stat}")
print(f"P-value: {p_value}")
```
KS Statistic: 0.02
P-value: 0.78
The KS statistic (p = .78) suggests that there is not a significant difference between the distributions.

To analyze how different the average times were between the left and right side, I ran a paired-samples t-test between the left and right side times.
```
# Sample Data
np.random.seed(42)  # For reproducibility

# Paired t-test
def paired_t_test(left_data, right_data):
    # Calculate the differences
    differences = left_data - right_data
    # Perform the t-test
    t_stat, p_value = stats.ttest_1samp(differences, 0)
    return t_stat, p_value

# Perform the paired t-test
t_stat, p_value = paired_t_test(COD_tests['Agility Time L'].values, COD_tests['Agility Time R'].values)

# Output the results
print(f"T-statistic: {t_stat}")
print(f"P-value: {p_value}")
```
T-statistic: 2.44
P-value: 0.014
The average sprint time on the left side (M = 4.82, SD = 0.26) was slightly longer than the right side (M = 4.70, SD = 0.25). This difference in sprint times was statistically significant, suggesting that players tend to be faster in changing direction on the right side compared to their left. 

## Part IV: Visualization
### PowerBI Dashboard
I created a PowerBI dashboard with a few basic visualizations in the PowerBI browser version. In contrast to the desktop version, PowerQuery and other functioinalities are quite limited. This led to a few challenges, such as struggling to change data types after loading data in. For example, many columns were loaded in as strings rather than integers, which meant that the only aggregate function that could be applied to much of the data was the count function, whereas if columns were integers or decimals data types, aggregate functions such as 'sum' and 'average' could have ben used to create more complex visualizations.
[MashReport_PowerBI.pdf](https://github.com/user-attachments/files/19837291/MashReport_PowerBI.pdf)

### Tableau Dashboard
I had not used Tableau before this class, but I used this project as an opportunity to learn how to make a simple dashboard in Tableau, gaining a better understanding in the similarities and differences between PowerBI/Tableau.
[Mash_Tableau_Dashboard.pdf](https://github.com/user-attachments/files/19837303/Mash_Tableau_Dashboard.pdf)
