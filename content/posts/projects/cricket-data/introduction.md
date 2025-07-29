---
title: "Discovering Cricket Data #1"
date: 2025-07-26T17:05:00Z
draft: false
tags: ["cricket", "python", "jupyter", "data"]
categories: ["Discovering Cricket Data"]
author: "Ben Bullock"
showToc: true
TocOpen: false
weight: 1
cover:
    image: "/images/cricket.jpg"
    alt: "T20 Cricketer"
---

This is my first foray into unpacking datasets using Python & Jupyter.

I am a fan of many sports, and a keen reader of cricket & football analytics writers.

I'll walk you through this introductory learning session where I unpack some T20 cricket data from cricsheet.

---

## Prerequisites

First of all, I'll setup my working environment & download some csv's from cricsheet:

```
mkdir -p cricket-analytics/{etl,model,newsletter,data_dictionary}
cd cricket-analytics

curl -O https://cricsheet.org/downloads/t20s_csv2.zip
unzip t20s_csv2.zip
```
Next, I'll open up some sample CSV's in my IDE to explore the data and make some notes on the data structures and overall data quality.

First observations are that the info files seem to contain top level match information: the umpire, the teams, the date, the venue etc.

The base file contains a far more in-depth outlook of a given match:

```
match_id,season,start_date,venue,innings,ball,batting_team,bowling_team,striker,non_striker,bowler,runs_off_bat,extras,wides,noballs,byes,legbyes,penalty,wicket_type,player_dismissed,other_wicket_type,other_player_dismissed
```

```
211028,2005,2005-06-13,The Rose Bowl,1,0.1,England,Australia,ME Trescothick,GO Jones,B Lee,0,0,,,,,,,,,

```

IE

| match\_id | season | start\_date | venue         | innings | ball | batting\_team | bowling\_team | striker        | non\_striker | bowler | runs\_off\_bat | extras | wides | noballs | byes | legbyes | penalty | wicket\_type | player\_dismissed | other\_wicket\_type | other\_player\_dismissed |
| --------: | -----: | ----------- | ------------- | ------: | ---: | ------------- | ------------- | -------------- | ------------ | ------ | -------------: | -----: | ----: | ------: | ---: | ------: | ------: | ------------ | ----------------- | ------------------- | ------------------------ |
|    211028 |   2005 | 2005-06-13  | The Rose Bowl |       1 |  0.1 | England       | Australia     | ME Trescothick | GO Jones     | B Lee  |              0 |      0 |       |         |      |         |         |              |                   |                     |                          |

Now I have a rough idea of what I'll be working with, I'll get set up in Jupyter:

```
cd etl && go mod init cricket-analytics/etl
cd ../model && python -m venv venv && source venv/bin/activate
pip install pandas pyarrow duckdb jupyterlab

jupyter lab
```

Next, I'll create a new notebook and start running some ### Cells.

---

## Exploring the datasets

### Cell #1

The purpose here is to unpack the CSV files that we have and log some data.

```
# Let's start fresh and see what we're working with
import pandas as pd
import os

# Validate if we have the CSV files
csv_files = [f for f in os.listdir('../') if f.endswith('.csv')]
print(f"Found {len(csv_files)} CSV files")

if len(csv_files) > 0:
    # Load the first file without any assumptions
    df = pd.read_csv(f'../{csv_files[0]}', header=None)  # No header assumption
    
    print(f"\nFile: {csv_files[0]}")
    print(f"Shape: {df.shape}")
    print("\nFirst 20 rows, first 5 columns:")
    print(df.iloc[:20, :5])
    
    print("\nFirst column values (first 30 rows):")
    print(df.iloc[:30, 0].tolist())
else:
    print("No CSV files found! Check if unzip worked correctly.")
```

Output:

