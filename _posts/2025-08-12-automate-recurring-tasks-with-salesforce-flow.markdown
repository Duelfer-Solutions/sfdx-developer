---
layout: post
title:  "Automate Recurring Tasks with Salesforce Flow"
date:   2025-08-12 05:00:00 -0400
categories: flows
author: Tamara Chance
comments: true
image: assets/img/stockImages/task-checklist.png
---
When I first tried to create a recurring task in Salesforce using Flow, I hit a wall.

I kept finding tutorials for how to make a task repeat manually through the UI. Or they showed how to build a scheduled flow to generate recurring tasks. But neither was a fit for the use case that I needed.

I had a **record-triggered flow** that already created tasks. I just needed one of those tasks to be recurring. And although the _due date_ would be 30 days out, it wasn't like a new task needed to be created _every_ 30 days. And I didn't want a long series of tasks either.

There were no clear tutorials for how to generate recurring tasks in a record-triggered flow. So I figured it out myself. And now I’m going to show you how to do it, step by step.

## The Problem with Other Solutions
Most of the advice out there assumes you're:

- Creating a task by hand (not using Flow), or
- Building a big scheduled automation to create a series of recurring tasks

That’s fine in some cases, but as I stated above, didn't really work for my use case.

The good news? You can do it by setting a few lesser-known standard fields on the Task record.

## The Simple Solution: Repeat This Task
We’re going to walk through how to:

- Add a Create Records element to your flow to create a new Task
- Set the right fields to make the task repeat
- Skip the need for any scheduled flows

**What We’re Building**

You’ll have a record-triggered flow that creates a task—and that task will repeat based on when the user completes the original task.

<!-- <image placeholder – screenshot of the flow or finished Create Records element> -->

**Step 1: Understand the Fields You Need**

When you're creating a recurring task in Flow, you need to set some standard fields that Salesforce doesn't normally use in the UI. **Note: This is an important step. `Repeat This Task` and `RecurrenceInterval` both need to be visible on the Task page layout.**

Here are the key fields you’ll need to know:

| **Field Name** | **Field Values** | **What It Does** |
| :------------: | :--------------: | :--------------: |
| IsRecurrence | True/False | Tells Salesforce if this is a repeating task |
| RecurrenceInterval | Number |	The number of _days_ after the task’s due date or completed date when you want the next task to be due. |
| RecurrenceRegeneratedType | After due date/After date completed/(Task Closed) |	Makes sure the new task is assigned the correct expected due date |

I'll link to the [documentation](https://help.salesforce.com/s/articleView?id=sales.tasks_repeating.htm&type=5), but here's just a quick run down of the picklist choices for `RecurrenceRegeneratedType` and what they mean.

- After Due Date: Creates recurrences with a due date set to the RecurrenceInterval value plus the previous task's due date.
- After Date Completed: Creates recurrences with a due date set to the RecurrenceInterval value plus the date the task was completed.
- (Task Closed): Indicates that the task was closed as part of a repeating series.

<!-- <image placeholder – screenshot showing these fields configured in the flow> -->

**Step 2: Add Your Task Create Records Element**

For this example, we'll use the scenario I mentioned at the start of the post. 

> When a Case is created, generate a new Task. The new task should have a due date set to 30 days from today. 
> Completion of this task should generate a recurring task with the same subject and due date.

1. Create your record-triggered flow, and add a new Create Records element that creates a Task.
2. Set your usual Task values (like Subject, WhatId, etc.)
3. Then include all the recurrence fields listed above:
    * IsRecurrence = TRUE
    * RecurrenceRegeneratedType = After Due Date
    * RecurrenceInterval = 30 (This is a required field.)

<!-- <image placeholder – full Create Records config screenshot> -->

**Step 3: Test It Out**

1. Run your flow in debug mode or trigger it by creating the right record. Personally, I prefer to test it in Debug Mode first to verify that the output has the correct values.
2. Go to the Debug Log and check the Task that was created. You should see the due date is 30 days from today. 
3. Now **Activate** your flow.
4. Trigger your flow to run by creating the right record. You should see the Task in the Activity Timeline. Make a note of the due date.
5. Mark the task as **Completed**, and refresh the Activity Timeline.
6. You should now see a new Upcoming Task with the same subject line. But check out the due date! It is 30 days from the _original task's__ due date. 

<!-- <image placeholder – screenshot of the Task in the Activity Timeline in the UI> -->

🎉 That’s It!
Now you’ve got:

- A record-triggered flow that creates tasks ✅
- A repeating task that doesn’t require a schedule-triggered flow ✅
- Full control over how often the task repeats ✅

**Bonus Tip: End the Repeating Task (Optional)**

Want the recurring task to stop after a while? You'll first need to trigger an automation to run. For instance, when a Case's status is updated to _Closed_ or something along those lines. This should launch another record-triggered flow. 

Set the `Task.RecurrenceRegeneratedType` to `(Task Closed)` _at the same time_ as the `Task.Status` is marked `Completed`. This will stop any future tasks from being created.

<!-- <insert image of stop tasks> -->

If you get stuck or something’s not working, drop a comment or DM me. I’d love to help.

And if you want more Salesforce flow tutorials, check out the following posts:
- [How to Unlock a Record in a Salesforce Approval Process Using Flow]({% post_url 2025-06-02-businesshours-in-flows %})
- [Working with BusinessHours in Salesforce Flows]({% post_url 2025-06-02-businesshours-in-flows %})