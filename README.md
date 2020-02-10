# Data Engineering - Superstore Dataset

* Michael Pheasant
* See [exploratory data analysis](data-eng.ipynb)
 ipython notebook

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

# Comments on data

* Order IDs are not unique to a given customer and order time
** A new composite key is built with Customer ID and Order Date to uniquely identify sales to a customer on a date in a market/region
* Product IDs (4.4%) have conflicting names (and Sub-Category in one instance)
** One Product Name is chosen arbitrarily
* Assume customers can buy in different markets/regions
* Assume products can be sold in different markets/regions
* Would prefer to use surrogate keys rather than business keys
