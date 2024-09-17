---
layout: post
title:  "How to Use Related Records as a Workaround for Dynamic Forms"
date: 2024-09-17 07:00:00 -0400
tag: dynamic-forms
categories: salesforce low-code
author: Tamara Chance
comments: true
image: assets/img/stockImages/shortcut-workaround-bypass.png
---
Despite the buzz surrounding the long-awaited support for Dynamic Forms on standard objects which Salesforce made available in September 2023, it falls short in addressing specific use cases. Not at all surprising when you read the long list of [known issues](https://help.salesforce.com/s/articleView?id=sf.dynamic_forms_known_issues.htm&type=5). 

Take for instance a recent use case wherein attempting to employ Dynamic Forms to conditionally display sections of a Detail Page as independent components on a record page seemed like a textbook use case. In practice, however, this had some peculiar side effects that resulted in having to scrap the solution altogether. So in this post, I'll share a similar example scenario and how to use a Related Records component as a workaround of Dynamic Forms.

## Use Case

Our client in this scenario was using the Case Standard object. They recently started using an custom object to track information about their products' inspection information. But prior to this improved data structure, the Case record contained the Date/Time of a selected product's most recent inspection in the _Additional Information_ section of the record details. 

**In this particular situation, a data migration on all existing Case records was not possible with the transition to the new custom object**. As a result, users of the client org needed a way to differentiate between Cases that existed prior to using the custom object, _and_ they wanted to be able to see that historical inspection information for those existing cases but not any new cases that would be using the new data model. 

To differentiate between Cases using the new inspection custom object and historical inspection information, we added a new `Pre_Inspection__c` flag field to the Case object. So that way, when this field is _true_ then, we can dynamically display that Case's existing historical inspection information on the page. 

But the client didn't just want to conditionally render the historical inspection fields within the record details; they wanted that information displayed in an editable format as a right-sidebar widget. 

You can see why we initially wanted to use dynamic forms for this scenario. Unfortunately, in implementing a dynamic form component on the case record page, this inadvertently disrupted the default New and Edit actions, posing a hurdle to the intended overall workflow. 

![New and Edit Overriding Behavior](/assets/img/posts/related-records-workaround/Error-Dynamic-Forms-New-Case-Modal.png)

When using Dynamic Forms this overriding behavior has some good use cases for being able to customize the New and Edit modal layouts. _But_ for our use case, the client still needed to be able to use the _default_ New and Edit modal layouts. This intended behavior made the use of Dynamic Forms nonviable. 

## Solution
Here's an alternative approach using a Related Records component with an Update action that we used to seamlessly resolve the challenge _without_ relying on Dynamic Forms:

### Step 1: Access the Record Page
Navigate to the record page where the 'Pre-Inspection' flag governs the display of the 'Historic Inspection Information' section.

![Pre-Inspection Field and Inspection Info Field](/assets/img/posts/related-records-workaround/Flag-field-and-Inspection-Info-in-Details-on-Case.png)

### Step 2: Implement an Update Quick Action
Create a simple Update Action on the Case object and add the necessary fields from the 'Historic Inspection Information' section to the Action Layout.

![Create Update Action](/assets/img/posts/related-records-workaround/Create-Update-Action.png) ![Add fields to action layout](/assets/img/posts/related-records-workaround/Create-Layout.png)

You may get a warning when removing required fields from the update action layout. For this use case, it wasn't necessary to keep any non-relevant fields on the layout, even required ones. But you should ensure that users will always have a value in any required fields when implementing this solution.

### Step 3: Integrate the Related Record Component
In the upper-right corner of the record page, click the gear icon and select **Edit Page** to open the page editor. Next, incorporate the 'Related Record Component' onto the desired location of the page layout.

![Add related record component onto the record page](/assets/img/posts/related-records-workaround/Add-Related-Record.png)

Make sure to select **Use this record** for the lookup field so that it knows to update the current record. Then, select your newly created Update Action. _Note that there is an existing Update Case action, but its layout is not editable. Thus you need to create a new one._ You can also wait to create your update action until this step, as well, by selecting the _New Update Action_ link. 

### Step 4: Configure the Related Record Component Visibility
Customize the Related Record Component visibility settings to display the 'Historic Inspection Information' section based on the 'Pre-Inspection' flag.

![Set Visibility Filter](/assets/img/posts/related-records-workaround/Set-Component-Visibility.png)

### Step 5: Remove the inspection information fields from the page layout
Now that the fields relating to historic inspection information are on the widget, they no longer need to be on Case record detail layout. 
Navigate to the Case page layout and remove the inspection information fields.

### Step 6: Validate the Solution
Thoroughly test the setup to ensure the 'Historic Inspection Information' section displays conditionally based on the 'Pre-Inspection' flag, without disrupting default modals during New and Edit actions. Below is an example UAT Plan you can use:

<script src="https://gist.github.com/tamarachance/f51b55dcc27ae2c3b6a20975f9e9e461.js"></script>

## Conclusion
By leveraging the Related Record Component in conjunction with a straightforward Update Action, we circumvented the limitations posed by Dynamic Forms. This approach facilitates the conditional display of the 'Historic Inspection Information' section while preserving the functionality of the default modals, providing an effective solution tailored to the client's specific use case.