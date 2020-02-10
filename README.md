# Data Engineering - Superstore Dataset

* Michael Pheasant
* See exploratory data analysis in`data-eng.ipynb` ipython notebook

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

## SaleItem Table:
    SaleItemID (RowID)
    CustomerID
    ProductID
    SaleDateID
    MarketRegionID
    Sales
    Quantity
    Discount
    Profit
    Shipping Cost

## SaleDate Table:
    SaleDateID
    SaleDate
    SaleYear
    SaleMonth
    SaleISOWeekNumber
    SaleISOWeekDay

## Customer Table:
    CustomerID
    Segment
    Customer Name

## Product Table:
    ProductID
    Category
    Sub-Category
    Product Name

## MarketRegion Table:
    MarketRegionID
    Market
    Region

# Comments on data

* Order IDs are not unique to a given customer and order time
** A new composite key is built with Customer ID and Order Date to uniquely identify sales to a customer on a date in a market/region
* Product IDs (4.4%) have conflicting names (and Sub-Category in one instance)
** One Product Name is chosen arbitrarily
* Assume customers can buy in different markets/regions
* Assume products can be sold in different markets/regions
* Would prefer to use surrogate keys rather than business keys
