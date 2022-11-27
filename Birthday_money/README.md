# Birthday Money

## Source

| Key | Value                                                                                                     |
| ------ |-----------------------------------------------------------------------------------------------------------|
| Author| Ram Parameswaran                                                                                          |
| Link| [![Birthday Monday](https://img.youtube.com/vi/qnn8p8xaP1U/0.jpg)](https://youtu.be/qnn8p8xaP1U?t=3999)   |
| Retrieved | 2022-Nov-27 |

Initially the policy for birthday money was calculated as $10.00 per year. Here is an intensionally defined relation (IDR), or 
table that captures this.

```
declare  
    var int: age;
    var currency: birthday_money;
    constraint birthday_money = age*10.00;
giving standard_birthday_money"v1"
```

with this table defined the following queries give the following answers:

```
# Query 1
select * from standard_birthday_money where age = 10;
```

| age | birthday_money |
|-----|----------------|
| 10  | $100.00        |

```
# Query 2
select * from standard_birthday_money where birthday_money=80;
```

| age | birthday_money |
|-----|----------------|
| 8   | $80.00         |

Some time later, the approach to birthday money was changed. Here is the code which updates the IDR.

```
with standard_birthday_money"v1"
declare
    var bool: good_behaviour;
    var date: this_birthday;
    par date: v2_start = "2000-01-01"
retract
    constraint birthday_money = age*10;
declare
    constraint birthday_money = if this_birthday>= v2_start then 
            if good_behaviour then age*20.00 else 0.00 endif
        else
            age*10.00
        endif
giving standard_birthday_money"v2"
```

Following the update, ```Query 1``` now returns no result:

```
# Query 1
select * from standard_birthday_money where age = 10;
```

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---|

Notice that the table now has two new columns that we added in v2. We have had no result returned because in the situation where
the full result would go on forever.
The birthday
money depends on
which year that the birthday occurred in. Therefore, to answer this
query the list would start at the big bang and go until the heat death of the universe. Instead of attempting that, the IDR
returns an empty table as the answer.

If we update the query to indicate that the birthday is this year:

```
# Query 3
select * from standard_birthday_money where age = 10 and this_birthday = '2022-11-27';
```
we get two results which expresses what we know given the information provided in the query.

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---|
| 10 | 0.00           | false | 2022-11-27 |
|10 | 200.00         | true | 2022-11-27|

As soon as we know the behaviour of the person, we can query the actual birthday money as follows:
```
# Query 4
select * from standard_birthday_money where age = 10 and this_birthday = '2022-11-27' and good_behaviour = true;
```
we get two results which expresses what we know given the information provided in the query.

| age | birthday_money | good_behaviour | this_birthday |
|-----|----------------|---|---|
|10 | 200.00         | true | 2022-11-27|

