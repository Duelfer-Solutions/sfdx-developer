---
layout: post
title:  "Working with BusinessHours in Salesforce Flows"
date:   2025-05-20 07:00:00 -0400
categories: salesforce flow businesshours
author: Tamara Chance
comments: true
image: 
---
### **"Using a Flow, I need to create a Task with a Due Date that's in 1 _business_ day."**

Calculations around Business Hours in Salesforce are a common ask. And if you’ve ever tried to solve it in Flow, you’ve probably run into a frustrating wall.

At first glance, it seems simple:

Just set the Due Date to TODAY() + 1, right?

🚫 Not quite.

That method ignores your org’s business hours. It doesn’t account for weekends. It definitely doesn’t respect holidays. And if your users are relying on accurate due dates for SLAs or follow-ups, those gaps matter.

In this post, I'll walk you through [how to set up an Apex action within a Flow](#use-an-invocable-apex-action) to provide accurate Business Time and give you a [Real Example: Create a Task Due in 1 Business Day](#real-example-create-a-task-due-in-1-business-day).
### The Problem With Flows and Business Time
Salesforce Flow gives you some standard Date and DateTime functions like TODAY(), NOW(), and WEEKDAY(), and you can use those to fake some business-day logic.

For example, you might write something like:
> "If today is Friday, add 3 days instead of 1."

That helps with weekends. But what about a Monday holiday? What if business hours start at 9am and your Flow runs at 7am—should the task still be due the next morning?

Salesforce Flow just isn’t built for that level of time awareness.
### The Class That _Can_ Handle It
Thankfully, Salesforce does provide a way to do all this correctly—the BusinessHours Class. It understands:

- Your org’s business hours
- Your active holiday calendar
- Exact DateTime math, including time zones and offsets

It includes methods like:

- BusinessHours.add() – adds business time to a datetime
- BusinessHours.diff() – calculates business hours between two datetimes
- BusinessHours.isWithin() – checks if a datetime is during business hours
- BusinessHours.nextStartDate() – finds the next available business time slot

It’s incredibly powerful—but it’s not available in Flow. 😞
### Use an Invocable Apex Action
To bridge the gap between Flows and BusinessHours, I created a simple Invocable Apex Action.

That means:

- You can call it just like any other Flow Action.
- You provide the inputs (e.g., start datetime, interval in milliseconds, business hours ID).
- It gives you back the answers: the added datetime, the diff, and more.

Here’s what the Invocable Action supports:

| **Method** | **Description** |
| :---------------: | :-------------------------------------- |
| `add()` | Add business time to a datetime |
| `addGmt()` | Same as above, but using GMT |
| `diff()` | Get number of business milliseconds between two datetimes |
| `isWithin()` | Returns true if a datetime is within business hours |
| `nextStartDate()` | Finds the next datetime that falls inside business hours |

It runs all five of these calculations and returns all results in a single Flow-friendly output.

Below is just a snippet of the code. You can view the whole class in this [gist](https://gist.github.com/tamarachance/d81f1272ac1ba185e80c81108c90a783).

```apex
@InvocableMethod(label='Run Business Hours Calculations')
public static List<Output> runCalculations(List<Input> inputList) {
    List<Output> results = new List<Output>();
    for (Input ip : inputList) {
        Output op = new Output();
        // Run BusinessHours.diff() if start and end dates are present
        if (ip.startDate != null && ip.endDate != null) {
            op.diffMs = BusinessHours.diff(ip.businessHoursId, ip.startDate, ip.endDate);
        }
        // Run BusinessHours.add() & .addGmt() if startDate and interval are present
        if (ip.startDate != null && ip.intervalMilliseconds != null) {
            op.addedDate = BusinessHours.add(ip.businessHoursId, ip.startDate, ip.intervalMilliseconds);
            op.addedGmtDate = BusinessHours.addGmt(ip.businessHoursId, ip.startDate, ip.intervalMilliseconds);
        }
        // Run BusinessHours.isWithin() & .nextStartDate() if targetDate is present
        if (ip.targetDate != null) {
            op.isWithinBusinessHours = BusinessHours.isWithin(ip.businessHoursId, ip.targetDate);
            op.nextBusinessStartDate = BusinessHours.nextStartDate(ip.businessHoursId, ip.targetDate);
        }
        results.add(op);
    }
    return results;
}
```

### How to Use It in Flow
Let’s walk through the Flow side of this setup.

#### Step 1: Add the Action
In your Flow:

- Choose the Action element.
- Search for your custom Apex action—(the name depends on how you label it in the Apex).

#### Step 2: Provide the Inputs
You’ll need to pass in a few values:

- Business Hours Id – Use a Get Records to fetch the correct one. Most orgs will have one default.
- Start Date
- End Date
- Interval - (i.e. A 24 hour business day = 86400000 ms)
- Target Date – Optional, used for `isWithin()` and `nextStartDate()`.

<!-- <insert image here> -->

#### Step 3: Use the Outputs
The Apex Action returns a bunch of useful values:

- addedDate – The datetime after adding the interval (this is what we’ll use for our Task due date example)
- addedGmtDate – The same, but in GMT
- diffMs – The business time between start and end
- isWithinBusinessHours – True/false for whether targetDate is inside business hours
- nextBusinessStartDate – The next available business datetime from targetDate

You can use any of these downstream in your Flow.

<!-- <insert image here> -->

### Real Example: Create a Task Due in 1 Business Day
Let’s put it all together:

- You want a Flow that when triggered (in this case by a Case creation) assigns a follow-up Task that's due in 1 business day.

Steps:

- Get the Business Hours record (I'm using the default).
- Use a Constant or Assignment to define the interval:
86400000 (1 day in milliseconds)
- Call the Business Hours Apex Action.
- Use the addedDate output to set the Task Due Date.

<!-- <insert image here> -->

Result: the Task’s due date is exactly 1 (8hr) business day after creation—accounting for holidays, weekends, and your org’s official hours.

No formula hacks. No guessing. Just accurate business logic.

### Bonus: What Else You Can Do With This
Here are a few more ideas for using this Apex Action:

- Set due dates 4 business hours after a support request
- Escalate cases when `.diff()` exceeds your SLA
- Send reminders only if `.isWithin()` is true (during open hours)
- Schedule follow-ups for the nextStartDate after a holiday or weekend

### Testing Note
I’ve written a separate post all about how to test BusinessHours logic in Apex, which in my opinion is where things get a lot more complex. You can find that here:

👉 [Testing BusinessHours in Apex](./2025-05-14-apex-test-businesshours.html)

### Wrapping Up
Flows are powerful, but when it comes to anything date-related that requires business time, you need a little Apex assist. This invocable method bridges the gap between the declarative and programmatic worlds, so you can build smart, accurate automations.

Want help extending this for other use cases? Drop me a note or leave a comment below. 

Happy Flow-building! 👩‍💻✨