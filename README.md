# Data Engineering - Superstore Dataset

* Author: Michael Pheasant
* See [exploratory data analysis](data-eng.ipynb)
 ipython notebook for ETL results

# Source & Context

* https://www.kaggle.com/jr2ngb/superstore-data
* Retail dataset of a global superstore for 4 years.

# Requirements: Queries

## Qry 1

* Total sales by month for the last 4 years.
* Dimensions: (sales, month, year)

## Qry 2

* Top 5 products sold during January period, along with category and sub-category names.
* Dimensions: (product, category, sub-category, sales, month)

## Qry 3

* Total quantity sales for weekends by region by week.
* Dimensions: (region, week, weekday, sales)

# Dimensions of interest

Sales, broken down by: 
* time: month, year, week, weekday
* customer
* region
* product, category, sub-category

# Solution: OLAP Star Schema

* Develop star schema based on Sale Item (line item on an order)

<dl>
    <dt>Fact:</dt>
    <dd>SaleItem</dd>
    <dt>Dimensions:</dt>
    <dd>Customer, Product, SaleDate, MarketRegion</dd>
</dl>


```
            MarketRegion
                ^
                |
Product <- SaleItem -> Customer
                |
                V
            SaleDate
```

## Schema DDL 

* PostgresQL v12 DB on linux

```sql
-- OLAP Dimension Tables
create table SaleDate (
    SaleDateID          varchar not null,
    SaleDate            date not null,
    SaleYear            int not null,
    SaleMonth           int not null,
    SaleISOWeekNumber   int not null,
    SaleISOWeekDay      int not null,
    primary key (SaleDateID)
);

create table Customer (
    CustomerID      varchar not null,
    Segment         varchar not null,
    CustomerName    varchar not null,
    primary key (CustomerID)
);

create table Product (
    ProductID   varchar not null,
    Category    varchar not null,
    SubCategory varchar not null,
    ProductName varchar not null,
    primary key (ProductID)
);

create table MarketRegion (
    MarketRegionID  varchar not null,
    Market          varchar not null,
    Region          varchar not null,
    primary key (MarketRegionID)
);

-- OLAP Fact Tables
create table SaleItem (
    SaleItemID      int not null,
    CustomerID      varchar not null,
    ProductID       varchar not null,
    SaleDateID      varchar not null,
    MarketRegionID  varchar not null,
    Sales           decimal not null,
    Quantity        int not null,
    Discount        decimal not null,
    Profit          decimal not null,
    ShippingCost    decimal not null,
    primary key (SaleItemID)
);

-- load data (using psql)
\copy saledate from '/home/michael/Documents/datademo/load_saledate.csv' encoding 'UTF-8' csv header ;
-- COPY 25754
\copy customer from '/home/michael/Documents/datademo/load_customer.csv' encoding 'UTF-8' csv header ;
-- COPY 1590
\copy product from '/home/michael/Documents/datademo/load_product.csv' encoding 'UTF-8' csv header ;
-- COPY 10292
\copy marketregion from '/home/michael/Documents/datademo/load_marketregion.csv' encoding 'UTF-8' csv header ;
-- COPY 18
\copy saleitem from '/home/michael/Documents/datademo/load_saleitem.csv' encoding 'UTF-8' csv header ;
-- COPY 51290

-- Add Foreign Keys
ALTER TABLE SaleItem ADD CONSTRAINT fk_SaleItem_SaleDate FOREIGN KEY (SaleDateID) REFERENCES SaleDate (SaleDateID);
ALTER TABLE SaleItem ADD CONSTRAINT fk_SaleItem_Customer FOREIGN KEY (CustomerID) REFERENCES Customer (CustomerID);
ALTER TABLE SaleItem ADD CONSTRAINT fk_SaleItem_Product FOREIGN KEY (ProductID) REFERENCES Product (ProductID);
ALTER TABLE SaleItem ADD CONSTRAINT fk_SaleItem_MarketRegion FOREIGN KEY (MarketRegionID) REFERENCES MarketRegion (MarketRegionID);

-- Drop if needed
-- ALTER TABLE SaleItem DROP CONSTRAINT fk_SaleItem_SaleDate;
-- ALTER TABLE SaleItem DROP CONSTRAINT fk_SaleItem_Customer;
-- ALTER TABLE SaleItem DROP CONSTRAINT fk_SaleItem_Product;
-- ALTER TABLE SaleItem DROP CONSTRAINT fk_SaleItem_MarketRegion;

```

# Query Results

## Qry 1

* Total sales by month for the last 4 years.

