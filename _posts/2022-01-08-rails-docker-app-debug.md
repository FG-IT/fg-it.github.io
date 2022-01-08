---
title: Debug Rails App Using Gem pry-rails in Docker Enviroment (docker-compose)
date: 2022-01-08 15:30:09
author: YZ
categories:
- Debug
tags:
- rails
- pry-rails
- docker
- docker-compose
- debug
---

This article is aiming to provide a debug setup using [pry-rails](https://github.com/pry/pry-rails) on Rails app running in a docker enviroment. 

## install pry-rails
Add this line to your gemfile:
```ruby
  gem 'pry-rails', :group => :development
```

## docker-compose.yml config
add the following 2 options to the service serve rails application to enable interactive mode
```yaml
tty: true
stdin_open: true
```

## add breakpoint
add `binding.pry` to any place you want the program to stop.

## get interactive shell
run `docker ps` to get the rails container id, then run `docker attach [container id]`. Now you are intercepting the breakpoint. 

please refer [pry's documentation](https://github.com/pry/pry/wiki) on usage.
