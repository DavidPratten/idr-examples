# Examples of Intensionally Defined Relations

## A table by any other name!
In the examples found here a relation is "a table of data, with columns and rows". ( The word "relation" here merely flags 
that there is a connection to the relational database model.)

## A table can be defined in two ways
Every table has an intension, which is its intended meaning and its extension which is its list of rows. But not all tables 
are defined in the same way!  

| How defined           | Intension (a test that is true if a row is a member of the table) | Extension (list of rows)          |
|-----------------------|-------------------------------------------------------------------|-----------------------------------|
| Extensionally defined | Natural Language                                                  | Listed out in a table             |
| Intensionally defined | Computable Constraints and Rules                                  | Generated or recognised on demand |

Our databases and CRMs are full of extensionally defined tables, here are a few:

| Intension (intended meaning) | Extension (list of rows)           |
|------------------------------|------------------------------------|
| Our Customers                | 1 row in a table for each customer |
| Our Orders                   | 1 row in a table for each order    |

## Why Intensionally Defined Relations (IDRs)?
The value of IDRs as a construct is an open question. Examples in this repository are designed to illustrate advantages and 
drawbacks of IDRs and are mainly drawn 
from 
the Rules As Code domain.

## Examples of Intensionally Defined Relations (IDRs)
This repository contains a set of worked examples of IDRs.

- [Birthday Money](https://github.com/DavidPratten/idr-examples/tree/main/Birthday_money)
- [Range](https://github.com/DavidPratten/idr-examples/tree/main/Range)
- [Australian GST](https://github.com/DavidPratten/idr-examples/tree/main/Australian_GST)




