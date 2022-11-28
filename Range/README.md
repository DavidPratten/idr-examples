# Range

When appropriately constrained in a query, the ```range``` intentionally defined relation (IDR) returns all elements in a 
range of 
```N``` equally spaced 
values ```FromStart``` to ```ToStop```.  

The ```range``` relation is used to illustrate three aspects of intensionally defined relations:
-  Multi-row result sets
-  The folding of constraints from the query environment down to create a new intensionally defined relation that may be queried without a ```WHERE``` clause
-  Querying a relation from multiple directions as we normally do for extensionally defined relations.

Here is the definition using a [MiniZinc](https://www.minizinc.org/) like form.

```

declare
    var float: FromStart;
    var float: ToStop;
    var int: N;
    var 0..N-1: nth;
    var float: nthValue;
    constraint nthValue = FromStart + (ToStop-FromStart) / (N-1) * nth;
giving range

```

The first example query illustrates how an intensionally defined relation may return multiple rows when queried:
```

select * from range where N=5 and FromStart=10 and ToStop = 20;
    
```
gives a result which starts at 10, finishes at 20 and has 5 values in the sequence:

| FromStart | ToStop | N | nth | nthValue |
|-----------|--------|---|-----|----------|
| 10.0      | 20.0   | 5 | 0   | 10       |
| 10.0      | 20.0   | 5 | 1   | 12.5     |
| 10.0      | 20.0   | 5 | 2   | 15       |
| 10.0      | 20.0   | 5 | 3   | 17.5     |
| 10.0      | 20.0   | 5 | 4   | 20       |

The second example query shows how the contents of ```WHERE``` clauses may be folded into intentional definition of the relation.

```
select * from range 
    where N=5 
        and FromStart=10 
        and ToStop = 20 
        and nthValue < 13;
    
```
gives a two row resultset:

| FromStart | ToStop | N | nth | nthValue |
|-----------|--------|---|-----|----------|
| 10.0      | 20.0   | 5 | 0   | 10       |
| 10.0      | 20.0   | 5 | 1   | 12.5     |

It is an implementation detail as to what plan was followed to generate this result.  Here are a couple of alternative plans:

1.  Compute the result of the first example query and filter out rows which do not have ```nthValue < 13```
2.  Push the "```nthValue < 13```" constraint down into an intensionally defined relation ```range1``` and query it instead of 
    ```range```.

Here is an intensional definition of a relation where the query constraints in the ```WHERE``` clause are pushed down into the MiniZinc-like definition. Notice that equality constraints turn variables (```var```) into parameters (```par```):
```

declare
    par float: FromStart=10.0;
    par float: ToStop=20.0;
    par int: N = 5;
    var 0..N-1: nth;
    var float: nthValue;
    constraint nthValue = FromStart + (ToStop-FromStart) / (N-1) * nth;
    constraint nthValue < 13.0;
giving range1

select * from range1;

```

The third example query illustrates how the ```range``` relation may be queried from multiple directions, as is usually expected for relations:
```

select * from range 
    where N=5 
        and ToStop = 20 
        and nthValue=15.0;
    
```
gives a relation which finds the FromStart and Nth values which belong to sequences with nthValue of 15.0 which finish at 20 and have 5 values in each sequence.  There are four tuples in the result:

| FromStart | ToStop | N | nth | nthValue |
|-----------|--------|---|-----|----------|
| 15.0      | 20.0   | 5 | 0   | 15.0     |
| 12.5      | 20.0   | 5 | 1   | 15.0     |
| 10.0      | 20.0   | 5 | 2   | 15.0     |
| 0.0       | 20.0   | 5 | 3   | 15.0     |
