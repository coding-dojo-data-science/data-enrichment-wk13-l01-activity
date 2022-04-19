# Chinook Analysis

## ERD

![png](Data/ERD.png)

## Connect to MySQL


```python
import pandas as pd
from sqlalchemy import create_engine
import pymysql
pymysql.install_as_MySQLdb()

## Change username and password to match your personal MySQL Server settings
username = 'root' # default username for MySQL db is root
password = 'YOUR_PASSWORD' # whatever password you chose during MySQL installation.

connection = f'mysql+pymysql://{username}:{password}@localhost/Chinook'
engine = create_engine(connection)
```


```python
q = '''SHOW TABLES;'''
pd.read_sql(q,engine)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Tables_in_chinook</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Album</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Artist</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Customer</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Employee</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Genre</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Invoice</td>
    </tr>
    <tr>
      <th>6</th>
      <td>InvoiceLine</td>
    </tr>
    <tr>
      <th>7</th>
      <td>MediaType</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Playlist</td>
    </tr>
    <tr>
      <th>9</th>
      <td>PlaylistTrack</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Track</td>
    </tr>
  </tbody>
</table>
</div>



## Queries

1. What is the most a customer has spent on a single purchase and what is their CustomerId?
2. What country is the customer with the largest total bill from (from question 2)?
3. Which customerID made the fewest purchases (invoices)?
4. Retrieve the id, name, and # of playlists for tracks that appear on playlists more than 4 times?


### 1. What is the most a customer has spent on a single purchase and what is their customerId?


```python
## Solution 1 - vers A
q = """SELECT 
    customerId,Total
    FROM Invoice
    ORDER BY Total DESC
    LIMIT 1
"""
pd.read_sql(q,engine)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerId</th>
      <th>Total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6</td>
      <td>25.86</td>
    </tr>
  </tbody>
</table>
</div>



### 2. What country is the customer with the largest total bill from (from question 2)?


```python
q = """SELECT 
    i.CustomerId, i.Total, c.Country
    FROM Invoice i
    JOIN Customer c ON c.CustomerId = i.CustomerId
    ORDER BY Total DESC
    LIMIT 1
"""
pd.read_sql(q,engine)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CustomerId</th>
      <th>Total</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6</td>
      <td>25.86</td>
      <td>Czech Republic</td>
    </tr>
  </tbody>
</table>
</div>



### 3. Which customerID made the fewest purchases (invoices)?


```python
q = """SELECT 
    customerId, COUNT(*) AS num_purchases
FROM
    Invoice
GROUP BY customerId
ORDER BY num_purchases ASC
LIMIT 1
"""
pd.read_sql(q,engine)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerId</th>
      <th>num_purchases</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>59</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



### 4. Retrieve the id, name, and # of playlists for tracks that appear on playlists more than 4 times?