```sql
select d.salemonth, sum(s.sales)::money as total_sales
from saleitem s
join saledate d using (SaleDateID)
group by d.salemonth
order by d.salemonth
;
```

```
 salemonth |  total_sales  
-----------+---------------
         1 |   $775,766.91
         2 |   $722,853.17
         3 |   $951,333.08
         4 |   $851,617.32
         5 |   $976,415.68
         6 | $1,152,367.79
         7 |   $838,743.56
         8 | $1,247,500.81
         9 | $1,244,139.73
        10 | $1,120,777.47
        11 | $1,377,651.29
        12 | $1,383,335.11
(12 rows)
```

* Total sales by month for _each of_ the last 4 years.

```sql
select d.saleyear, d.salemonth, sum(s.sales)::money as total_sales
from saleitem s
join saledate d using (SaleDateID)
group by d.saleyear, d.salemonth
order by d.saleyear, d.salemonth
;
```

```
 saleyear | salemonth | total_sales 
----------+-----------+-------------
     2011 |         1 | $138,241.30
     2011 |         2 | $134,969.94
     2011 |         3 | $171,455.59
     2011 |         4 | $128,833.47
     2011 |         5 | $148,146.72
     2011 |         6 | $189,338.44
     2011 |         7 | $162,034.70
     2011 |         8 | $219,223.50
     2011 |         9 | $255,237.90
     2011 |        10 | $204,675.08
     2011 |        11 | $214,934.29
     2011 |        12 | $292,359.97
     2012 |         1 | $162,800.89
     2012 |         2 | $152,661.15
     2012 |         3 | $201,608.73
     2012 |         4 | $187,469.96
     2012 |         5 | $218,960.16
     2012 |         6 | $249,289.77
     2012 |         7 | $174,394.03
     2012 |         8 | $271,669.66
     2012 |         9 | $256,567.85
     2012 |        10 | $239,321.10
     2012 |        11 | $270,723.05
     2012 |        12 | $291,972.33
     2013 |         1 | $206,459.20
     2013 |         2 | $191,062.77
     2013 |         3 | $230,547.79
     2013 |         4 | $233,181.35
     2013 |         5 | $304,509.96
     2013 |         6 | $341,162.34
     2013 |         7 | $223,642.66
     2013 |         8 | $323,876.61
     2013 |         9 | $326,897.27
     2013 |        10 | $270,121.88
     2013 |        11 | $383,039.21
     2013 |        12 | $371,245.41
     2014 |         1 | $268,265.52
     2014 |         2 | $244,159.30
     2014 |         3 | $347,720.97
     2014 |         4 | $302,132.54
     2014 |         5 | $304,798.84
     2014 |         6 | $372,577.23
     2014 |         7 | $278,672.17
     2014 |         8 | $432,731.04
     2014 |         9 | $405,436.71
     2014 |        10 | $406,659.42
     2014 |        11 | $508,954.73
     2014 |        12 | $427,757.40
(48 rows)
```

## Qry 2

* Top 5 products sold during January period, along with Category and Sub-Categeory names
* _(I assume over all years)_

```sql
select p.productid, p.category, p.subcategory, p.productname, sum(s.sales)::money as total_sales
from saleitem s
join saledate d using (SaleDateID)
join product p using (ProductID)
where d.salemonth = 1
group by p.productid, p.category, p.subcategory, p.productname
order by total_sales desc
fetch first 5 rows only
;
```

```
    productid    |    category     | subcategory |                productname                 | total_sales 
-----------------+-----------------+-------------+--------------------------------------------+-------------
 FUR-CH-10000027 | Furniture       | Chairs      | SAFCO Executive Leather Armchair, Black    |   $6,426.00
 TEC-CO-10000013 | Technology      | Copiers     | Brother Fax Machine, Laser                 |   $5,733.72
 FUR-TA-10000184 | Furniture       | Tables      | Barricks Conference Table, Fully Assembled |   $5,451.30
 OFF-BI-10004995 | Office Supplies | Binders     | GBC DocuBind P400 Electric Binding System  |   $5,443.96
 TEC-PH-10004583 | Technology      | Phones      | Motorola Smart Phone, Cordless             |   $5,302.94
(5 rows)
```

## Qry 3

* Total quantity sales for Weekends by Region by Week
* _(I assume over all years)_

```sql
select d.SaleISOWeekNumber as Week, m.market, m.region, sum(s.sales)::money as weekend_total_sales
from saleitem s
join saledate d using (SaleDateID)
join marketregion m using (MarketRegionID)
where d.SaleISOWeekDay >= 6 -- 6=sat, 7=sun
group by d.SaleISOWeekNumber, m.market, m.region
order by d.SaleISOWeekNumber, m.market, m.region
;
```

