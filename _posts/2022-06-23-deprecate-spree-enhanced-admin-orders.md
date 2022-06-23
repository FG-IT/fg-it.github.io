---
title: Steps to Deprecate spree_enhanced_admin_order Gem
date: 2022-06-23 15:06:00
author: YZ
categories:
- spree
tags:
- spree
- spree core
- spree_enhanced_admin_order
---

# Why Deprecate It
`spree_enhanced_admin_order` is a spree extension to add functionality to spree admin. Nearly all the projects use this gem by default. It has been a part of the `spree admin` module. Keep it seperate makes code hard to debug and introduce additional unnecessary dependency, especially for those extensions which rely on this extension.

# Steps to Deprecate it
1. remove `gem spree_enhanced_admin_order` from your `Gemfile`
2. remove `spree/backend/custom_shipment` js file if you `required` it in any asset pipeline files. For example `vendor/assets/javascripts/spree/backend/all.js` 
3. remove `spree/backend/spree_custom_admin` css file if you `required` it in any asset pipeline files. For example `vendor/assets/stylesheets/spree/backend/all.css`
4. remove `[date]_add_carrier_to_spree_shipments.spree_enhanced_admin_order.rb` from your project `db/migrate` folder, or rename it to `[date]_add_carrier_to_spree_shipments.spree.rb`
5. upgrade `spree` to tag `>=` `ev3.2.0`, run `bundle exec rake spree:install:migrations` and `bundle exec rake db:migrate`