# Part 3: Soccer Data

*Introductory - Intermediate level SQL*

---

## Setup

Download the [SQLite database](https://www.kaggle.com/hugomathien/soccer/download). *Note: You may be asked to log in, or "continue and download".* Unpack the ZIP file into your working directory (i.e., wherever you'd like to complete this challenge set). There should be a *database.sqlite* file.

As with Part II, you can check the schema:

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('database.sqlite')

query = "SELECT * FROM sqlite_master"

df_schema = pd.read_sql_query(query, conn)

df_schema.tbl_name.unique()
```

---

Please complete this exercise using sqlite3 (the soccer data, above) and your Jupyter notebook.

1. Which team scored the most points when playing at home?  
```python
def home_most(conn):
    cur = conn.cursor()
    
    query = """SELECT t.team_long_name, SUM(m.home_team_goal) AS total
               FROM Team t
               INNER JOIN Match m
               ON t.team_api_id = m.home_team_api_id
               GROUP BY m.home_team_api_id
               ORDER BY total DESC;"""
    cur.execute(query)
    home_most = cur.fetchall()
    
    return pd.DataFrame(home_most)

home_most(conn)
```
(Real Madrid CF scored the most points when playing at home, 505!)

2. Did this team also score the most points when playing away?  
```python
def away_most(conn):
    cur = conn.cursor()
    
    query = """SELECT t.team_long_name, SUM(m.away_team_goal) AS total
               FROM Team t
               INNER JOIN Match m
               ON t.team_api_id = m.away_team_api_id
               GROUP BY m.away_team_api_id
               ORDER BY total DESC;"""
    cur.execute(query)
    away_most = cur.fetchall()
    
    return pd.DataFrame(away_most)

away_most(conn)
```
(FC Barcelona scored the most points when playing away, 354!)

3. How many matches resulted in a tie?  
```python
def ties(conn):
    cur = conn.cursor()
    
    query = """SELECT COUNT(match_api_id)
               FROM Match 
               WHERE home_team_goal == away_team_goal;"""
    cur.execute(query)
    ties = cur.fetchall()
    
    return pd.DataFrame(ties)
    
ties(conn)
```
(6,596 matches resulted in a tie.)

4. How many players have Smith for their last name? How many have 'smith' anywhere in their name?
```python
def last_name_Smith(conn):
    cur = conn.cursor()
    
    query = """SELECT COUNT(*)
               FROM Player 
               WHERE player_name LIKE '% Smith';"""
    cur.execute(query)
    last_name_Smith = cur.fetchall()
    
    return pd.DataFrame(last_name_Smith)
    
last_name_Smith(conn)

def name_Smith(conn):
    cur = conn.cursor()
    
    query = """SELECT COUNT(*)
               FROM Player 
               WHERE LOWER(player_name) LIKE '%smith%';"""
    cur.execute(query)
    name_Smith = cur.fetchall()
    
    return pd.DataFrame(name_Smith)

name_Smith(conn)
```
(There are 15 players have last name Smith. 18 players have "smith" anywhere in heir name.)

5. What was the median tie score? Use the value determined in the previous question for the number of tie games. *Hint:* PostgreSQL does not have a median function. Instead, think about the steps required to calculate a median and use the [`WITH`](https://www.postgresql.org/docs/8.4/static/queries-with.html) command to store stepwise results as a table and then operate on these results. 
```python
def ties_median(conn):
    cur = conn.cursor()
    
    query = """WITH total AS(
                    SELECT COUNT(*) AS total
                    FROM Match 
                    WHERE home_team_goal == away_team_goal),
                    top AS (
                    SELECT home_team_goal
                    FROM Match
                    WHERE home_team_goal == away_team_goal
                    ORDER BY home_team_goal ASC
                    LIMIT(SELECT total/2 from total)),
                    bottom AS (
                    SELECT home_team_goal
                    FROM Match
                    WHERE home_team_goal == away_team_goal
                    ORDER BY home_team_goal DESC
                    LIMIT(SELECT total/2 from total))
                    
                SELECT (MAX(top.home_team_goal) + MIN(bottom.home_team_goal))/2
                FROM top, bottom;"""
    cur.execute(query)
    ties_median = cur.fetchall()
    
    return pd.DataFrame(ties_median)
    
ties_median(conn)    
```
(The median tie score was 1.)

6. What percentage of players prefer their left or right foot? *Hint:* Calculate either the right or left foot, whichever is easier based on how you setup the problem.
```python

def pct_left(conn):
    cur = conn.cursor()
    
    query = """WITH left AS(   
               SELECT COUNT(DISTINCT(player_api_id)) AS left
               FROM Player_Attributes
               WHERE preferred_foot == 'left')
               
               SELECT left.left * 100.0 /COUNT(DISTINCT(player_api_id))
               FROM left, Player_Attributes;"""
    cur.execute(query)
    pct_left = cur.fetchall()
    
    return pd.DataFrame(pct_left)
    
pct_left(conn)
```
(29.0% of players prefer their left foot.)
