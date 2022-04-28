+++
date = "2014-01-04T00:31:00+00:00"
draft = false
tags = ["coding", "api"]
title = "Facebook insights have different return formats."
+++
https://graph.facebook.com/me/insights/page_impressions_unique/day?access_token={{TOKEN}}&since=1388649600&until=1391241600

- 1388649600 = 1/2/2014 12:00:00 AM GMT-8
- 1391241600 = 2/1/2014 12:00:00 AM GMT-8

returns something that looks like this:


    {
      /* previous, next page, etc keys */
      "data": [
        {
          "values": [
            {
              "end_time": "2014-01-03T08:00:00+0000",
              "value": 4
            },
            {
              "end_time": "2014-01-04T08:00:00+0000",
              "value": 1
            },
            {
              "end_time": "2014-01-05T08:00:00+0000",
              "value": 0
            }
            .... /* More 0 values here until Feb 2 */
          ],
          "period": "day",
          "description": "Daily: The number of people who have seen any content associated with your Page. (Unique Users)",
          "title": "Daily Total Reach"
        }
      ]
    }


Whereas 

https://graph.facebook.com/me/insights/page_views/day?access_token={{TOKEN}}&since=1388649600&until=1391241600

returns:


    {
    /* previous, next, etc keys*/
         data: []
    }


It seems at some point that Facebook has fixed the problem where some insights are always delayed by two days, because I obtained up-to-date data for "page_impressions_unique" (as a matter of fact, I have data for January 4 at midnight even though right now it's January 3rd.. hm.. and the timezone is supposed to be PST for Insights data). But for page_views there is no data available yet, so I get a blank array. So why don't I get an array full of 0s, Facebook?

Of course, I probably should not be requesting data in the future. That bug is my fault, and it was uncaught because Facebook always returned no data if it wasn't available until sometime recently apparently. 

In any case, this bug sucks. The fix is probably to request data only until the current day's midnight in PST, but Facebook should be more consistent with its responses and not change them randomly. And yes, I understand that nowhere in the specification does it say what should happen if you request data for dates in the future.

APIs are broken.