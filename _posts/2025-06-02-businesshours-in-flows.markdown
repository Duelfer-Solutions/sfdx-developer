---
layout: post
title:  "Working with BusinessHours in Salesforce Flows"
date:   2025-06-02 05:00:00 -0400
categories: salesforce flow businesshours
author: Tamara Chance
comments: true
image: assets/img/stockImages/laptop-frame-with-flow-image.png
---
### "Using a Flow, I need to create a Task with a Due Date that's in 3 _business_ days."

Calculations around Business Hours in Salesforce are a common ask. And if you’ve ever tried to solve it in Flow, you’ve probably run into a frustrating wall.

At first glance, it seems simple:

Just set the Due Date to TODAY() + 3, right?

🚫 Not quite.

That method ignores your org’s business hours. It doesn’t account for weekends. It definitely doesn’t respect holidays. And if your users are relying on accurate due dates for SLAs or follow-ups, those gaps matter.

In this post, I'll walk you through [how to set up an Apex action within a Flow](#how-to-use-it-in-flow-a-real-world-example) to provide accurate Business Time using a real example use case.
### The Problem With Flows and Business Time
Salesforce Flow gives you some standard Date and DateTime functions like TODAY(), NOW(), and WEEKDAY(), and you can use those to fake some business-day logic.

For example, you might write something like:
> "If today is Friday, add 3 days instead of 1."

That helps with weekends. But what about a Monday holiday? What if business hours start at 9am and your Flow runs at 7am—should the task still be due the next morning?

Salesforce Flow just isn’t built for that level of time awareness.
### The Apex Class That _Can_ Handle It
Thankfully, Salesforce does provide a way to do all this correctly—the BusinessHours Apex Class. It understands:

- Your org’s business hours
- Your active holiday calendar
- Exact DateTime math, including time zones and offsets

It includes methods like:

- [`BusinessHours.add(businessHoursId, startDate, intervalMilliseconds)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_add)
- [`BusinessHours.addGmt(businessHoursId, startDate, intervalMilliseconds)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_addGmt)
- [`BusinessHours.diff(businessHoursId, startDate, endDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_diff)
- [`BusinessHours.isWithin(businessHoursId, targetDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_isWithin)
- [`BusinessHours.nextStartDate(businessHoursId, targetDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_nextStartDate)

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

It runs all five of these calculations and returns all results in a single Admin-friendly flow output.

Below is just a snippet of the code so you can see how the flow inputs affect which values are returned. You can view the whole class in this [gist](https://gist.github.com/tamarachance/d81f1272ac1ba185e80c81108c90a783). Only one invocable method can be included in a single Apex Class, so for simplicity and ease of use I included all of the business time logic in a single class. 

The only required input is the _Business Hours Id_ since it is used in all of the [BusinessHours](#the-apex-class-that-can-handle-it) methods. The rest of the inputs are optional and based on whatever calculation you need returned. 

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

### How to Use It in Flow: A Real World Example
You want a Flow that when triggered (in this example by a Case creation) assigns a follow-up Task that's due in 3 business days.

Let’s walk through the Flow side of this setup. The first thing I did was to setup a Record-Triggered Flow, which I'll assume you already know how to do. 

#### Step 1: Retrieve your Business Hours
Once you've set up your entry criteria, getting your Business Hours into the flow is easy enough. `BusinessHours` records are accessible from the standard **Get Records** element in Flow. Here, I'm selecting the default Business Hours, which I've configured to be a typical Mon-Fri 9am-5pm workweek. Not shown below, I've also included Memorial Day as a Holiday.

![Screenshot of Get Records configuration](/assets/img/posts/businesshours-in-flows/get-records-setup.png)
![Screenshot of BusinessHours configuration page](/assets/img/posts/businesshours-in-flows/default-business-hours.png)
 
#### Step 2: Add the Action
In your Flow:

- Choose the Action element.
- Search for your custom Apex action—(the name depends on how you labeled it in the Apex).

![Screenshot of Apex Action](/assets/img/posts/businesshours-in-flows/select-apex-action.png)

#### Step 3: Provide the Inputs
Next, you’ll need to configure the Apex Action element and pass in a few values:

- Business Hours Id – Use the BusinessHours record retrieved in your Get Record element.
- Start Date - Since this is a record triggered flow, I'm using the current datetime of the running flow.
- End Date - Optional, used for `diff()` method. _Not applicable in this example._
- Interval - The increment of time _in milliseconds_ (For example, one 8 hour business day is 28800000 ms and 3 8hr business days or 24 hrs is 86400000 ms.)
- Target Date – Optional, used for `isWithin()` and `nextStartDate()`. _Not applicable in this example._

![Screenshot of Apex Action Configuration](/assets/img/posts/businesshours-in-flows/apex-action-inputs.png)

In this example, because I'm specifically looking for the `add()` calculation, I'm only providing (3) arguments to the apex action: **BusinessHours Id**, **startDate**, and **intervalMilliseconds**.
I'm using a constant variable to hold my inveral value in milliseconds because I prefer not to hardcode the value within an element. If I need to update this later, it's much easier to find the constant variable and update it there. But this value could also be held in a Custom Settings object, if that's your preference.

#### Step 3: Use the Outputs
The Apex Action returns a bunch of useful values:

- addedDate – The datetime after adding the interval (this is what we’ll use for our Task due date example)
- addedGmtDate – The same, but in GMT
- diffMs – The business time between start and end
- isWithinBusinessHours – True/false for whether targetDate is inside business hours
- nextBusinessStartDate – The next available business datetime from targetDate

You can use any of these downstream in your Flow!

![Screenshot of Apex Action Configuration](/assets/img/posts/businesshours-in-flows/apex-action-outputs.png)

Here, I'm storing the `addedDate` output in a variable called `dueDate` that will be passed into the **New Task** element as the `ActivityDate`.
#### Result
The Task’s due date is exactly 3 (8hr) business days after creation—accounting for holidays, weekends, and your org’s official hours. 

![Results in Debug Log](/assets/img/posts/businesshours-in-flows/results.png)

Note that I'm showing you the results in the debug log so that you can see both the _startDate_ and the output assigned to _dueDate_ side-by-side. Notice how it skipped the Friday, Saturday, Sunday, and Memorial Day.

No formula hacks. No guessing. Just accurate business logic.
### Bonus: What Else You Can Do With This
Here are a few more ideas for using this Apex Action:

- Set due dates 4 business hours after a support request
- Escalate cases when `.diff()` exceeds your SLA
- Send reminders only if `.isWithin()` is true (during open hours)
- Schedule follow-ups for the nextStartDate after a holiday or weekend

### Testing Note
I’ve written a separate post all about how to test BusinessHours logic in Apex, which in my opinion is where things get a lot more complex. You can find that here:

👉 [Testing BusinessHours in Apex]({% post_url 2025-05-23-apex-test-businesshours %})

### Wrapping Up
Flows are powerful, but when it comes to anything date-related that requires business time, you need a little Apex assist. This invocable method bridges the gap between the declarative and programmatic worlds, so you can build smart, accurate automations.

Want help extending this for other use cases? Drop me a note or leave a comment below. 

Happy Flow-building! 👩‍💻✨