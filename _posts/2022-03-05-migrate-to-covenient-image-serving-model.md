---
title: Migrate to A More Covenient Image Serving Way for Spree
date: 2022-03-05 11:50:00
author: YZ
categories:
- spree
tags:
- image processing
- active storage
- image resize
- google cloud
---

## Deploy Image Variant Processing Service
1. checkout the [repo](https://github.com/FG-IT/image_processing), please read `readme.md`
2. config `gcloud` command line env.
3. run `gcloud run deploy`, proceed by answering all the prompted questions. *note: when select deploy region, select one closer to your cloud storage location.*

You may also need to tune the configurations of `cloud run`. The following is my suggestion, you may need a different config set. 
* CPU: 500m, only allocated during request processing
* Memory: 256MiB
* Maximum request per container: 1
* Execution: First Generation (faster cold start)
* Autoscaling: minimum instances: 2

Please note than in order to avoid the chance of cold start, we need leave some instances idle for fast response. idle instances charge full price for memory, but only 13% for cpu. If you just deployed the service, I suggest to set a large quantity of minimum instances for a few days. Because lots of image variants requests will hit the service directly. 

### Environment Variables
In order to make the service run properly, you need to add google cloud storage related environment variables.
#### Non-Google Cloud Environment
|   ENV name      | value        | 
|---------------  |--------------|
|GCS_PROJECT_ID|  |gc project id |
|GCS_CLIENT_EMAIL |refer service account|
|GCS_PRIVATE_KEY  |refer service account|
|GCS_BUCKET       |storage bucket name|

#### Google Cloud Environment
please make sure the bucket and the service you're going to deploy are under the same project.
|   ENV name      | value        | 
|---------------  |--------------|
|GC_DEPLOY        | true         |
|GCS_BUCKET       |storage bucket name|

## Setup Cloud CDN
Follow the [instruction](https://cloud.google.com/cdn/docs/setting-up-cdn-with-serverless) to set up a cdn with the cloud run service we just deployed as the backend.

You may find it useful to [add some custom headers to the response of the backend service](https://cloud.google.com/load-balancing/docs/https/custom-headers), such as `cdn_cache_id`, `cdn_cache_status`. 

## Update Spree
### How it works
Basically, a field called `attached_file` is added to `spree_assets`. Every time when a `Spree::Image` is created, its `attachment_blob.key` will be saved to `attached_file`. This key is also the file name in storage. So, we use the key to generate a url which can be used directly find the original file. Then, use our own image processing service to serve images with different sizes and formats. 

### Gem Upgrade
update spree to tag `ev3.0.0`
`gem 'spree', github: 'FG-IT/spree', tag: 'ev3.0.0'`

you'll also need to copy & run migrations.

### Environment Variable
You must set `IMAGE_RESIZING_URL` environment variable. It's the endpoint of your cdn. eg `https://img.everymarket.com`.

### Utility Provided
in [variation_service.rb](https://github.com/FG-IT/spree/blob/ev3.0.0/core%2Fapp%2Fmodels%2Fspree%2Fimage%2Fconfiguration%2Fvariation_service.rb) file, a few method are defined for `Spree::Image`.
1. `public_url(style, format='jpg')`, params: `style` is the name of image styles, such as `small`, `product`, `large`. `format` is either `jpg` or `webp`
2. `#{style_name}_image_url(format='jpg')`, such as `small_image_url`, `product_image_url`. Param `format` is either `jpg` or `webp`.
   
I'd like to suggest you override image styles instead of renaming. Two reasons: 
1. Keep all the naming consistent. All the extensions will solely depend on `spree` to retrive link of image url.
2. make use of the methods pre-defined by `define_method`. If you add new style names, for example `list: [300, 300]`, the method `list_image_url` is not defined automatically.

you can do it like the following:
```ruby
module Everymarket
  module Spree
    module ImageDecorator
      extend ActiveSupport::Concern

      prepended do 
        styles.merge!(
          {
            mini: [48, 48],
            small: [100, 100],
            product: [270, 270],
            large: [650, 650],
            zoomed: [1044, 1044]
          }
        )
      end
    end
  end
end
```
In terms of the value of each style, array and string are supported. eg `[48, 48]` or `'48x48>'`

## Pre Deployment
1. Tag your current stable project, just in case anything wrong happens you can rollback easily and quickly.
2. please run the following code from console to pre-fill `attached_file` attribute of `Spree::Image`
```ruby
   Spree::Image.where(attached_file: nil).includes(:attachment_blob).find_each do |image|
     attached_file = image.attachment_blob.key
     image.update_column(:attached_file, attached_file)
   end
```
3. make sure you add all the enviroment variables.

## After Deployment
1. Clear website cache. You can do it from `admin` or run related code via `rails console`
2. You may need to re-run the code to pre-fill `attached_file` again.
3. If you have sidekiq job running in a different workflow, please also remember update it with the corrent docker image.