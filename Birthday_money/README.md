# Birthday Money

## Source

| Key               | Value                                                                                                   |
|-------------------|---------------------------------------------------------------------------------------------------------|
| Author            | Ram Parameswaran                                                                                        |
| Link              | [![Birthday Monday](https://img.youtube.com/vi/qnn8p8xaP1U/0.jpg)](https://youtu.be/qnn8p8xaP1U?t=3999) | 
| Original Language | OpenFisca                                                                                               |
| Retrieved         | 2022-Nov-27                                                                                             |

_Lets revisit the example above using intensionally defined relations (IDRs)..._

## Initially a flat rate that applies to all children

Initially the policy for birthday money was calculated as $10.00 per year. Here is an intensionally defined relation (IDR) 
```standard_birthday_money``` that captures this.

```
% this is a comment

predicate standard_birthday_money(
        var int: age, % var defines an attribute of the relation (column of the table)
        var float: birthday_money) =
let {
    constraint birthday_money = age*10.00; % constraints express a relationship
} in true;
```

The language used to define IDRs is not so important, the crucial aspect is that the reader, and the computing system, understand the intension.  The 
definition above happens to have been written in [MiniZinc](https://www.minizinc.org/).  

IDRs can be queried as if they are an ordinary database table that contains _all_ the answers. With this table defined, the 
following 
query:

```
-- Query 1
select * from standard_birthday_money where age = 10;
```
gives the expected answer:

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

And Query 2 also gives the same answer!
```
-- Query 2
select * from standard_birthday_money where birthday_money=100;
```

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

Notice that the relationship can be queried from multiple directions.

## Made conditional on behaviour

At some point of time, the amount of birthday money was made contingent on the child's behaviour. Here is the second version. 

```
predicate standard_birthday_money(
    var int: age, % var defines an attribute of the relation (column of the table)
    var bool: good_behaviour,
    var float: birthday_money) =
let {
    constraint birthday_money = if good_behaviour then age*20.00 else 0.00 endif
} in true;
```

If we now repeat ```Query 1``` we get a different output.

```
-- Query 1
select * from standard_birthday_money where age = 10;
```

we get two results which expresses what we know to be true.

| age | birthday_money | good_behaviour | 
|-----|----------------|---|
| 10 | 0.00           | false | 
|10 | 200.00         | true | 

This may be read as "a child turning ten will receive either nothing, or two hundred dollars birthday money"

As soon as we know the behaviour of the person, we can find the actual birthday money with the following query:

```
-- Query 4
select * from standard_birthday_money 
    where age = 10 
        and good_behaviour = true;
```

it turns out this person had 'good_behaviour', (however that was defined) and will therefore be given $200.00 for their birthday.

| age | birthday_money | good_behaviour |
|-----|----------------|---|
|10 | 200.00         | true |

And again to show that the direction of querying is flexible
```
-- Query 5
select good_behaviour from standard_birthday_money 
    where age = 10 
        and birthday_money = 200;
```

will yield:

| good_behaviour |
|----------------|
| true           |

# Test Data
To test an IDR the test data are prepared as a table which captures a portion of the values that are intended to be part of it.
Here is a small sample of rows from the infinite rows that ```standard_birthday_money``` represents:

```standard_birthday_money_test```

| age | birthday_money | good_behaviour |
|-----|----------------|----------------|
| 5   | 100.00         | true           |
| 10  | 200.00         | true           | 
| 5   | 0.00         | false          | 
| 10  | 0.00         | false          | 

The correctness of this IDR can be tested simply by comparing, row by row, what would be returned by the IDR for each.

