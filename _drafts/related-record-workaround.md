---
layout: post
title:  "How to Use Related Records as a Workaround for Dynamic Forms"
tag: dynamic-forms
categories: salesforce low-code
author: Tamara Chance
comments: true
image: assets\img\stockImages\shortcut-workaround-bypass.png
---
Despite the buzz surrounding the Winter '23 Salesforce release and its Dynamic Forms feature, it fell short in addressing specific use cases. For instance, attempting to employ Dynamic Forms to conditionally display sections as components on a record page while preserving default modals resulted in unintended interference and limitations. In this post, we'll cover how to workaround the limitations that still exist for Dynamic Forms using the Related Records component.

## Case Study

We recently encountered a scenario where a client required a section on the Detail page of a record to be displayed as a right-sidebar component. The fields displayed in this component also needed to be editable. Let's call that section, the 'Historic Decision Information' section, which would conditionally appear when a 'Pre-Decision' flag field was set to true. Implementing Dynamic Forms initially seemed promising. It seemed like a text-book example use case for the feature. 

Unfortunately, it inadvertently disrupted the default New and Edit modals, posing a hurdle to the intended workflow. When using Dynamic Forms this overriding behavior has some good use cases for being able to customize the New and Edit modals. But for our use case, we still need the default New and Edit modals to load. For us, this intended behavior was more of a bug. 

Here's an alternative approach that we used to seamlessly resolve the challenge _without_ relying on Dynamic Forms:

### Step 1: Access the Record Page
Navigate to the record page where the 'Pre-Decision' flag governs the display of the 'Historic Decision Information' section.

<<Add image of pre-decision flag>>

### Step 2: Integrate the Related Record Component
In the page editor, incorporate the 'Related Record Component' onto the desired location of the page layout.

<<Add image or gify of adding the related rcord component and setting to use this record>>

### Step 3: Configure the Related Record Component
Customize the Related Record Component settings to link and display the necessary fields from the 'Historic Decision Information' section based on the 'Pre-Decision' flag.

### Step 4: Implement an Update Quick Action
Create a simple Update Quick Action containing the necessary fields from the 'Historic Decision Information' section.

### Step 5: Associate the Update Quick Action
Associate the Update Quick Action with the 'Pre-Decision' field to enable users to modify the section when needed.

### Step 6: Validate the Solution
Thoroughly test the setup to ensure the 'Historic Decision Information' section displays conditionally based on the 'Pre-Decision' flag, without disrupting default modals during New and Edit actions. Her is an example Test Plan you can use:

<<Embed Gist of Smoke Test Here>>

## Conclusion
By leveraging the Related Record Component in conjunction with a straightforward Update Quick Action, we circumvent the limitations posed by Dynamic Forms. This innovative approach facilitates the conditional display of the 'Historic Decision Information' section while preserving the functionality of default modals, providing an effective solution tailored to the specific use case.