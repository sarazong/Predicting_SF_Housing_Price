# Part 2: Baseball Data

*Introductory - Intermediate level SQL*

---

## Setup

`cd` into the directory you'd like to use for this challenge. Then, download the Lahman SQL Lite dataset

```
curl -L -o lahman.sqlite https://github.com/WebucatorTraining/lahman-baseball-mysql/raw/master/lahmansbaseballdb.sqlite
```

*The `-L` follows redirects, and the `-o` uses the filename instead of outputting to the terminal.*

Make sure sqlite3 is installed

```
conda install -c anaconda sqlite
```

In your notebook, check out the schema

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('lahman.sqlite')

query = "SELECT * FROM sqlite_master;"

df_schema = pd.read_sql_query(query, conn)

df_schema.tbl_name.unique()
```

---

Please complete this exercise using SQL Lite (i.e., the Lahman baseball data, above) and your Jupyter notebook.

1. What was the total spent on salaries by each team, each year? 
```python
def team_salaries(conn):
    cur = conn.cursor()
    
    query = """SELECT teamID, team_ID, SUM(salary) 
               FROM salaries
               GROUP BY teamID, yearID;"""
    cur.execute(query)
    team_salaries = cur.fetchall()
    
    return pd.DataFrame(team_salaries)

team_salaries(conn)
```
2. What is the first and last year played for each player? *Hint:* Create a new table from 'Fielding.csv'. 
```python
def years_played(conn):
    cur = conn.cursor()
    
    query = """SELECT playerID, MIN(yearID) AS first_year, MAX(yearID) AS last_year
               FROM fielding
               GROUP BY playerID;"""
    cur.execute(query)
    years_played = cur.fetchall()
    
    return pd.DataFrame(years_played)
    
years_played(conn)
```
3. Who has played the most all star games?
```python
def allstar_games(conn):
    cur = conn.cursor()
    
    query = """SELECT playerID, COUNT(gameID)
               FROM allstarfull
               GROUP BY playerID
               ORDER BY COUNT(gameID) DESC;"""
    cur.execute(query)
    allstar_games = cur.fetchall()
    
    return pd.DataFrame(allstar_games)
    
allstar_games(conn)
```
(aaronha01 has played the most all star games, 24 of them.)

4. Which school has generated the most distinct players? *Hint:* Create new table from 'CollegePlaying.csv'.
```python
def school_players(conn):
    cur = conn.cursor()
    
    query = """SELECT schoolID, COUNT(DISTINCT(playerID)) AS player_num
               FROM collegeplaying
               GROUP BY schoolID
               ORDER BY player_num DESC;"""
    cur.execute(query)
    school_players = cur.fetchall()
    
    return pd.DataFrame(school_players)
    
school_players(conn)
```
(Texas has generated the most distinct players, 107 of them.)

5. Which players have the longest career? Assume that the `debut` and `finalGame` columns comprise the start and end, respectively, of a player's career. *Hint:* Create a new table from 'Master.csv'. Also note that strings can be converted to dates using the [`DATE`](https://wiki.postgresql.org/wiki/Working_with_Dates_and_Times_in_PostgreSQL#WORKING_with_DATETIME.2C_DATE.2C_and_INTERVAL_VALUES) function and can then be subtracted from each other yielding their difference in days.
```python
def long_career(conn):
    cur = conn.cursor()
    
    query = """SELECT playerID, nameFirst, nameLast, debut, finalGame, 
                      (DATE(finalGame) - DATE(debut)) as duration
               FROM people
               ORDER BY duration DESC;"""
    cur.execute(query)
    long_career = cur.fetchall()
    
    return pd.DataFrame(long_career)
    
long_career(conn)
```
(Nick Altrock has the longest career at 35 years.)

6. What is the distribution of debut months? *Hint:* Look at the `DATE` and [`EXTRACT`](https://www.postgresql.org/docs/current/static/functions-datetime.html#FUNCTIONS-DATETIME-EXTRACT) functions.
```python
def debut_month(conn):
    cur = conn.cursor()
    
    query = """SELECT COUNT(debut), STRFTIME('%m', debut) AS month
               FROM people
               GROUP BY month;"""
    cur.execute(query)
    debut_month = cur.fetchall()
    
    return pd.DataFrame(debut_month)
    
debut_month(conn)
```

7. What is the effect of table join order on mean salary for the players listed in the main (master) table? *Hint:* Perform two different queries, one that joins on playerID in the salary table and other that joins on the same column in the master table. You will have to use left joins for each since right joins are not currently supported with SQLalchemy.
```python
def mean_salary(conn):
    cur = conn.cursor()
    
    query = """SELECT p.nameFirst, p.nameLast, AVG(s.salary)
               FROM people p
               LEFT JOIN salaries s
               ON p.playerID = s.playerID
               GROUP BY p.playerID;"""
    cur.execute(query)
    mean_salary = cur.fetchall()
    
    return pd.DataFrame(mean_salary)
    
mean_salary(conn)
```
