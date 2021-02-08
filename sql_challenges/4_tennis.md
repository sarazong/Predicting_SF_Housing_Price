j# Part 4: Tennis Data

*Intermediate - Advanced level SQL*

---

## Setup

We'll be using tennis data from [here](https://archive.ics.uci.edu/ml/datasets/Tennis+Major+Tournament+Match+Statistics).

Navigate to your preferred working directory, and download the data.

```bash
curl -L -o tennis.zip http://archive.ics.uci.edu/ml/machine-learning-databases/00300/Tennis-Major-Tournaments-Match-Statistics.zip
unzip tennis.zip -d tennis
```

Make sure you have Postgres installed and initialized

```bash
brew install postgresql
brew services start postgres
```

Install SQLAlchemy if you haven't already

```
conda install -c anaconda sqlalchemy
```

Start Postgres in your terminal with the command `psql`. Then, create a `tennis` database using the `CREATE DATABASE` command.

```
psql

<you_user_name>=# CREATE DATABASE TENNIS;
CREATE DATABASE
<you_user_name>=# \q
```

Pick a table from the *tennis* folder, and upload it to the database using SQLAlchemy and Pandas.

```python
from sqlalchemy import create_engine
import pandas as pd


engine = create_engine('postgresql://<your_user_name>:localhost@localhost:5432/tennis')

aus_men = pd.read_csv('./tennis/AusOpen-men-2013.csv')

# I'm choosing to name this table "aus_men"
aus_men.to_sql('aus_men', engine, index=False)
```

*Note: In the place of `<your_user_name>` you should have your computer user name ...*

Check that you can access the table

```python
query = 'SELECT * FROM aus_men;'
df = pd.read_sql(query, engine)

df.head()
```

Do the same for the other CSV files in the *tennis* directory.

---

## The challenges!

This challenge uses only SQL queries. Please submit answers in a markdown file.

1. Using the same tennis data, find the number of matches played by
   each player in each tournament. (Remember that a player can be
   present as both player1 or player2). 
```python
query = """CREATE VIEW us_women_total AS(
           SELECT player1."Player 1" AS player, 
                  (player1."num_game1" + player2."num_game2") AS total,
                  'US_women' AS tournament
           FROM(SELECT us_women."Player 1", COUNT(*) AS num_game1
                FROM us_women
                GROUP BY us_women."Player 1") AS player1, 
               (SELECT us_women."Player 2", COUNT(*) AS num_game2
                FROM us_women
                GROUP BY us_women."Player 2") AS player2
           WHERE player1."Player 1" = player2."Player 2");"""
```
Use the code above eight times (on the eight corresponding datasets) to create eight summary tables. Then combine the tables row-wise to get the master table with the following code:
```python
query = """SELECT * FROM aus_men_total
           UNION ALL
           SELECT * FROM french_men_total
           UNION ALL
           SELECT * FROM gb_men_total
           UNION ALL
           SELECT * FROM us_men_total
           UNION ALL
           SELECT * FROM aus_women_total
           UNION ALL
           SELECT * FROM french_women_total
           UNION ALL
           SELECT * FROM gb_women_total
           UNION ALL
           SELECT * FROM us_women_total;"""
```

2. Who has played the most matches total in all of US Open, AUST Open, 
   French Open? Answer this both for men and women.
```python
query = """SELECT master_men."player", SUM(master_men."total") AS grand_total
           FROM(SELECT * FROM aus_men_total
                UNION ALL
                SELECT * FROM french_men_total
                UNION ALL
                SELECT * FROM us_men_total) AS master_men
            GROUP BY master_men."player"
            ORDER BY grand_total DESC
            LIMIT(1);"""
```
(Stanislas Wawrinka has played 17 matches in all of US Open, Aus open, and French Open.)
```python
query = """SELECT master_women."player", SUM(master_women."total") AS grand_total
           FROM(SELECT * FROM aus_women_total
                UNION ALL
                SELECT * FROM french_women_total
                UNION ALL
                SELECT * FROM us_women_total) AS master_women
            GROUP BY master_women."player"
            ORDER BY grand_total DESC
            LIMIT(1);"""
```
(Dominika Cibulkova has played 7 matches in all of US Open, Aus open, and French Open.)