```
Found 8968 CSV files

File: 1327228.csv
Shape: (242, 22)

First 20 rows, first 5 columns:
           0       1           2                          3        4
0   match_id  season  start_date                      venue  innings
1    1327228    2022  2022-09-20  National Stadium, Karachi        1
2    1327228    2022  2022-09-20  National Stadium, Karachi        1
3    1327228    2022  2022-09-20  National Stadium, Karachi        1
4    1327228    2022  2022-09-20  National Stadium, Karachi        1
5    1327228    2022  2022-09-20  National Stadium, Karachi        1
6    1327228    2022  2022-09-20  National Stadium, Karachi        1
7    1327228    2022  2022-09-20  National Stadium, Karachi        1
8    1327228    2022  2022-09-20  National Stadium, Karachi        1
9    1327228    2022  2022-09-20  National Stadium, Karachi        1
10   1327228    2022  2022-09-20  National Stadium, Karachi        1
11   1327228    2022  2022-09-20  National Stadium, Karachi        1
12   1327228    2022  2022-09-20  National Stadium, Karachi        1
13   1327228    2022  2022-09-20  National Stadium, Karachi        1
14   1327228    2022  2022-09-20  National Stadium, Karachi        1
15   1327228    2022  2022-09-20  National Stadium, Karachi        1
16   1327228    2022  2022-09-20  National Stadium, Karachi        1
17   1327228    2022  2022-09-20  National Stadium, Karachi        1
18   1327228    2022  2022-09-20  National Stadium, Karachi        1
19   1327228    2022  2022-09-20  National Stadium, Karachi        1

First column values (first 30 rows):
['match_id', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228', '1327228']
```

### Cell #2:

For the second ### Cell, we want to log the data with headers:

```
df = pd.read_csv(f'../{csv_files[0]}')  # This will use first row as header

print("Columns found:")
print(df.columns.tolist())
print(f"\nShape: {df.shape}")
print("\nFirst 5 rows:")
df.head()
```

Output:

```
Columns found:
['match_id', 'season', 'start_date', 'venue', 'innings', 'ball', 'batting_team', 'bowling_team', 'striker', 'non_striker', 'bowler', 'runs_off_bat', 'extras', 'wides', 'noballs', 'byes', 'legbyes', 'penalty', 'wicket_type', 'player_dismissed', 'other_wicket_type', 'other_player_dismissed']

Shape: (241, 22)

First 5 rows:
match_id	season	start_date	venue	innings	ball	batting_team	bowling_team	striker	non_striker	...	extras	wides	noballs	byes	legbyes	penalty	wicket_type	player_dismissed	other_wicket_type	other_player_dismissed
0	1327228	2022	2022-09-20	National Stadium, Karachi	1	0.1	Pakistan	England	Mohammad Rizwan	Babar Azam	...	0	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN
1	1327228	2022	2022-09-20	National Stadium, Karachi	1	0.2	Pakistan	England	Babar Azam	Mohammad Rizwan	...	0	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN
2	1327228	2022	2022-09-20	National Stadium, Karachi	1	0.3	Pakistan	England	Babar Azam	Mohammad Rizwan	...	0	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN
3	1327228	2022	2022-09-20	National Stadium, Karachi	1	0.4	Pakistan	England	Mohammad Rizwan	Babar Azam	...	0	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN
4	1327228	2022	2022-09-20	National Stadium, Karachi	1	0.5	Pakistan	England	Babar Azam	Mohammad Rizwan	...	0	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN
5 rows × 22 columns
```

### Cell #3:

Now we've taken a look at a high-level, lets double-click on some data and see if we can pull some more interesting information:

```
# Count balls per innings
balls_per_innings = df.groupby('innings').size()
print("Balls per innings:")
print(balls_per_innings)
print(f"\nTotal balls in match: {len(df)}")

# Check batting team distribution
if 'batting_team' in df.columns:
    print("\nBalls per batting team:")
    print(df['batting_team'].value_counts())
```

Output:

```
Balls per innings:
innings
1    124
2    117
dtype: int64

Total balls in match: 241

Balls per batting team:
batting_team
Pakistan    124
England     117
Name: count, dtype: int64
```

### Cell #4:

Now, lets pull some datatypes and perform some validations for good measure:

```
# Data types and nulls
print("Data types:")
print(df.dtypes)

print("\nColumns with null values:")
null_counts = df.isnull().sum()
null_cols = null_counts[null_counts > 0]
if len(null_cols) > 0:
    print(null_cols)
else:
    print("No null values found!")

# Get a sense of the numeric columns
print("\nNumeric columns summary:")
df.describe()
```

Output:

