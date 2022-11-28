# Australian GST Example

The following is an example intensionally defined relation (IDR) that captures
the relationship between consumer prices, price without the Australian
Goods and Consumption Tax (GST), and the GST amount. The definition is
written using a linguistic form inspired by MiniZinc . At the end, an alternative SQL-like form for defining IDRs is shown.


``` 

declare 
    var money: Price;
    var money: ExGSTAmount;
    var money: GSTAmount;
    constraint Price/11 = GST;
    constraint Price-GST = ExGST;
giving GST__A_New_Tax_System_Goods_and_Services_Tax_Act_1999
```

The above definition may be read as follows:

  - Declaration of headings (attributes) using var \<type\>: \<name\>.

  - Two declarations of the relationships between attributes using
    constraints. 

  - The intensionally defined relation is given the name "GST".

  - This particular declaration of the intensionally defined relation is
    labelled with the name of the source of this definition: "A_New_Tax_System_Goods_and_Services_Tax_Act_1999".

The constraints above, should be read in light of the fact that the
following constraints all express the same relationship between Price
and GST: 

``` 

constraint Price/11 = GST;
constraint GST = Price/11;
constraint GST*11 = Price;
constraint Price = GST*11;
```

The following is the behaviour of this intensionally defined relation
under various queries. The GST relation returns an empty extension when
insufficiently constrained to lead to a finite extension.

    select * from GST;

| price | ExGSTAmount | GSTAmount |
| :---- | :---------- | :-------- |

Query result is an empty relation.

The same empty result is obtained when the relation is constrained in a
manner that violates the intension. For example:

``` 

select * from GST where Price=1 and ExGSTAmount=1 and GSTAmount=1;
```

When, however, any one of its attributes is constrained to a value, it
returns a single tuple extension.

``` 

select * from GST where Price = 110;
```

| price | ExGSTAmount | GSTAmount |
| :---- | :---------- | :-------- |
| 110   | 100         | 10        |

Query result is the corresponding single tuple relation.

The same result is obtained with any of the following queries.

``` 

select * from GST where ExGSTAmount = 100;
select * from GST where GSTAmount = 10;
select * from GST where GSTAmount = 10 and Price=110;
select * from GST where GSTAmount = 10 and ExGSTAmount=100;
select * from GST where ExGSTAmount = 100 and Price=110;
select * from GST where ExGSTAmount = 100 and Price=110 and GST=10;
```

The above example demonstrates how programmers do not need to repeat
themselves
“[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)” when
driving computations in different directions. This is in sharp contrast
to non-declarative programming, whereby the GST computation needs to be
restated multiple times, depending on what value is known and which ones
are unknown. 

## Alternative definition

To complete the example here is a SQL-like form for specifying the GST
intensionally defined relation. This definition overloads the `VIEW`
construct with typed attribute names. Under query this `VIEW` will
behave exactly as the `GST` intentional relation defined earlier:

``` 

create view GST
    (money: Price, money: ExGSTAmount, money: GSTAmount ) as 
    select Price, Price-GSTAmount, Price\11;
    
```
