# Australian Goods and Services Tax (GST) Example

The following is an example intensionally defined relation that captures
the relationship between consumer Price, the Goods and Services Tax
(GSTAmount), and the price before application of the GST (ExGSTAmount).
The definition is written in MiniZinc .

``` 

predicate Australian_GST(var float: Price, 
                         var float: ExGSTAmount, 
                         var float: GSTAmount) = 
let {
    constraint Price/11 = GSTAmount;
    constraint ExGSTAmount = Price-GSTAmount;
    } 
in true;
```

The above definition may be read as follows:

  - `predicate` is the MiniZinc term meaning "relation".

  - The intensionally defined relation is given the name
    `Australian_GST`.

  - Declaration of headings (attributes) using `var <type>: <name>`.

  - Two declarations of the relationships between attributes using
    constraints.

The constraints above, should be read in light of the fact that the
following constraints all express the same relationship between Price
and GSTAmount: 

``` 

constraint Price/11 = GSTAmount;
constraint GSTAmount = Price/11;
constraint GSTAmount*11 = Price;
constraint Price = GSTAmount*11;
```

The following is the behaviour of this intensionally defined relation
under various queries. The `Australian_GST` relation returns an empty
extension when insufficiently constrained to lead to a finite extension.

    select * from Australian_GST;

| price | ExGSTAmount | GSTAmount |
| :---- | :---------- | :-------- |

Query result is an empty relation.

The same empty result is obtained when the relation is constrained in a
manner that violates the intension. For example:

``` 

select * from Australian_GST where Price=1 and ExGSTAmount=1 and GSTAmount=1;
```

When, however, any one of its attributes is constrained to a value, it
returns a single tuple extension.

``` 

select * from Australian_GST where Price = 110;
```

And the same result is obtained with any of the following queries.

``` 

select * from Australian_GST where ExGSTAmount = 100;
select * from Australian_GST where GSTAmount = 10;
select * from Australian_GST where GSTAmount = 10 and Price=110;
select * from Australian_GST where GSTAmount = 10 and ExGSTAmount=100;
select * from Australian_GST where ExGSTAmount = 100 and Price=110;
select * from Australian_GST where ExGSTAmount = 100 and Price=110 and GSTAmount=10;
```

| price | ExGSTAmount | GSTAmount |
| :---- | :---------- | :-------- |
| 110   | 100         | 10        |

Query result is the corresponding single tuple relation.

The above example demonstrates how programmers do not need to repeat
themselves "DRY”  when driving computations in different directions.
This is in sharp contrast to procedural and functional programming,
whereby the GST computation needs to be restated multiple times,
depending on what value is known and which ones are unknown.
