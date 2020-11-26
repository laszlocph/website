---
layout: default
title: "2020: consulting, products, finances - a year in review"
image: images/revenue-cropped.png
link: /2020-consulting-products-finances-a-year-in-review
permalink: /2020-consulting-products-finances-a-year-in-review
excerpt: 2020 has been a transition year in my consulting business. In this post I make an inventory of all software assets I have control of,then I look at the costs of independent product development by comparing my revenue to previous years

---

## 2020: consulting, products, finances - a year in review

2020 has been a transition year in my consulting business. The year in which I dialed down trading hours for money, and started building products.

In this post I make an inventory of all software assets I have control of, their story and future potential. Then I look at the costs of independent product development by comparing my revenue to previous years.

### So what products do I have?

#### Woodpecker

[Woodpecker](https://github.com/laszlocph/woodpecker/){:target="_blank"} is an open-source CI engine. It is a fork of the famed Drone.io project, just before its version 1.0 release.

Drone 1.0 was a major change from 0.8. Large part of it has been rewritten, it got a new look, new features, and new licensing terms. I forked Drone 0.8 in April 2019, when Drone's licensing was changing rapidly, and not in favor of scaled up open usage.

Woodpecker is dear to my heart, even though I don't consider it strategic at this point. It was a driver to level up my Golang and React skills, and me getting into the vendor side of the software business. I felt empowered by its source code, discovered Golang's simplicity and it was a good reason to pick up frontend chops.

Woodpecker seems to have a small niche, and gets a Github star a week. It is almost at a hundred stars, and getting Pull Requests by a handful of people. I still think that Woodpecker is a fully capable CI engine and it will be alive for years to come.

However, I find the CI space quite uninspiring. Even its latest contender, Github Actions just builds on the status quo, and in many aspects made the status quo stink even more. But Woodpecker could be some point a starting point if I decide to enter the space again.

I think about it from time to time. Especially about adding conventions to CI to make it less of a time sink ([read my ramblings about CI in this post from earlier this year](/ci-is-a-timesink-and-it-is-our-fault){:target="_blank"}).

#### 1clickinfra.com

[1clickinfra.com](https://1clickinfra.com){:target="_blank"} is my latest addition. A freemium product to manage Kubernetes cluster add-ons, as even with managed clusters it is a considerable effort to make Kubernetes an application platform.

1-Click Infra configures logging, metrics, HTTP routing, security and also gives you a git based workflow to keep the components up to date.

I launched it late august this year, and gets around 150-200 unique visitors a month and have 15 signups. I constantly add more components to it and I actively promote it on meetups. So far it is getting interest and I'm hopeful to crank up the usage in the coming months once word of mouth picks up and the product is getting into the no-brainer value add category.

It nicely augments my consulting business, as the artifacts it gives to users is identical to the work I deliver to my customers in the first weeks of our engagement. I'm hoping to strengthen my consulting offering by being able to provide this head start, and also sell enterprise packages to companies who enjoyed 1-Click Infra in their test environments.

#### OneChart.dev

[Onechart.dev](https://onechart.dev){:target="_blank"} is a small piece in the machinery. I wouldn't even call it a product. But so much of my work is building on it that I wanted to include it here.

It is great for prototyping, demoing and even for devs who don't care much about Kubernetes but are forced to deploy on it.

It embodies my thinking around Kubernetes tooling: make it simple to get started but also composable to stair-step your way up to production usage.

I really do hope it gets picked up once the Gimlet universe around it unfolds.

#### Gimlet

[Gimlet](https://gimlet.io/){:target="_blank"} is my largest product. It is an internal developer platform that covers Kubernetes deployment and rollback automation, and an application focused UI. All that based on gitops.

Currently it is installed at a single company serving 10 developers and all their deployment needs. It is generally well received and also proven to be on good technical foundation.

The product direction was also proven over the past year. Gimlet appeared on paper November 2019, released in April 2020, and in the meantime several industry players moved in a similar direction. Rancher's Fleet has the same  gitops approach to manage environments, Google's Application Manager is similar in managing applications. And GitOps got mainstream this year too.

While I feel validated by my users, and there is an industry trend I can  piggyback on, I have yet to gain traction on my inbound lead channels. I had zero inbound requests out of thousand unique website visitors.

I also had minimal success in my direct sales efforts. I managed to run a few trials over the summer, but none of them were really ready to dedicate the time for a proper evaluation. In general I also felt shitty during my sales efforts. I knew I had a great product, but didn't find a way to convey that in an easy message that runs home with my prospects.

But Gimlet is not going away, on the contrary. I'm doubling down on it in 2021, with an opensource packaging.

But first a bit of finances, as having this many bets on the table doesn't come for free.

### Finances
Talking money on the internet usually stirs up some emotions. This is not my goal. I'd like to show part of my finances to demonstrate the cost of product development - at least in my part of the world. I live in Hungary and have been living in Denmark earlier in my consulting career, so given that context a 100.000EUR a year revenue for my solo consulting company gets me a good living.

I made
 - 91K in 2017
 - 114K in 2018
 - then 91K again in 2019
 - My projected revenue for 2020 is 20K

That means working on products has its toll on my revenue while it hasn't contributed to it yet.

I started hacking on Woodpecker in April 2019 and on the following chart you can spot the trend that I started holding back my billed hours little after that.

Consulting hours billed, yearly revenue - click for details:

<a href="/images/revenue.png" target="_blank"><img src="images/revenue-cropped.png" alt="Revenue"/></a>

You can calculate the unrealized revenue and put a price tag on my product work, and while I certainly did that myself, I think there are two things to add.

The first is on the chart: the steady level of monthly income in 2020. While I wanted to cut all client work this year, I eventually engaged with a couple of previous clients. Looking back it was a good decision as it gives me some confidence now in late 2020: I'm without any product revenue, but I have a seemingly good command of my income. I can put my revenue / free time balance anywhere on this chart.

The second one is not part of this chart, but worth knowing. I am on this path with a large amount of savings. Larger than the 6-9 months cushion I had when I started freelancing back in 2016. The psyche of this path is worth another post, as it took me quite a while to become this entrepreneurial - while I'm not really that at all, not even today, as I carry so little risk.

### Product and consulting vision for 2021

Alright so after spending all this money, how can I really transition to product revenue? Well, perhaps I won't, or not in a way that I initially dreamed of.

It is certainly a longer path than just writing software and selling 400USD a month licenses, like I planned with Gimlet. It may happen, but I'm actually looking to leverage a little bit of consulting in 2021.

With Gimlet I found it difficult to sell expensive monthly subscriptions, with consulting however I managed to sell many times of that amount, while essentially developing Gimlet in-house for my clients. Or actually much worse, Gimlet implemented in CI, with glue.

Irony aside, I understand that it can be scary to buy something this expensive from someone you don't know, especially if you can't talk to him on your daily scrum. So I try to mix the two approaches a bit.

Let my 2020 be the model for my 2021 sales efforts: with my products, the Gimlet universe, I can provide much of the Kubernetes tooling and deployment needs of B2B SaaS companies up to fifty devs, all on day one.

I will also support it throughout the year, and will cover my clients ad-hoc cloud consultancy needs. Meaning I will be available on their daily scrum.

With this approach my clients get more value for less money, and they will have the safety net that I used to provide as a full-time devops consultant. I will also have more revenue for 2021, and by the end of the year my software assets will mature even more. Perhaps even allowing low touch product sales over the internet.

Onwards to 2021!

ps.: If you think my path is interesting, please like the social media post that led you to this blog entry. It will help my 2021 sales, and opensource product development.