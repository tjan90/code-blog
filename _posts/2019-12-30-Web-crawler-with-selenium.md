---
layout: post
author: Tanveer jan
title: Web Crawler with Selenium
date: 2018-12-23
thumbnail: /assets/img/posts/we-scraper.jpeg
category: Python
summary: Scraping data from the websites
---
### Description
This app scrape the price of the item in e-commerce site i.e amazon. After checking the price it check if the price is reduced, will send an email every 24 hrs

### Installation
Clone the repository by executing this command in terminal/command

{% highlight console %}
git clone [url-here](https://github.com/tjan90/Web-Crawler-with-BeautifulSoup.git)
{% endhighlight %}

open the folder and install the required libraries
{% highlight console %}
cd Web-Crawler-with-BeautifulSoup
pip install requirements.txt
python main.py
{% endhighlight %}


### Built With
  - Python 3.8
  - BeautifulSoup4
  - SMTP(Gmail) - You can also get up seprate app password for gmail.

### Usage
Modify the credentials with you own. and
  - Email & Password
  - URL for required Item
  - Previous price( As this is static so you can further develop the app by using storage to store the previous value.)


### Further Developement
I will be uploading another project that will be developed with selenium, which crawl through website and save the data. It will be and extension to this app. 