3. Who has the highest first serve percentage? (Just the maximum value
   in a single match.)
```python
query = """CREATE VIEW us_fsp AS(
           SELECT us_men."Player1" AS player, us_men."FSP.1" AS fsp FROM us_men
           UNION ALL
           SELECT us_men."Player2" AS player, us_men."FSP.2" AS fsp FROM us_men
           UNION ALL
           SELECT us_women."Player 1" AS player, us_women."FSP.1" AS fsp FROM us_women
           UNION ALL
           SELECT us_women."Player 2" AS player, us_women."FSP.2" AS fsp FROM us_women);"""
```
Use the code above four times (on the corresponding datasets for the four tournaments, AUS, French, Wimbledon, and US) to create summary table for first serve percentage. Then use the following code to combine summary tables to make the master summary table:
```python
query = """SELECT * FROM aus_fsp
           UNION ALL
           SELECT * FROM french_fsp
           UNION ALL
           SELECT * FROM gb_fsp
           UNION ALL
           SELECT * FROM us_fsp
           ORDER BY fsp DESC
           LIMIT(1);"""
```
(S Errani  has the highest first serve percentage at 93%.)

4. What are the unforced error percentages of the top three players
   with the most wins? (Unforced error percentage is % of points lost
   due to unforced errors. In a match, you have fields for number of
   points won by each player, and number of unforced errors for each
   field.)
```python
query = """CREATE VIEW us_pct_ufe AS(
           SELECT us_men."Player1" AS player, us_men."DBF.1" AS dbf, 
                  us_men."UFE.1" AS ufe, us_men."FNL1" AS games_won FROM us_men
           UNION ALL
           SELECT us_men."Player2" AS player, us_men."DBF.2" AS dbf,
                  us_men."UFE.2" AS ufe, us_men."FNL2" AS games_won FROM us_men
           UNION ALL
           SELECT us_women."Player 1" AS player, us_women."DBF.1" AS dbf, 
                  us_women."UFE.1" AS ufe, us_women."FNL.1" AS games_won FROM us_women
           UNION ALL
           SELECT us_women."Player 2" AS player, us_women."DBF.2" AS dbf,
                  us_women."UFE.2" AS ufe, us_women."FNL.2" AS games_won FROM us_women);"""
```
Use the code above four times (on the corresponding datasets for the four tournaments, AUS, French, Wimbledon, and US) to create summary table for unforced error. Then use the following code to combine summary tables to make the master summary table:
```python
query = """SELECT master."player", SUM(master."dbf") AS dbf_player, SUM(master."ufe") AS ufe_player, 
                  SUM(master."ufe")*100.0/(SUM(master."dbf") + SUM(master."ufe")) AS pct_ufe, 
                  SUM(games_won) AS num_win
           FROM(SELECT * FROM aus_pct_ufe
                UNION ALL
                SELECT * FROM french_pct_ufe
                UNION ALL
                SELECT * FROM gb_pct_ufe
                UNION ALL
                SELECT * FROM us_pct_ufe) AS master
            GROUP BY master."player"
            ORDER BY SUM(games_won) DESC
            LIMIT(5);"""
```
(The unfored error percentage for Rafael Nadal is 91.6%, for Novak Djokovic is 93.4%, and for Stanislas Wawrinka is 86.9%.) <br>

*Hint:* `SUM(double_faults)` sums the contents of an entire column. For each row, to add the field values from two columns, the syntax `SELECT name, double_faults + unforced_errors` can be used.


*Special bonus hint:* To be careful about handling possible ties, consider using [rank functions](http://www.sql-tutorial.ru/en/book_rank_dense_rank_functions.html).
