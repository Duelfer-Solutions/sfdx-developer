---
layout: post
title:  "Why You Can’t Refresh the Salesforce Activity Timeline Programmatically"
date:   2025-08-12 05:00:00 -0400
categories: activity timeline
author: Tamara Chance
comments: true
image: assets/img/stockImages/activity-timeline-refresh.png
---
If you’ve ever tried to programmatically force the standard Activity Timeline to magically update itself when a new task is generated — you’ve probably discovered something frustrating:

It doesn’t.

And after hours of googling, testing every API you can find, and scouring the community, you land on the harsh truth:

There is no supported way to programmatically refresh the Activity Timeline — yet.

Let’s unpack that.

## The Problem
You build a custom component that closes tasks or logs events related to a Case. It works great.

But when you look at the Activity Timeline, _Salesforce’s slick, built-in feed of past and upcoming activities_, it just sits there, totally unaware anything happened. The user has to manually refresh the page or click the _Refresh_ button, which kind of defeats the point of all that automation, right?

So naturally, you try to fix it.

## Everything That Doesn’t Work (I Tried It So You Don’t Have To)
Here’s a quick rundown of all the usual suspects, and why none of them get the job done:

| **Approach** | **What It Does** |	**Why It Fails** |
| :---------------: | :-------------------------: | :-------------------------: |
| `$A.get('e.force:refreshView').fire();` | Aura event that refreshes the view. | Only works in Aura components, and doesn’t affect the Timeline. |
| `notifyRecordUpdateAvailable()` | Tells UI API to refresh data for a record. | Tasks and Events aren’t supported by the UI API, so nothing happens. |
| RefreshEvent or RefreshViewAPI | Forces LDS to re-fetch data in your LWC tree. | Works great for custom Lightning Web Components. Does nothing for standard Lightning Components. |
| Publishing a Platform Event | Send out an event when something changes. | You can listen in custom components, but standard ones like the Activity Timeline don’t care. |

## So What Does Work?
Honestly? While it may be painful for the user from a UX perspective, the best way to maintain data integrity is to force a full page refresh in your custom component. Or continue manually refreshing the page.

Here's an example to refresh programmatically:
```
import { LightningElement, wire } from 'lwc';
import { NavigationMixin, CurrentPageReference } from 'lightning/navigation';

export default class ExampleComponent extends NavigationMixin(LightningElement) {

    @wire(CurrentPageReference)
    _pageRef;

    handleSave() {
        // TODO: execute your code
        // then, if successful...
        this.refreshPage();
    }

    // Full page refresh to force the activity timeline to reload new data
    refreshPage() {
        this[NavigationMixin.Navigate](this._pageRef);
    }
}
```
## Is There Any Hope?
Yes. Salesforce has an [IdeaExchange request](https://ideas.salesforce.com/s/idea/a0B8W00000GdnHYUAZ/refresh-the-timeline-activity-from-the-lightning-component) that’s currently marked “In Development.”

That’s promising.

Once it becomes GA (Generally Available), we’re hoping for an official API or refresh mechanism to finally make the Activity Timeline react to backend changes — like any other reasonable component.

But until then, your options are limited.

Your best bet is to wait it out and live with the refresh button a little longer or programmatically force a page refresh.

## Final Thoughts
Salesforce is an amazing platform — but sometimes the standard components don’t quite keep up with modern use cases. If you’re hitting this wall, you’re not alone. Just know that the limitation isn’t with your code — it’s with the platform.