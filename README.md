Based on theme [NexT](https://github.com/Simpleyyt/jekyll-theme-next)

`Pull`, `Edit` and `Push`, visit at [https://fg-it.github.io](https://fg-it.github.io/). That's it.
## Contribute To Resources
Hi, Resources tab is added to this repo. Basically, we want to collect as many useful resources as possible. Currently there is only one collection - resources about startup. 

Please feel free to contribute if you find any interesting and useful resources. It can be a website, blog, online courses, or even a single article. 

Please open an issue with the link and a simple description on what it is if you find one. I'll organize it and add to the page. 
## Author Setup
### add your avatar
add your avatar to folder `/assets/images` 
### add a markdown file in `_authors` directory
the file name should be `[your short name].md`
```yaml
---
short_name: [your short name]
name: [your name]
description: [your description] 
avatar: [your avartar placed in /assets/images folder, eg /assets/images/yz.jpg] 
github: [your github link]
---
```

## Write Posts
add a file to `_posts` directory with the following format:
```
YEAR-MONTH-DAY-title.md
```
Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and MARKUP is the file extension representing the format used in the file. For example, the following are examples of valid post filenames:
```
2011-12-31-new-years-eve-is-awesome.md
2012-09-12-how-to-write-a-blog.md
```

The post files must begin with [front matter](https://jekyllrb.com/docs/front-matter/) which is typically used to set a layout or other meta data. At least, the following meta data should be included, by default, all the posts are using `post` layout:
```yaml
---
title: 
date: [yyyy-mm-dd h:m:s]
author: [your short name, NOT name]
categories: [yaml list]
tags: [yaml list]
---
```

Please check the exsiting categories and tags before you add a new one. If there is no sutable one you can use, just add a new one. They will automatically added to categories or tags page.

**Happy writing!  😎**