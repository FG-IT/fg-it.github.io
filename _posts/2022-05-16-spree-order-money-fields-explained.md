---
title: How Are Number Attributes of Spree Order Calculated?
date: 2022-05-16 14:27:00
author: YZ
categories:
- spree
tags:
- spree
- spree core
- spree order
---

In spree order model, there are a few number attributes. This article is a summary on what there are, how there are calculated, and the relationship between other associations.

# What the Attributes Are
From [spree official documentation](https://dev-docs.spreecommerce.org/internals/orders), there are a few number attributes:
- **item_total**: The sum of all the line items for this order.
- **adjustment_total**: The sum of all adjustments on this order.
- **total**: The result of the sum of the item_total and the adjustment_total
- **payment_total**: The total value of all finalized payments.
- **shipment_total**: The total value of all shipments' costs.
- **additional_tax_total**: The sum of all shipments' and line items' additional_tax.
- **included_tax_total**: The sum of all shipments' and line items' included_tax.
- **promo_total**: The sum of all shipments', line items' and promotions' promo_total.
- **item_count**: The total value of line items' quantity.

# How the Attibutes Are Calculated
let's assume there is an order instance variable named `order`, then the following comparisons stand (*change `==` to `=`, that's exactly how they are calculated*):
- `order.item_count == order.line_items.sum(:quantity)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order.rb#L587)
- `order.item_total == order.line_items.sum('price * quantity')`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L100) 
- `order.payment_total == order.payments.completed.includes(:refunds).inject(0) { |sum, payment| sum + payment.amount - payment.refunds.sum(:amount) }`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L68)
- `order.shipment_total == order.shipments.sum(:cost)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L72)
- `order.adjustment_total == order.line_items.sum(:adjustment_total) + order.shipments.sum(:adjustment_total) + order.adjustments.eligible.sum(:amount)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L82)
- `order.included_tax_total == order.line_items.sum(:included_tax_total) + order.shipments.sum(:included_tax_total)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L85)
- `order.additional_tax_total == order.line_items.sum(:additional_tax_total) + order.shipments.sum(:additional_tax_total)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L86)
- `order.promo_total == order.line_items.sum(:promo_total) + order.shipments.sum(:promo_total) + order.adjustments.promotion.eligible.sum(:amount)`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L88)
- `order.total == order.item_total + order.shipment_total + order.adjustment_total`, [source code](https://github.com/FG-IT/spree/blob/v4.2.5/core/app/models/spree/order_updater.rb#L77)
