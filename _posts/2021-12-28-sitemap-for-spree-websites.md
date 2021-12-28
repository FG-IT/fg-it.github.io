---
title: Sitemap setup for Spree Websites
date: 2021-12-28 10:30:09
author: YZ
categories:
- Spree
tags:
- spree
- e-commerce
- sitemap
- spree-extension
---

Basically we are using [spree_sitemap](https://github.com/spree-contrib/spree_sitemap). However, because of the distributed nature of the cloud, we need a few extra configurations.

This post applies to the technology stack: `active job`, `sidekiq`, `sidekiq-cron`, and `cloud storage`. 

The basic idea is: we are using sidekiq to execute active jobs asynchronously, if sidekiq server is different from website server, we need to make all the sitemap files available at links like `https://website.com/sitemap.xml.gz`. So, we need to upload the generated sitemap files to cloud storage, and setup a redirect when the sitemap link is visited.

## Installation
```ruby
gem 'spree_sitemap', github: 'spree-contrib/spree_sitemap'
```
please refer to the detailed installation guide of [spree_sitemap](https://github.com/spree-contrib/spree_sitemap)

## Make Rake Task `sitemap:refresh` a Active Job
```ruby
require "rake"
class RefreshSitemapJob < ApplicationJob
  queue_as :default
  def perform(*args)
    Everymarket::Application.load_tasks # first load all the rake tasks, change application name accordingly
    Rake::Task['sitemap:refresh'].reenable # important! it makes sure the task is able to run everytime we run this job.
    Rake::Task['sitemap:refresh'].invoke
    Dir.glob(Rails.root.join('public', 'sitemap*')).each do |filepath| # upload sitemap to cloud storage
      service = ActiveStorage::Blob.service
      file = File.open(filepath)
      key = File.basename(filepath)
      service.upload(key, file)
    end
  end
end
```

## Add the job to `sidekiq-cron.rb`
```ruby
Sidekiq::Cron::Job.create(
  name: 'update sitemap everyday at 00:30',
  cron: '30 0 * * *',
  queue: 'first_priority',
  class: 'RefreshSitemapJob'
)
```

## Route config
Add the following line to `route.rb`. 
```ruby
get '/:sitemap', 
  to: redirect(ActiveStorage::Blob.service.url('%{sitemap}'), status: 301),
  constraints: {sitemap: /sitemap\d*\.xml\.gz/}
```

