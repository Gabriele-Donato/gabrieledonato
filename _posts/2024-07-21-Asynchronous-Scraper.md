---
layout: default
title:  "Coding an Asynchronous Sitemap Generator (With Full Code on Github)"
date:   2024-07-21 10:00:00 +0200
categories: jekyll update
excerpt: ""
image: /assets/images/rambosson.jpg
---

## writing in progess...
## this is a draft 

This article explores the creation of an asynchronous sitemap generator in Python. The code for this project can be found [here](https://github.com/Gabriele-Donato/website-materials-/blob/Scraping/Asynchronous_Sitemap_Generator/Asynchronous_Sitemap_Generator.ipynb).
However, before introducing the tools that will be used, I would like to give a sense of the utility of what has been developed. The first
and foremost aim of my work was experimenting with the _asyncio_ library of Python, I like this library because it makes non-linear
programming accessible. 

I reserve a more thorough account of the concurrency/parallel programming tools offered by Python to a future article (or to updates of this article), for the moment I will limit the discussion to an applied project.
The reason why I chose this strategy is because I believe that asynchronous
programming is far from intuitive and can be better understood by applying it to a specific case. Nonetheless, the interested reader should be aware that things are not necessarily as straightforward as they migt seem, and that 
a more complete account should not avoid the question of _why is asynchronous programming better that multithreading or multiprocessing in the present case?_, and ultimately of _what are asyncio, multithreading and multiprocessing about?_
The answers to these questions demonstrate that the coding approach chosen here is optimal in terms of the resources used.
\
\

# Why this is a boring interesting project
\
A "sitemap" is the representation of the structure of a website: a list of unique links starting from a given page (usually the homepage) and proceeding to increasing depths within the structure of the website.
The depth of a sitemap can be chosen 
arbitrarily, and ideally should include all of the pages of a website. 
However, a sitemap is also a tool useful for website optimization, and therefore it should include primarily the pages that are relevant 
to the issue at hand. For instance, in a shopping website we may be interested in how difficult it is for the user to get from the homepage to the cart, 
or whether it is more difficult to reach man-related product than woman-related ones: in
these cases the depth of those pages with respect to the homepage may be an information of crucial value.
\
A good sitemap is also a guide for browsers through a what is known as _priority_: a number that helps indexing the pages depending on the importance given to them by the websmaster. Other useful indications may include the date
of last modification, and the number of visits that a certain page has received. 
\
Although it may be an element of particular importance, websites may or may not have sitemaps. If the sitemap is not available, and the information is needed,
scraping is the way to go. However, if the sitemap is available, it should be downloaded from the website since it surely lists what has been deemed important by
the webmaster. 
\
What is fascinating about sitemaps, is that, when scraping is involved, they are almost the equivalent of a geographic-map: if you know precisely how to go from 
one place to another, you don't have to meander or risk getting lost. In the case of sitemaps, the story is a little bit different, but not that much: since it is 
a list of links, the scraper can teleport itself from a page to another. However, this does not imply that such a program will also be efficient since websites take their 
time to load, and the bandwidth may be scarce. Moreover, does it really make sense exploring blindly a whole website? In most of the cases the answer is no.[^1] For this
reason it is common practice to check first if any API does exist for the info needed.
\
Nowadays, nobody has to build a sitemap generator, or even a custom scraper: this is fun and instructive, but frameworks like _scrapy_ make the process immediately scalable and 
professional. Nonetheless, _scrapy_ itself does not work with magic (even though it might look like so given the impressive performances), but using asynchronous programming. 
The latter will be the core element of the ensuing discussion. Therefore, even if the project is not as exciting as making [a news retriever with real time processing](), it 
is the place to start. Let's put our hands to work!
\

# The structure of a Sitemap
\
As explained above, sitemaps are a serious thing and a does exist: check [sitemap.com](https://www.sitemaps.org/protocol.html). The following table taken from the website summarizes
the main tag (note that even when the sitemap is present, most are optional):
\
<img alt='sitemap_tags_table' src = "{{ '/assets/images/post_images/sitemap_project/sitemap_tags_table.png' | relative_url}}">
\
A sitemap may have at most 50,000 urls and be no more of 50MB, but it is possible to join multiple sitemaps by specifying an index through the <sitemapindex> tag.
Moreover, special characters should be escaped and the encoding should be UTF-8. Generally, sitemaps are stored in XML files, and they can be validated through
schemas (e.g. [](http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd)).
\
These problems become relevant when storing the scraped data, therefore will appear again only at the end.
\

# S
\



[^1]: For instance, once I scraped the European Parliament website for retrieving the contributions of the partisans during plenary sessions. If I had to scrape the whole website I would never have accomplished the task.
