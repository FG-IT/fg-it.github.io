---
title: Spree Product Image Handle
date: 2022-02-24 10:30:00
author: YZ
categories:
- spree
tags:
- image processing
- active storage
---

## Upload and Variation Generation
Use whatever Active Storage provides.

## Serving Model
### Assumption
1. all the images should be publicly accessible. _If you need to control how and who can access it, maybe active storage is better._
2. Use a single storage backend to store all the images.

### Goal
serve the page view request as fast as possible, which means generate all image link as fast as possible. Not tons of sql queries, not check if the image exists or not. 

### Strategy
- database records variation information, which is generated,which is not.
- if the variation is generated, which means it exists in our storage, return the serve link directly(either backend storage link or cdn). 
- if the variation is NOT generated, return a link to generate it. 

*Note: it makes no assumption on whether the variation is preprocessed or not, preprocess is highly recommended if the computation resource is allowed.*

### Serving Link
For any ActiveStorage::Blob instance, we can use its `key` attribute to get the serving link by calling `model.url` or `ActiveStorage::Blob.service.url(key)` . So, if we have `key`, we can easily generate the serving url.
1. for existing images, simply use `ActiveStorage::Blob.service.url(key)`. If there is a cdn in front of the storage backend, use something like `cdn_url(key)`
2. for non exsiting images, we can compose the key using original `blob key` and `transformation`(*refer key generation part*), and make a route looks somthing like `/rails/images/variation/:key` to handle variation generation.

### Variation Key Generation
`[blob key]_[image style name(small, large. etc)].[original blob file extension]`
When generate variations, a set of `transformations` are applied. There are a few parameters such as `size`, `quality`, `gravity` and so on. We assume these parameters are fixed and not supposed to change for each styles. So, we simply use `style name` to identify which `transformation` is supposed to be applied.
You are free to add/delete styles, but not change parameter of existing styles.


## Controller
1. get `variation key` from params
2. get blob id and image style name from `variation key`
3. generate variation
4. mark this variation as generated
5. respond with the variation

## Database/Model Attributes Change
add two fields to `spree_assets`: `blob_key` and `variations`. `variations` field record which image styles are processed. Initially, its `nil` or `[]`. No need to add any additional index.

It will make image link generation fast, no need to look up `active_storage_attachments` and `active_storage_blobs`

When a blob is created, add `blob_key` to its associated `spree_assets` record. When a variation is processes, add the style name to `variations` attribute.

## Model
provide utilities to preprocess all variations, by default preprocess is disabled.

## System Requirement
- (**required**) at least 1 server serving page requests.
- (**required**) cloud storage
- (optional but highly recommended) cdn in front of cloud storage
- (optional but highly recommended) at least 1 server to generate variations. (if no, you possibly will share the same server/cluster to serve normal application requests and resize images)

## Restriction
If you change image styles parameters, for example, the `small` image style has a size `100x100`, if you change it to `120x120`, the current system will not notice that. You may need to re-run variation process script.