```
Data types:
match_id                    int64
season                      int64
start_date                 object
venue                      object
innings                     int64
ball                      float64
batting_team               object
bowling_team               object
striker                    object
non_striker                object
bowler                     object
runs_off_bat                int64
extras                      int64
wides                     float64
noballs                   float64
byes                      float64
legbyes                   float64
penalty                   float64
wicket_type                object
player_dismissed           object
other_wicket_type         float64
other_player_dismissed    float64
dtype: object

Columns with null values:
wides                     236
noballs                   241
byes                      241
legbyes                   239
penalty                   241
wicket_type               230
player_dismissed          230
other_wicket_type         241
other_player_dismissed    241
dtype: int64

Numeric columns summary:
match_id	season	innings	ball	runs_off_bat	extras	wides	noballs	byes	legbyes	penalty	other_wicket_type	other_player_dismissed
count	241.0	241.0	241.000000	241.000000	241.000000	241.000000	5.000000	0.0	0.0	2.0	0.0	0.0	0.0
mean	1327228.0	2022.0	1.485477	9.721162	1.273859	0.045643	1.800000	NaN	NaN	1.0	NaN	NaN	NaN
std	0.0	0.0	0.500829	5.708693	1.449030	0.356475	1.788854	NaN	NaN	0.0	NaN	NaN	NaN
min	1327228.0	2022.0	1.000000	0.100000	0.000000	0.000000	1.000000	NaN	NaN	1.0	NaN	NaN	NaN
25%	1327228.0	2022.0	1.000000	4.600000	0.000000	0.000000	1.000000	NaN	NaN	1.0	NaN	NaN	NaN
50%	1327228.0	2022.0	1.000000	9.600000	1.000000	0.000000	1.000000	NaN	NaN	1.0	NaN	NaN	NaN
75%	1327228.0	2022.0	2.000000	14.600000	1.000000	0.000000	1.000000	NaN	NaN	1.0	NaN	NaN	NaN
max	1327228.0	2022.0	2.000000	19.600000	6.000000	5.000000	5.000000	NaN	NaN	1.0	NaN	NaN	NaN
```

### Cell #5

Lets explore some more columns:

```
# Runs distribution
if 'runs_off_bat' in df.columns:
    print("Runs per ball distribution:")
    print(df['runs_off_bat'].value_counts().sort_index())
    
# Wickets
if 'wicket_type' in df.columns:
    print("\nWicket types:")
    print(df['wicket_type'].value_counts())
    print(f"Total wickets: {df['wicket_type'].notna().sum()}")

# Bowlers and batters
if 'striker' in df.columns and 'bowler' in df.columns:
    print(f"\nUnique batters: {df['striker'].nunique()}")
    print(f"Unique bowlers: {df['bowler'].nunique()}")
```

Output:

```
Runs per ball distribution:
runs_off_bat
0     73
1    117
2     12
3      2
4     31
6      6
Name: count, dtype: int64

Wicket types:
wicket_type
caught               6
bowled               2
stumped              1
caught and bowled    1
lbw                  1
Name: count, dtype: int64
Total wickets: 11

Unique batters: 14
Unique bowlers: 11
```

### Cell #6:

Lets produce a metadata repository:

```
# Create a summary for your data dictionary
print("\n=== DATA DICTIONARY DRAFT ===")
print(f"File format: CSV2 (structured ball-by-ball data)")
print(f"Total columns: {len(df.columns)}")
print(f"Total balls: {len(df)}")

print(f"\nKey columns found:")
for col in df.columns:
    non_null = df[col].notna().sum()
    unique = df[col].nunique()
    dtype = df[col].dtype
    
    # Add example values for categorical columns
    if dtype == 'object' and unique < 10:
        examples = df[col].dropna().unique()[:3]
        print(f"- {col}: {dtype}, {non_null} non-null, {unique} unique values, e.g. {examples}")
    else:
        print(f"- {col}: {dtype}, {non_null} non-null, {unique} unique values")
```

Output:

