---
title: My experimentation with passive income
date: 2016-03-20 21:50:30
tags:
---

Back in the spring of '12 I wanted to experiment with passive income. My selection criteria for projects were:
* Had to be buildable in 1 week. This made me focus on small, solvable problems.
* Had to be able to provide income.
* Minimal maintenance after the intial 1 week.

I ended up building two projects:
* [Gladomus](https://github.com/jiahuang/gladomus): a text message based service that would allow a user to search for things such as Wikipedia or Google Maps over text. I didn't have a data plan on my phone back then, so I would get lost often. My plan for monitization was to make people text a premium phone number.
* [DoTA 2 Guide](https://github.com/jiahuang/dota2builds): DoTA 2 from Steam had just released and took most of the existing fan base from the original DoTA as well as the spinoff game Heroes of Neworth (HoN). DoTA has a notoriously high learning curve, and this combined with small changes from the original DoTA and HoN made it extremely hard for newbies to get invovled. I had played a lot of DoTA but never got very good, and losing sucks so I wanted an app that would guide me to win. My computer was also pretty bad back then, so it made alt+tabbing impossible during gameplay.

I didn't really know how to reach the base of users for Gladomus (people who would know enough tech to use Google Maps & Wiki, but were too cheap to pay for data plans), and to be honest, my texting interface wasn't very good either, so I gave up on that pretty quickly.

The DoTA 2 guide did pretty well though. Over the course of its ~2 year lifespan it made about $2.5k, and was #1 in the search results for "DoTA 2 guide" on the Play store with 200k+ downloads.

## Developing the app in 1 week
Each DoTA character has multiple configurations of skills and items. My original plan was to outsource guide writing to my friends, but that failed due to them not really having any incentive. It was too time consuming for me to try to write the guides (plus I didn't have the expertise), so I ended up writing a scraper for existing DoTA 2 sites which had poor mobile rendering. I decided to store all the content in a SQLite database on the device to avoid having to build any kind of cloud backend.

The scrapers turned out to be my biggest advantage. Steam was actively developing the game at the time, so there would be new characters released every few weeks. I had all my data collection automated, so it was easy for me to update.

My breakdown of time was something around:
* 1 day app design
* 1 day trying to convince friends to write guides
* 2 days of writing a scraper
* 3 days of writing the Android app

In the end the app looked like this:
![dota 2 screen](/images/dota2_screen1.png) ![dota 2 screen](/images/dota2_screen2.png) ![dota 2 screen](/images/dota2_screen3.png)

Not terribly pretty, but it does the job. I didn't spend any additional time past the one week on app design. Futher development work was just bug fixing or data collection.

## Marketing
In order to get the word out, I published a post about the app on the DoTA 2 subreddit. It wasn't highly active at the time, so my post stayed on the front page for a week or so. That week got me the initial 100 downloads, which bumped me to ~#5 on the search results for "DoTA 2 guide". Over the next month I was very active in releasing builds and bug fixing which would push up the app rankings again. I got a lot of positive reviews about how fast the app updates were compared to competitor apps (+1 for automation). Within about a month the app was within the top 3 search results.

## Monetization
I released two versions of the app, one with ads and a "donate" version for $1 without ads. The ad version averaged ~$90/mo while the donate version only averaged ~$10/mo. The first few months I was lucky to clear ~$10 - $30/mo. The peak was around Dec '12 when the app made ~$450.

There were a few things that could have driven up my profits had I been willing to spend time on them:
* Changing my ad provider (AdMob). I read a few articles from other app developers getting higher eCPMs by switching.
* Make more features available on the donate version. Ideas I had were:
  1. Have the player sync with the app when the game starts. The app could then notify during the time of the game what levels/items/actions the player should be doing.
  2. Donate-only guides.
  3. Ability to write custom builds & share them.

## Things that went wrong

This was the app logo:
![dota 2 app icon](/images/dota2app_icon.png)

This is Steam's official DoTA logo:
![official dota 2 icon](/images/dota_icon.png)

That wasn't smart.

I also used "DoTA 2" directly in my app name. There were plenty of other apps that had "DoTA" in the name, so I just kinda didn't really think about it. 2 years later I got a DMCA notice that my app had been removed from Google Play.

I'm not a very good Android developer, and I only had ever bothered testing it on my Nexus One. I couldn't even verify bugs that existed on some other platforms. I also didn't have the energy to do so, which resulted in a few negative reviews. Fortunately this kind of thing seems to be common for "indie" Android apps, and I don't think it really effected downloads in a large way.

The web scraper scripts started breaking down when the layouts of the sites changed. I did some maintanence on it for a while, but never really fixed all the bugs. By that time I was busy with school (and the actual funded startup I was trying to run), so I just didn't release updates. This eventually ended with a ton of uninstalls which was terrible since the majority of the income came from ad impressions. Lowered ad revenue made me more reluctant to spend any time updating the app, a vicious cicle which just lead to more uninstalls.

I think I should have tried to flip the app about a year into it. I was no longer motivated to spend the time to build & release updates. At that time the app was still high in the search results, and could have done a lot better with a bit more development effort.

## Things I learned

* **The software lifecycle is long.** I basically learned some Android code in a week and then had it running on devices for 2 years. About a year in I tried to go in and play with some features, took one look at things, didn't know what they did, and Nope'd out. Maintenance at the end was painful so I didn't want to maintain it which just made things more painful.

* **Passive income is not passive.** There is still maintenance work which is a non-zero brain drain. Spend as much time building good automation tools while the motivation is still high, because it's much harder to do so a year into it.
