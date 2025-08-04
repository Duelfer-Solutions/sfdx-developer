---
layout: post
title:  "Better UX with Dependent Picklists: Required Fields Done Right"
date:   2025-08-11 05:00:00 -0400
categories: no-code config
author: Tamara Chance
comments: true
image: 
---
Ever needed a field to be required—but only _sometimes_?

We ran into a common UX challenge while configuring a Salesforce record page:

✅ We needed the _Closed Reason_ field on the Case object to be required,

❌ but _only_ when the Status field was set to **Closed**.

Requiring it at the field level wasn’t going to work—it would force users to fill it out no matter what the Status was. Not helpful, and definitely not a great experience.

In our case, when the **Case.Status** picklist field is set to _Closed_, we want to **require** the _Closed Reason_ field have a value. But for statuses like _Open_ or _In Progress_, the Closed Reason should be totally optional (or even hidden).

We wanted a way to:

- Prevent users from skipping the Closed Reason when closing something
- Avoid annoying them with an irrelevant field during other statuses
- Keep the logic in configuration (no Apex triggers or custom validations)

Turns out, the combination of two things made this really smooth:

1. **Dependent Picklists**

    We made _Closed Reason_ a dependent picklist that is only enabled when _Status = Closed_.

2. **Page Layout Requirement**

    We set _Closed Reason_ as **required on the page layout**, not at the field level.

The magic is: when the controlling field (Status) isn't set to Closed, the Closed Reason field is disabled—so the required setting is gracefully bypassed. But when Status is _Closed_, the field is enabled _and_ it's required. 👨‍🍳

### How to Set Up a Dependent Picklist
This makes sure the _Closed Reason_ field only appears when _Status = Closed_.

1. Go to **Setup → Object Manager →** select your object (e.g., Case or Opportunity).
2. Click **Fields & Relationships**.
3. Click into the **Closed Reason** field.
4. Click the **Field Dependencies** button.
5. Click **New** to create a new dependency.
6. For **Controlling Field**, select **Status**.
7. For **Dependent Field**, select **Closed Reason**.
8. Click **Continue**.
9. In the matrix that appears, check the box for the value _Closed_ under the Status column, and select the applicable values for Closed Reason that should be available when Status = Closed.
10. Save the dependency.

👉 Result: Closed Reason only appears when Status = Closed.

### How to Make the Field Required on the Page Layout
This ensures the field is only required when it’s enabled (i.e., when Status is Closed).

1. Still in Object Manager, go to the **Page Layouts** section for your object.
2. Open the layout where the field appears (e.g., Case Layout).
3. Find the **Closed Reason** field on the layout.
4. Click the **wrench icon** (🔧) or double-click the field.
5. Check the **Required** checkbox.
6. Click **OK**, then **Save** the layout.

👉 Important: Do _not_ mark the field required at the field-level definition—only on the layout.

### Why This Works So Well
This setup:

- Keeps users focused only on the fields they _need_ to see
- Avoids conditional validation rules (and their maintenance headaches)
- Gives you a low-code, maintainable UX pattern that scales across similar scenarios

It’s small. But it’s mighty.