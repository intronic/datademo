

Retail dataset of a global superstore for 4 years.



# Dimensions of interest

Sales, broken down by: 
* time: month, year, week, weekday
* region
* product, category, sub-category

# Columns

Product >-< Order >- Customer


Row ID
Order ID
Order Date
Ship Date
Ship Mode
Sales
Quantity
Discount
Profit
Shipping Cost
Order Priority

Customer ID
Customer Name
City
State
Country
Postal Code
Segment
Market
Region

Product ID
Category
Sub-Category
Product Name


# Queries

## Qry 1

Total sales by month for the last 4 years.

Dimensions: (sales, month, year)

## Qry 2

Top 5 products sold during January period, along with category and sub-category names.

Dimensions: (product, category, sub-category, sales, month)


## Qry 3

Total quantity sales for weekends by region by week.

Dimensions: (region, week, weekday, sales)

