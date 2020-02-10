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


# Comments on data

* Order IDs are not unique to a given customer and order time
** A new composite key is built with Customer ID and Order Date to uniquely identify sales to a customer on a date in a market/region
* Product IDs (4.4%) have conflicting names (and Sub-Category in one instance)
** One Product Name is chosen arbitrarily
* Assume customers can buy in different markets/regions
* Assume products can be sold in different markets/regions
* Would prefer to use surrogate keys rather than business keys
