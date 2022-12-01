# ACT Conveyance Duty
The following is an intensionally defined relation (IDR) that captures
the relationship between price and [conveyance duty in the ACT from 1 July 2022](https://www.revenue.act.gov.au/duties/conveyance-duty). The duty payable depends firstly on whether the purchase is non-commercial or commercial and 
secondly, if non-commercial, whether the purchase is by an owner occupier. The definition is
written MiniZinc.

Callouts from this example include:

- the usage of a library IDR `units` to capture the idea of applying the duty to units of hundred dollars rounding up. 
- the IDR captures the rule once and is queryable in any direction, just like an ordinary database table


```
% units IDR calculates the number of units of "unit_size" with optional "round_up"
predicate units(var int:amount, var int:unit_size, var bool: round_up, var int:units ) = 
  units == amount div unit_size + if round_up then amount mod unit_size > 0 else 0 endif;

predicate ACT_Conveyance_Duty(var bool: non_commercial, var bool: eligible_owner_occupier, var int:price, var float:duty) = 
let {
  constraint eligible_owner_occupier -> non_commercial,
  constraint price >= 0,
  constraint duty >= 0,
  var int: num_units,
  int: unit = 100;} 
in duty == 
  if non_commercial then
    if eligible_owner_occupier then
      if price > 0       /\ price <=260000  /\ units(price, unit, true, num_units) then num_units * 60 else 
      if price > 260000  /\ price <=300000  /\ units(price-260000, unit, true, num_units)  then 1560.00  + num_units * 2.20 else 
      if price > 300000  /\ price <=500000  /\ units(price-300000, unit, true, num_units)  then 2440.00  + num_units * 3.40 else 
      if price > 500000  /\ price <=750000  /\ units(price-500000, unit, true, num_units)  then 9240.00  + num_units * 4.32 else 
      if price > 750000  /\ price <=1000000 /\ units(price-750000, unit, true, num_units)  then 20040.00 + num_units * 5.90 else 
      if price > 1000000 /\ price <=1455000 /\ units(price-1000000, unit, true, num_units) then 34790.00 + num_units * 6.40 else 
      if price > 1445000                    /\ units(price, unit, true, num_units) then num_units * 4.54 
      else -1 endif endif endif endif endif endif endif
    else % not eligible_owner_occupier
      if price > 0       /\ price <=200000  /\ units(price, unit, true, num_units)         then num_units * 120 else 
      if price > 200000  /\ price <=300000  /\ units(price-200000, unit, true, num_units)  then 2400.00  + num_units * 2.20 else 
      if price > 300000  /\ price <=500000  /\ units(price-300000, unit, true, num_units)  then 4600.00  + num_units * 3.40 else 
      if price > 500000  /\ price <=750000  /\ units(price-500000, unit, true, num_units)  then 11400.00 + num_units * 4.32 else 
      if price > 750000  /\ price <=1000000 /\ units(price-750000, unit, true, num_units)  then 22200.00 + num_units * 5.90 else 
      if price > 1000000 /\ price <=1455000 /\ units(price-1000000, unit, true, num_units) then 36950.00 + num_units * 6.40 else 
      if price > 1445000                    /\ units(price, unit, true, num_units)         then num_units * 4.54 
      else -1 endif endif endif endif endif endif endif
    endif
  else % commercial
    if price > 0         /\ price <= 1700000                                          then 0 else
    if price > 1700000                      /\ units(price, unit, true, num_units)         then num_units * 5.00 else -1 endif endif
  endif
; 
```
## Querying the IDR
Here is a selection of queries that demonstrate the number of different insights the IDR can provide.

### How much duty for a home occupier on a house of $1.2M?
```
-- Example 1
select duty from ACT_Conveyance_Duty where price = 1200000 and eligible_owner_occupier;

```
gives

| duty   |
|--------|
| 47,590 |

### How much duty for is chargeable for a property valued at $1.2M?
```
-- Example 2
select * from ACT_Conveyance_Duty where price = 1200000;

```
shows the comparative conveyancing duty across the various pricing regimes. Notice that the IDR has deduced and is reporting that 
there are 
_three_ categories of payers.

| non_commercial | eligible_owner_occupier | price     | duty    |
|----------------|-------------------------|-----------|---------|
| true| true| 1,200,000 |  47,590 |
|true| false | 1,200,000 | 49,750 |
| false | false| 1,200,000 | 0|
 
### How much duty for is chargeable for a property valued at $2M?
```
-- Example 3
select * from ACT_Conveyance_Duty where price = 2000000;

```
shows the convergence of the owner occupied and non-occupied regimes at higher prices.

| non_commercial | eligible_owner_occupier | price     | duty    |
|----------------|-------------------------|-----------|---------|
| true| true| 2,000,000 | 140,740 |
|true| false | 2,000,000 | 140,740 |
| false | false| 2,000,000 | 155,000 |

### An owner occupier pays $50,150 in conveyancing duty.  What is the corresponding house price?
This query
```
-- Example 4
select price from ACT_Conveyance_Duty where duty = 50150 and eligible_owner_occupier;
```
shows that there is more than one possible price that gives a duty value of $50,150. The IDR has deduced that there are 100 
such values and reports them.

| price     |
|-----------|
| 1,239,901 |
| 1,239,902 |
| 1,239,903 |
| ...       |
| 1,239,998 |
| 1,239,999 |
| 1,240,000 |

### A property owner pays 140,740 in conveyancing duty. What are the corresponding property prices?
```
-- Example 5
select non_commercial, 
    eligible_owner_occupier, 
    min(price) min_price, 
    max(price) max_price 
from ACT_Conveyance_Duty 
    where duty = 140740 ;
```
shows the price ranges across the three regimes

| non_commercial | eligible_owner_occupier | min_price | max_price |
|----------------|-------------------------|-----------|-----------|
| true| true| 3,099,901 | 3,100,000 |
|true| false | 3,099,901 | 3,100,000 |
| false | false| 2,814,701 | 2,814,800 |

