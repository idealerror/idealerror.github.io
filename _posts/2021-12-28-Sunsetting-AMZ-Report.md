---
title: Sunsetting AMZ.report / Big Data Learnings
author: Chris Geiger
date: 2021-12-28 12:00:00
categories: [Programming,DevOps]
tags: [bigdata,amazon,programming]
---

Today I sunset the AMZ.report platform.  Building this platform was a lot of fun and came with a lot of learnings.  

Some quick stats of the data I was able to aggregate:

| Stat      | Value |
| ----------- | ----------- |
| Search Term Reports (day/month/quarter)      | 840+       |
| Unique Search Terms   | 7.5M+        |
| Unique ASINs | 9.8M+ | 
| Unique Ranking Rows | 838M+ | 

While it was fun to build the tool and maintain it, there was some costs associated.  Primarily with storing the data in a database. Below are some average costs of the services utilized.

| Service      | Cost |
| ----------- | ----------- |
| Database/Storage (RDS)     | ~$300/mo       |
| Compute (EC2)   | ~$40/mo        |
| AWS Average Cost | ~$340/mo | 

This was a very simplified build with no high availability, fault tolerance, etc.  Storing over 1B rows in a database and running large queries on it is not cheap, especially if you expect the responses in less than a couple of seconds.

This definitely benefited a lot of people who used the platform, but it did not generate any revenue, so with that, it's been sunset.  

I did back up the code and the data in the event that if I need to re-deploy it in the future - I can.

Feel free to reach out if you have any questions.  My contact information can be found on the bottom of the page sidebar.