```
=== DATA DICTIONARY DRAFT ===
File format: CSV2 (structured ball-by-ball data)
Total columns: 22
Total balls: 241

Key columns found:
- match_id: int64, 241 non-null, 1 unique values
- season: int64, 241 non-null, 1 unique values
- start_date: object, 241 non-null, 1 unique values, e.g. ['2022-09-20']
- venue: object, 241 non-null, 1 unique values, e.g. ['National Stadium, Karachi']
- innings: int64, 241 non-null, 2 unique values
- ball: float64, 241 non-null, 125 unique values
- batting_team: object, 241 non-null, 2 unique values, e.g. ['Pakistan' 'England']
- bowling_team: object, 241 non-null, 2 unique values, e.g. ['England' 'Pakistan']
- striker: object, 241 non-null, 14 unique values
- non_striker: object, 241 non-null, 15 unique values
- bowler: object, 241 non-null, 11 unique values
- runs_off_bat: int64, 241 non-null, 6 unique values
- extras: int64, 241 non-null, 3 unique values
- wides: float64, 5 non-null, 2 unique values
- noballs: float64, 0 non-null, 0 unique values
- byes: float64, 0 non-null, 0 unique values
- legbyes: float64, 2 non-null, 1 unique values
- penalty: float64, 0 non-null, 0 unique values
- wicket_type: object, 11 non-null, 5 unique values, e.g. ['bowled' 'caught' 'stumped']
- player_dismissed: object, 11 non-null, 11 unique values
- other_wicket_type: float64, 0 non-null, 0 unique values
- other_player_dismissed: float64, 0 non-null, 0 unique values
```

### Cell #7:

Finally, for a bit of fun, lets produce some match summary insights:

```
# Let's understand the match better
print("=== MATCH SUMMARY ===")
print(f"Match ID: {df['match_id'].iloc[0]}")
print(f"Date: {df['start_date'].iloc[0]}")
print(f"Venue: {df['venue'].iloc[0]}")
print(f"Teams: {df['batting_team'].unique()}")

# Innings breakdown
for inning in df['innings'].unique():
    inning_df = df[df['innings'] == inning]
    batting_team = inning_df['batting_team'].iloc[0]
    total_runs = inning_df['runs_off_bat'].sum() + inning_df['extras'].sum()
    wickets = inning_df['wicket_type'].notna().sum()
    balls = len(inning_df)
    overs = balls / 6
    
    print(f"\nInnings {inning}: {batting_team}")
    print(f"  Score: {total_runs}/{wickets} in {overs:.1f} overs")
    print(f"  Run rate: {total_runs/overs:.2f}")
    
# Top scorers
print("\n=== TOP SCORERS ===")
striker_runs = df.groupby('striker')['runs_off_bat'].sum().sort_values(ascending=False).head(5)
print(striker_runs)

# Top bowlers
print("\n=== WICKET TAKERS ===")
bowler_wickets = df[df['wicket_type'].notna()].groupby('bowler').size().sort_values(ascending=False)
print(bowler_wickets)
```

Output:

```
=== MATCH SUMMARY ===
Match ID: 1327228
Date: 2022-09-20
Venue: National Stadium, Karachi
Teams: ['Pakistan' 'England']

Innings 1: Pakistan
  Score: 158/7 in 20.7 overs
  Run rate: 7.65

Innings 2: England
  Score: 160/4 in 19.5 overs
  Run rate: 8.21

=== TOP SCORERS ===
striker
Mohammad Rizwan    68
AD Hales           53
HC Brook           42
Babar Azam         31
Iftikhar Ahmed     28
Name: runs_off_bat, dtype: int64

=== WICKET TAKERS ===
bowler
L Wood             3
AU Rashid          2
Usman Qadir        2
Haris Rauf         1
MM Ali             1
SM Curran          1
Shahnawaz Dhani    1
dtype: int64
```

---

## Key Takeaways

Through this exploration, I've discovered:
- Cricsheet's CSV2 format is remarkably clean - one row per ball with clear column names
- The data captures every detail: who bowled to whom, what happened, and the match context
- With 8,968 T20 matches available, there's rich potential for analysis
- Python/Pandas makes it surprisingly easy to extract match narratives from raw data

---

## What's Next?

Now that I understand the data structure, my next steps following the curriculum are:
- Set up AWS infrastructure for automated ingestion
- Build validation schemas to ensure data quality
- Start developing the Empirical Bayes Talent Index

This foundation will help me move from exploration to building a production analytics pipeline.