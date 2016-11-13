---
layout: post
title:  "Sprint 3 Progress (Ethan Gill)"
date:   2016-11-12 18:44:20 -0400
categories: gill
---

During Sprint 3, our team finished work on our project's requirements, and drafted a document summarizing the work completed. Additionally, we created a keynote presentation to accompany a live demo of our work. The paper, code, and keynote will be available on this site and will be turned in on November 14, 2016.

While building the 'google_calendar' module, our team ran into an issue. To allow for ease of use, we created one client id and secret pair from the Google Developer Console. Every user must be authenticated using OAuth2 in order to allow MagicMirror to access their account using our Developer Console id and secret. To accomplish this, the 'google_calendar' MagicMirror module checks at launch for any configured Google Calendars without a 'token' key specified. If it finds one, it exits with an explanatory error message to be displayed in the terminal.

This solution strikes a balance between ease of use and configurability. Users can use multiple Google Accounts with this module, such as a business account and a personal one, without the need to set up their own Google Developer Console Projects.