```python
q = """SELECT 
    pt.trackId, t.Name, COUNT(*)
FROM
    Playlist p
        JOIN
    PlaylistTrack pt ON p.PlaylistId = pt.PlaylistID
        JOIN
    Track t ON pt.TrackId = t.TrackId
GROUP BY pt.TrackId
HAVING COUNT(*) > 4
ORDER BY COUNT(*) DESC"""
pd.read_sql(q,engine)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trackId</th>
      <th>Name</th>
      <th>COUNT(*)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3403</td>
      <td>Intoitus: Adorate Deum</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3404</td>
      <td>Miserere mei, Deus</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3408</td>
      <td>Aria Mit 30 Veränderungen, BWV 988 "Goldberg V...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3409</td>
      <td>Suite for Solo Cello No. 1 in G Major, BWV 100...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3410</td>
      <td>The Messiah: Behold, I Tell You a Mystery... T...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3411</td>
      <td>Solomon HWV 67: The Arrival of the Queen of Sheba</td>
      <td>5</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3415</td>
      <td>Symphony No.5 in C Minor: I. Allegro con brio</td>
      <td>5</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3416</td>
      <td>Ave Maria</td>
      <td>5</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3417</td>
      <td>Nabucco: Chorus, "Va, Pensiero, Sull'ali Dorate"</td>
      <td>5</td>
    </tr>
    <tr>
      <th>9</th>
      <td>3418</td>
      <td>Die Walküre: The Ride of the Valkyries</td>
      <td>5</td>
    </tr>
    <tr>
      <th>10</th>
      <td>3421</td>
      <td>Nimrod (Adagio) from Variations On an Original...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>11</th>
      <td>3423</td>
      <td>Jupiter, the Bringer of Jollity</td>
      <td>5</td>
    </tr>
    <tr>
      <th>12</th>
      <td>3424</td>
      <td>Turandot, Act III, Nessun dorma!</td>
      <td>5</td>
    </tr>
    <tr>
      <th>13</th>
      <td>3426</td>
      <td>Carmina Burana: O Fortuna</td>
      <td>5</td>
    </tr>
    <tr>
      <th>14</th>
      <td>3430</td>
      <td>Toccata and Fugue in D Minor, BWV 565: I. Toccata</td>
      <td>5</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3432</td>
      <td>Scheherazade, Op. 35: I. The Sea and Sindbad's...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>16</th>
      <td>3434</td>
      <td>Concerto for Piano No. 2 in F Minor, Op. 21: I...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>17</th>
      <td>3435</td>
      <td>Cavalleria Rusticana  Act  Intermezzo Sinfonico</td>
      <td>5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>3436</td>
      <td>Karelia Suite, Op.11: 2. Ballade (Tempo Di Men...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>19</th>
      <td>3437</td>
      <td>Piano Sonata No. 14 in C Sharp Minor, Op. 27, ...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>20</th>
      <td>3438</td>
      <td>Fantasia On Greensleeves</td>
      <td>5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>3439</td>
      <td>Das Lied Von Der Erde, Von Der Jugend</td>
      <td>5</td>
    </tr>
    <tr>
      <th>22</th>
      <td>3442</td>
      <td>Wellington's Victory or the Battle Symphony, O...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>23</th>
      <td>3444</td>
      <td>Romeo et Juliette: No. 11 - Danse des Chevaliers</td>
      <td>5</td>
    </tr>
    <tr>
      <th>24</th>
      <td>3445</td>
      <td>On the Beautiful Blue Danube</td>
      <td>5</td>
    </tr>
    <tr>
      <th>25</th>
      <td>3446</td>
      <td>Symphonie Fantastique, Op. 14: V. Songe d'une ...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>26</th>
      <td>3447</td>
      <td>Carmen: Overture</td>
      <td>5</td>
    </tr>
    <tr>
      <th>27</th>
      <td>3448</td>
      <td>Lamentations of Jeremiah, First Set  Incipit L...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>28</th>
      <td>3449</td>
      <td>Music for the Royal Fireworks, HWV351 (1749): ...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>29</th>
      <td>3451</td>
      <td>Die Zauberflöte, K.620: "Der Hölle Rache Kocht...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>30</th>
      <td>3454</td>
      <td>Symphony No. 41 in C Major, K. 551, "Jupiter":...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>31</th>
      <td>3481</td>
      <td>A Midsummer Night's Dream, Op.61 Incidental Mu...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>32</th>
      <td>3482</td>
      <td>Suite No. 3 in D, BWV 1068: III. Gavotte I &amp; II</td>
      <td>5</td>
    </tr>
    <tr>
      <th>33</th>
      <td>3485</td>
      <td>Symphony No. 3 Op. 36 for Orchestra and Sopran...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>34</th>
      <td>3489</td>
      <td>Symphony No. 2: III. Allegro vivace</td>
      <td>5</td>
    </tr>
    <tr>
      <th>35</th>
      <td>3490</td>
      <td>Partita in E Major, BWV 1006A: I. Prelude</td>
      <td>5</td>
    </tr>
    <tr>
      <th>36</th>
      <td>3492</td>
      <td>Sing Joyfully</td>
      <td>5</td>
    </tr>
    <tr>
      <th>37</th>
      <td>3493</td>
      <td>Metopes, Op. 29: Calypso</td>
      <td>5</td>
    </tr>
    <tr>
      <th>38</th>
      <td>3498</td>
      <td>Concerto for Violin, Strings and Continuo in G...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>39</th>
      <td>3499</td>
      <td>Pini Di Roma (Pinien Von Rom)  I Pini Della Vi...</td>
      <td>5</td>
    </tr>
    <tr>
      <th>40</th>
      <td>3503</td>
      <td>Koyaanisqatsi</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>


