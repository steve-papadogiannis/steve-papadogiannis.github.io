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
