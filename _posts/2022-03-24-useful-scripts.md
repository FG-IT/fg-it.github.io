---
title: Useful Scripts/Code Snippets for Spree Based System 
date: 2022-03-24 14:49:00
author: YZ
categories:
- spree
tags:
- sql
---

## Export All The Variants With No Barcode
run the following `SQL` in a SQL viewer software and export the results.

```SQL
SELECT p.id AS product_id, 
	v.id AS variant_id, 
    concat('https://everymarket.com/admin/products/', p.slug) AS admin_link,
    p.name, v.sku, v.barcode
FROM spree_variants AS v
JOIN spree_products AS p
	ON p.id = v.product_id
WHERE p.deleted_at IS NULL AND v.barcode IS NULL
ORDER BY p.id;
```