* _Note: Results show the first 2 and last 1 week for brevity (859 rows total)_

```
 week | market |     region     | weekend_total_sales 
------+--------+----------------+---------------------
    1 | Africa | Africa         |           $2,605.38
    1 | APAC   | Central Asia   |          $10,792.05
    1 | APAC   | North Asia     |          $10,881.07
    1 | APAC   | Oceania        |           $3,617.90
    1 | APAC   | Southeast Asia |           $2,703.18
    1 | Canada | Canada         |             $346.59
    1 | EMEA   | EMEA           |           $2,873.52
    1 | EU     | Central        |          $10,059.96
    1 | EU     | North          |           $9,605.47
    1 | EU     | South          |             $808.41
    1 | LATAM  | Caribbean      |           $4,104.11
    1 | LATAM  | Central        |             $637.02
    1 | LATAM  | North          |             $438.78
    1 | LATAM  | South          |           $2,776.97
    1 | US     | Central        |             $605.16
    1 | US     | East           |           $2,218.90
    1 | US     | South          |           $4,318.13
    1 | US     | West           |           $1,519.48
    2 | Africa | Africa         |           $3,103.34
    2 | APAC   | Central Asia   |           $1,279.35
    2 | APAC   | Oceania        |             $403.65
    2 | APAC   | Southeast Asia |           $5,767.75
    2 | Canada | Canada         |           $1,561.47
    2 | EMEA   | EMEA           |           $1,227.31
    2 | EU     | Central        |           $3,811.73
    2 | EU     | North          |              $16.50
    2 | EU     | South          |              $58.98
    2 | LATAM  | Caribbean      |             $543.64
    2 | LATAM  | Central        |             $617.86
    2 | LATAM  | North          |             $817.60
    2 | LATAM  | South          |           $1,082.24
    2 | US     | Central        |           $1,556.22
    2 | US     | East           |           $4,171.63
    2 | US     | South          |           $2,597.58
    2 | US     | West           |              $25.83
   ...
   52 | Africa | Africa         |           $2,568.81
   52 | APAC   | Central Asia   |           $1,255.04
   52 | APAC   | North Asia     |             $997.68
   52 | APAC   | Oceania        |           $3,392.58
   52 | APAC   | Southeast Asia |           $2,256.80
   52 | EMEA   | EMEA           |           $1,771.31
   52 | EU     | Central        |           $5,353.85
   52 | EU     | North          |           $2,520.56
   52 | EU     | South          |             $213.73
   52 | LATAM  | Caribbean      |              $86.67
   52 | LATAM  | Central        |           $3,203.84
   52 | LATAM  | North          |           $1,294.89
   52 | LATAM  | South          |           $1,077.42
   52 | US     | Central        |           $1,817.78
   52 | US     | East           |           $4,894.86
   52 | US     | South          |               $2.61
   52 | US     | West           |           $4,038.11
(859 rows)
```

# Discussion of Issues/Disadvantages with this Dimensional Model

## Advantages

* This star schema is simple to understand, build, and use:
    * a single fact table represents sale information;
    * simple dimension tables are a single join away;
    * the schema answers many common business questions.

## Disadvantages

* For simplicity the join keys are reusing 'business' keys - query and storage performance may be improved using surrogate keys;
* The schema may be difficult to use to answer more complex and interesting business questions;
* The schema is not appropriate for OLTP applications;
* It would be interesting to compare cost, speed and usefulness with massive amounts of data over extended time periods in using different schemas and query technologies such as Google BigQuery and OLAP data warehouses.

## Additional Comments on Data

* Order IDs are not unique to a given customer and order time:
    * It seems to me that the business should require Order ID to be properly unique in this instance, maybe with some form of UUID or index-friendly time-sortable hybrid (eg SQUUID);
    * A new composite key has been added with Customer ID and Order Date to uniquely identify sales to a customer on a date in a market/region.
* Product IDs (4.4%) have conflicting names (and Sub-Category in one instance):
    * The names don't appear to be simply alternate equivalent products, nor data-entry typos corrected over time, nor different product names over time, but they generally seem to be similar products;
    * Therefore, a single Product Name is chosen arbitrarily for each ID.
* Market/region mostly independent of product & customer:
    * Assume customers can buy in different markets;
    * Assume products can be sold in different markets;
    * this is all observed in the data.
* I would prefer to use surrogate keys rather than business keys for joining but did not deem it worth the effort in this case.
* Some fact (and dimension) data is included above requirements for these queries, likewise some unneeded dimension data is excluded.
 