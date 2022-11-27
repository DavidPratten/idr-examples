# Birthday Money

## Source

| Key                | Value                                                                                                   |
|--------------------|---------------------------------------------------------------------------------------------------------|
| Author             | Ram Parameswaran                                                                                        |
| Link               | [![Birthday Monday](https://img.youtube.com/vi/qnn8p8xaP1U/0.jpg)](https://youtu.be/qnn8p8xaP1U?t=3999) | 
| Langauge | OpenFisca                                                                                               |
| Retrieved          | 2022-Nov-27                                                                                             |

## Initially a flat rate that applies to all children

Initially the policy for birthday money was calculated as $10.00 per year. Here is an intensionally defined relation (IDR) 
```standard_birthday_money``` that captures this.

```
% this is a comment

declare  
    var int: age; % var defines an attribute of the relation (column of the table)
    var currency: birthday_money;
    constraint birthday_money = age*10.00; % constraints express a relationship
giving standard_birthday_money__v1 % __ v1 here is an optional name of the
                                   % definition used for literate
                                   % programming below.
```

The language used to define IDRs is not so important, the crucial aspect is that the reader, and the computing system, understand the intension.  The 
definition above happens to have been written in [MiniZinc](https://www.minizinc.org/).  

IDRs can be queried as if it were an ordinary database table that contains _all_ the answers. With this table defined, the 
following 
query:

```
# Query 1
select * from standard_birthday_money where age = 10;
```
gives the expected answer:

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

And Query 2 also gives the same answer!
```
# Query 2
select * from standard_birthday_money where birthday_money=100;
```

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

## Made conditional on behaviour

From January 1st, 2000, the the amount of birthday money was made contingent on the child's behaviour. Here is the literate 
programming code 
which 
updates the IDR 
to a 
second version. 

```
with standard_birthday_money__v1
declare
    var bool: good_behaviour;
    var date: this_birthday;
    par date: v2_start = "2000-01-01" % "par" or "parameter" defines a fixed value
retract
    constraint birthday_money = age*10;
declare
    constraint birthday_money = if this_birthday>= v2_start then 
            if good_behaviour then age*20.00 else 0.00 endif
        else
            age*10.00
        endif
giving standard_birthday_money__v2
```

Following the literate programming update, ```standard_birthday_money``` now has the following definition:

```
declare  
    var int: age;
    var currency: birthday_money;
    var bool: good_behaviour;
    var date: this_birthday;
    par date: v2_start = "2000-01-01"
    constraint birthday_money = if this_birthday>= v2_start then 
            if good_behaviour then age*20.00 else 0.00 endif
        else
            age*10.00
        endif
giving standard_birthday_money
```




If we now repeat ```Query 1``` we get a different output.

```
# Query 1
select * from standard_birthday_money where age = 10;
```

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---|

Notice firstly, that the table has the two new columns that we added in v2, and secondly that we have had no result returned. 

The amount of birthday
money now depends on
which year that the birthday occurred in. Therefore, to answer ```Query 1``` as stated above, the list would have to give an 
answer for every year, ever. Instead of attempting that, the IDR
returns an empty table as the answer.

If we update the query to indicate that the birthday is in a particular year:

```
# Query 3
select * from standard_birthday_money 
    where age = 10 and this_birthday = '2000-11-27';
```

we get two results which expresses what we know to be true.

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---------------|
| 10 | 0.00           | false | 2000-11-27    |
|10 | 200.00         | true | 2000-11-27    |

This may be read as "a child turning ten in 2000 will receive either nothing, or two hundred dollars birthday money"

As soon as we know the behaviour of the person, we can find the actual birthday money with the following query:

```
# Query 4
select * from standard_birthday_money where age = 10 and this_birthday = '2022-11-27' and good_behaviour = true;
```

it turns out this person had 'good_behaviour', (however that was defined) and will therefore be given $200.00 for their birthday.

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---------------|
|10 | 200.00         | true | 2000-11-27    |

# Test Data
To test an IDR the test data are prepared as a table which captures a portion of the values that are intended to be part of it.
Here is a small sample of rows from the infinite rows that ```standard_birthday_money``` represents:

```standard_birthday_money_test```

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|----------------|---------------|
| 5   | 50.00          | true           | 1999-12-31    |
| 10  | 100.00         | true           | 1999-12-31    |
| 5   | 50.00          | false          | 1999-12-31    |
| 10  | 100.00         | false          | 1999-12-31    |
| 5   | 100.00         | true           | 2000-01-01    |
| 10  | 200.00         | true           | 2000-01-01    |
| 5   | 0.00         | false          | 2000-01-01    |
| 10  | 0.00         | false          | 2000-01-01    |

The correctness of this IDR can be tested simply by comparing, row by row, what would be returned by the IDR for each.

## Versions of Intensionally Defined Relations
When declared, Intensionally Defined Relations may be labelled with a version reference. Above we have two versions of 
```standard_birthday_money```:
1. ```standard_birthday_money__v1```, and
2. ```standard_birthday_money__v2```.

When queried, without a label the most recently declared version is used.  

If a user specifically wishes to access a specific version, the optional definition identifier may be used in the query.
```
# Query 5
select * from standard_birthday_money__v1 where age = 10;
```

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

