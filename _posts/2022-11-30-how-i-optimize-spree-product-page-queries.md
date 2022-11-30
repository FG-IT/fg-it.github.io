---
title: How I Optimize Mysql Queries on Spree Product Detail Page
date: 2022-11-30 18:06:00
author: YZ
categories:
- spree
tags:
- spree
- performance
- product page
- mysql 8
- json
---

# The Problem
Spree product detail page runs lots(a few dozen even more than one hundred depends on product's properties) of database queries. Lots of tables are invloved(Appendix). Each query is excuted quite fast in database engine, but travels through network takes lots of time. Let's assume there are 80 queries, it takes 1ms for each query travel through the fast google cloud's inner network. Even if we omit the time mysql takes to excute those queries, it would take 80ms for data fetching. 

# The Bottleneck
The internet is fast. The database is also fast. However, too much queries taking too much time travele on the network.

# Proposal
Of course, we need to reduce the queries. Can we fetch the same data with significantly less(1 is optimal) queries? After researching the posible solutions of mysql(we already use mysql), the following proposal seems work.

* using a database view to put all the data in
* using JSON supports in mysql 8 (it's quite easy to upgrade mysql from current version 5.6 to 8.0)
* using `join` and `sub queries` to gather data from multiple table.


# My Solution
_note: my solution only works on mysql 8.0 and higer_

This is the [query](https://github.com/FG-IT/spree-representable/blob/153-plan/app/models/spree_representable/product_representation_query.rb) to create the view.

Here, I just use a virtual view instead of a materialized view. A materialized view would be faster but it seems more difficult to refresh it when data changes. 

One thing to mention is that every time the query to create the view changes, a new database migration should be generated and ran to update the view in mysql.

# Result
The query to fetch product related information reduced to exactly 1. Through google cloud query insights, the avg execution time is about 4.5ms. There are 412,847 products. 

# Potential Problem
Not sure if it still works good if there are more products records in the database. If it takes linear longer time as the products increase, it will be good enough. If when products amount exceed a threshold, it takes significant longer, maybe this solution is not feasible.

Because the way to fetch data is changed, lots of other works may be done to make product page show the information.

# Appendix
## tables involved in product detail page
* spree_products
* spree_variants
* spree_assets
* active_storage_attachments
* active_storage_blobs
* spree_shipping_categories
* spree_shipping_methods
* spree_shipping_method_zones
* spree_countries
* spree_prices
* spree_option_values
* spree_option_value_variants
* spree_stock_items
* spree_stock_locations
* spree_taxons
* spree_products_taxons
* spree_product_properties
* spree_properties
* spree_option_types
* spree_product_option_types