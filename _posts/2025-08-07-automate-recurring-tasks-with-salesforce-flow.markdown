---
layout: post
title:  "Automate Recurring Tasks with Salesforce Flow"
date:   2025-08-07 05:00:00 -0400
categories: flows
author: Tamara Chance
comments: true
image: 
---
When I first tried to create a recurring task in Salesforce using Flow, I hit a wall.

I kept finding tutorials for how to make a task repeat manually through the UI. Or they showed how to build a scheduled flow to generate recurring tasks. But neither was a fit for the use case that I needed.

I had a **record-triggered flow** that already created tasks. I just needed one of those tasks to be recurring. And although the _due date_ would be 30 days out, it wasn't like a new task needed to be created _every_ 30 days. And I didn't want a long series of tasks either.

There were no clear tutorials for how to generate recurring tasks in record-triggered flow. So I figured it out myself. And now I’m going to show you how to do it, step by step.

## The Problem with Other Solutions
Most of the advice out there assumes you're:

- Creating a task by hand (not using Flow), or
- Building a big scheduled automation to create tasks every week

That’s fine in some cases, but as I stated above, didn't really work for my use case.

The good news? You can do it by setting a few lesser-known standard fields on the Task record.

## The Simple Solution: Repeat This Task
We’re going to walk through how to:

- Add a Create Records element to your flow
- Set the right fields to make the task repeat
- Skip the need for any scheduled flows

**What We’re Building**

You’ll have a record-triggered flow that creates a task—and that task will repeat based on when the user completes the original task.

<image placeholder – screenshot of the flow or finished Create Records element>

**Step 1: Understand the Fields You Need**

When you're creating a recurring task in Flow, you need to set some hidden fields that Salesforce normally uses behind the scenes when you click “Repeat This Task” in the UI.

Here are the key fields you’ll need:

Field Name	What It Does
IsRecurrence = TRUE	Tells Salesforce this is a repeating task
RecurrenceStartDateOnly = {!$Flow.CurrentDate}	Sets the start date
RecurrenceInterval = X	How often it repeats (e.g., every 1 week)
RecurrenceTimeframe = 'W'	D = Daily, W = Weekly, M = Monthly, Y = Yearly
RecurrenceRegeneratedType = 'Recurrence' (optional)	Makes sure the new task is tied to the original

<image placeholder – screenshot showing these fields configured in the flow>

**Step 2: Add or Update Your Create Records Element**

Inside your record-triggered flow:

Add a new Create Records element (or update an existing one that creates a Task)

Set your usual Task values (like Subject, WhatId, etc.)

Then include all the recurrence fields listed above

💡 Example setup:

IsRecurrence = TRUE

RecurrenceInterval = 1

RecurrenceTimeframe = 'W' → this means “every week”

RecurrenceStartDateOnly = {!$Flow.CurrentDate}

<image placeholder – full Create Records config screenshot>

**Step 3: Test It Out**

Run your flow in debug mode or trigger it by updating/creating the right record.

Then go check the Task that was created:

You should see the due date is 30 days from the previous task's due date.

<image placeholder – screenshot of the Task in UI showing “Repeats every week”>

🎉 That’s It!
Now you’ve got:

- A record-triggered flow that creates tasks ✅
- A repeating task that doesn’t require a schedule-triggered flow ✅
- Full control over how often the task repeats ✅
And you did it all without needing Apex or any extra logic.

**Bonus Tip: End the Repeating Task (Optional)**

Want the recurring task to stop after a while? Add this field:

This marks the task closed _at the same time_ as the Task is marked _Closed_. Therefore, the task does not repeat.

If you get stuck or something’s not working, drop a comment or DM me. I’d love to help.

And if you want more Salesforce flow tutorials, check out the following posts:
- [link here]()
- [link here]()