---
layout: post
title:  "Capital Guessing Game 🌎"
date:   2023-05-05 22:01:010 +0200
categories: blog
---

I had some fun this weekend creating a capital guessing game. Try it out at [https://www.capitalguesser.com](https://www.capitalguesser.com)
<!--more-->

## Functionality

The website is really simple: the user is prompted with a country and has to answer the name of the capital. If the guess is correct, they move on to the next question. On the other hand, if the guess is incorrect, they have three options: try again, reveal the answer, or move on to the next question. The page shows the current score for the session (this will set back to 0 if the page is refreshed).

I made this as a simple tool to make a presentation about how to deploy software to production.

## Tech stack

It's built using a simple Flask application, served by uwsgi behind nginx. It uses JS and CSS, there really isn't anything fancy to it.

It's all hosted on AWS, running on a t4g.small ARM instance. I wanted to stay within the free tier. After the first 12 months, the running cost will be $0.0212 USD per hour or roughly $186 per year (the instance is in ap-southeast-2). If utilisation is low I might also move to a t4g.micro making it $92 per year. There currently isn't any load balancer (this might change if the number of concurrent users gets higher)!

I bought the domain name of namecheap for $9 USD.

The TLS certificate was obtained using Let's Encrypt.

### Design

I'm not really a front-end person (this might be obvious if you've seen the website...) but I followed some simple design principles, in no particular order:
* avoid using pure white or black since it tends to increase reading fatigue
* find a color palette and stick to it - I love [https://coolors.co](https://coolors.co) for this
* try contrast the text with the background. In case you really have to have bright text on a bright background, style the text with a border to make it readable e.g. `-webkit-text-stroke: 0.5px #141414`


## Continuous Integration

The repository is hosted on Github (private repo) and the CI is done using a workflow.

## Monitoring

I'm using [uptime robot](https://uptimerobot.com/) to check if the website is up. it's simple and free


## Contact

Let me know if you enjoyed the website, feel free to reach out by mail at the address on my homepage for any feedback! If this project gets attention, I'll consider making the repository public