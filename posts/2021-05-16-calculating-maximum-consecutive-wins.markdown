---
title: Calculate Maximum Consecutive Wins in SQL
---

## Intro

In this post I will elaborate on the concept of calculating 
the maximum consecutive wins on a **game-like** project via **SQL**.

Let\'s put things in context. Assume you have a project that has a
gamified aspect in its domain where you have users who win some kind of game.
In this context, you may want to award players based on their performance and
especially for the maximum consecutive wins. So, you need a way to 
have a calculation deriving from your data for this accumulation.
Another assumption is that the game data are stored in some kind of database
that supports **SQL**.

---

## Setup

The practical example will be in **PL/SQL** of **Oracle DB** 

We will use a table with at least the below specified columns, named **GAMES**:


<div class="format-inner-table">

| Column Name      | Data Type    | Scope                                                            |
|------------------|--------------|------------------------------------------------------------------|
| ID               | NUMBER       | The unique identifier of a game and the primary key of the table |
| PLAYER_ID        | NUMBER       | The unique identifier of a player                                |
| OPPONENT_ID      | NUMBER       | The unique identifier of a player                                |
| IS_PLAYER_WINNER | NUMBER(1, 0) | A flag indicating whether the player won                         | 
| LOG_DATE         | TIMESTAMP(6) | A timestamp for the game                                         |

</div>

The **PLAYER_ID** and the **OPPONENT_ID** match to a **USER_ID** of the domain, which means
that they can be foreign keys to another table **USERS** representing the users of the domain
(_for the purpose of this post, the actual correlation is out of scope_).

In addition, if the game is also playable in a **non-PVP** mode then the **OPPONENT_ID**
can have the same **USER_ID** as the **PLAYER_ID**.

Another practical assumption that holds is that the **PLAYER_ID** is the initiator of 
the game and a real user, so we can have **three** game cases:

* A player (**USER_ID**) starts a game (**USER_ID = PLAYER_ID**) 
  versus another player (**PLAYER_ID != OPPONENT_ID**)
* A player (**USER_ID**) starts a game (**USER_ID = PLAYER_ID**)
  versus the program (_non-PVP, with Bot_) (**PLAYER_ID = OPPONENT_ID**)
* A player (**USER_ID**) joins (**USER_ID = OPPONENT_ID**) another\'s player game 
  (**OPPONENT_ID != PLAYER_ID**)
  
Below is the table creation script:

<script src="https://gist.github.com/steve-papadogiannis/3e3fcb0d34bfbb6e826b2df31cc1d934.js"></script>

---

## Union (\'em all)

With the basic domain and assumptions defined we can proceed to defining the data set that
we will aggregate upon.

So, we have **three** game cases that will be matched to **three** basic **Select**s:

* Get all **PVP** games that are started by a specific player:

<script src="https://gist.github.com/steve-papadogiannis/e416f62a1e4e4c562957af9a2704ed19.js"></script>

* Get all **non-PVP** games that are started by a specific player:

<script src="https://gist.github.com/steve-papadogiannis/c1ad8de2e6f1593e78d9107e8c67bc64.js"></script>

* Get all **joined** games that a specific user has participated:

<script src="https://gist.github.com/steve-papadogiannis/ad4b41f2b1987bda9f8dfabccc29c84f.js"></script>

---

## Key Concept

Ok, so now, we have to come up with a basic calculation that will 
lead us to the solution of the problem. From a top-down perspective 
we can see that we want to end up with the maximum of a number of 
consecutive wins. 

What, however, identifies as a consecutive win?

That, will be a difference between an end point,
and a start point, in our data, of marked wins where no loses are intervened.

Furthermore, with the basic calculation at hand, now, we are missing an
intermediate step in order to reify the key concept and that will be 
the demarcation of the aforementioned points in our data set.

---

## Demarcation of Wins

As a start point in a row of wins in our data set we can mark 
a row that represents a game the player won, and the previous row
of that row represents a game the player lost.

In order to define the \"previous row\" concept we have to have a column
of the rows that we will traverse upon and order it by
another specific column. That, would be a **PARTITION** of the data set
by the **PLAYER_ID** which should be ordered by (_ascending_) 
the **LOG_DATE** of each row.

The **SELECT** statement should be like this:

<script src="https://gist.github.com/steve-papadogiannis/e908684f01768947eab25a3f20eae61d.js"></script>

Let\'s elaborate on this a little bit more. If the player has won this
game (**IS_PLAYER_WINNER = 1**) and the previous game (**LAG(\...)**)
of the partition is a loss (**NVL(LAG(IS_PLAYER_WINNER) OVER (PARTITION BY PLAYER_ID ORDER BY LOG_DATE), 0) = 0**)
(_Let\'s take a pause here to clarify the use of **NVL**. The first row of 
the data set does not have a previous row so in order to not break the calculation
we assume that this absence of previous row is equivalent to a loss_) then we
return its **ROW_NUMBER** in order to mark this row as the start of a (maybe) 
sequence of wins.

The same thought process applies for the definition of the end point.


As an end point in a row of wins in our data set we can mark
a row that represents a game the player won, and the next row
of that row represents a game the player lost.

To define the \"next row\" concept, our new deus ex machina is the
**LEAD** function.

The **SELECT** statement should be like this:

<script src="https://gist.github.com/steve-papadogiannis/16ec415bad4871d923db2314042e4c85.js"></script>

(_A similar explanation is due for the use of **NVL** here. The last row of
the data set does not have a next row so in order to not break the calculation
we assume that this absence of next row is equivalent to a loss_)

Let\'s wrap up the section with the whole query thus far:

<script src="https://gist.github.com/steve-papadogiannis/fb83a698ec85b95189843c611e174415.js"></script>

---

## Consecutive Wins

The intermediate step is well-defined, so, the calculation of 
consecutive wins is as simple as this:

<script src="https://gist.github.com/steve-papadogiannis/6f9bb5da037dcce0e02d4a07e6ceb2c1.js"></script>

Here we cast out the unnecessary rows, meaning, either the ones that
are the intermediate wins in a sequence of wins (**WINS_START IS NULL AND WINS_END IS NULL**,
they are neither a starting point, nor an end point for the calculation),
or the ones that are single wins (**WINS_START IS NOT NULL AND WINS_END IS NOT NULL**)
which cannot identify as consecutive wins. 

With the remaining data
we calculate the differences between the **ROW_NUMBER** of any next row
with the **ROW_NUMBER** of the current row (plus one).

The final result in this step is a collection of consecutive wins for 
the specific player of our game.

--- 

## Get the Maximum

Now we can get the **MAX** of the consecutive wins and also have a 
fail safe clause when we may not have any data for a user (**SELECT 0 AS MAX_CONSECUTIVE_WINS
FROM DUAL**):

<script src="https://gist.github.com/steve-papadogiannis/3a34b592b12f1b288a485394e39fcdd9.js"></script>

---

## Outro

Let\'s now revisit the steps of the calculation to sum up the solution given:

* We define the data set of the calculation (_the info that is useful_)
* We mark the eligible data with their row number
* We calculate the differences of the eligible rows
* We get the maximum difference or zero if no data at all

