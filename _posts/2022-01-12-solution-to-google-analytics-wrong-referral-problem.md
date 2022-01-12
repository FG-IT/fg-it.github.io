---
title: Solution to google analytics counts payment gateway domain(eg. paypal.com) as transaction referral
date: 2022-01-12 09:30:09
author: YZ
categories:
- Troubleshooting
tags:
- google analytics
- paypal
- google ads
- referral
---

We use **paypal** as a payment gateway on our online store. However, when we check **google analytics**, no matter what bring the customer to our website, if they pay using paypal, paypal.com is counted a referral. The original referral is lost.

## Why it happens
By default, a referral automatically triggers a new session. So, when user is redirected from paypal to our website, a new session starts. This time, paypal.com is the referral.

## Solution - exclude referrral source

>When you exclude a referral source, traffic that arrives to your site from the excluded domain doesn’t trigger a new session. If you want traffic arriving from a specific site to trigger a new session, don't include that domain in this table.
>
>Because each referral triggers a new session, excluding referrals (or not excluding referrals) affects how sessions are calculated in your account. The same interaction can be counted as either one or two sessions, based on how you treat referrals.
>
>For example, a user on my-site.com goes to your-site.com, and then returns to my-site.com. If you do not exclude your-site.com as a referring domain, two sessions are counted, one for each arrival at my-site.com. If, however, you exclude referrals from your-site.com, the second arrival to my-site.com does not trigger a new session, and only one session is counted.

## Step by step configuration
You can exclude referral directly from google analytics.
- Click the **Admin** option at the bottom of the Analytics panel on the left.
- In the **Property** column, find **Tracking Info**, and then select **Referral Exclusion List**
- Click **Add Referral Exclusion** button, add the domain(eg. paypal.com) you want to exclude.

[Another more detailed guide](https://www.monsterinsights.com/how-to-add-paypal-to-referral-exclusion-list-in-google-analytics/)
