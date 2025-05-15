---
layout: post
title:  "Using AgentForce to Learn LWC Testing: A Mixed Experience"
date:   2025-05-12 04:00:00 -0400
categories: salesforce agentforce test generator dev assistant
author: Caitlin Sullivan
comments: true
image: /assets/img/stockImages/code-editor.png
---
As a newer developer with absolutely no experience writing Jest tests for lightning web components, I was excited to try out AgentForce for Developers to get jump-started with writing tests for my most recent LWC. While I didn’t expect it to be fool-proof, the tool seemed like it could be really useful in simplifying some of the more tedious parts of test writing.

### **Getting Started with AgentForce for Developers**

To get started, I made sure I had the AgentForce for Developers extension installed in VS Code. Once installed, an AgentForce icon appears on the lefthand side activity bar. When you open it, you’ll see the Dev Assistant. There’s a few ways you can interact with this extension, but my goal was to try out the test generation.

![Launching the Dev Assistant](/assets/img/posts/agentforce-test-generator/launch-dev-assistant.png)

### **Generating a Test File from Scratch**

The interface is intuitive: you select which type of test you want to generate (LWC or Apex) and then choose the component you want to generate tests for. After clicking "Generate", it will add tests to your existing test file, if you already have one, or it will create a brand new test file for you. This comes in handy when you don’t want to take the time to create a test folder and set up the test file, or if you're new to writing tests and you don't know how to set up the file. Most of this will be done for you. 

Once it’s done generating, you can take a look at the difference between the two files (in this case there was no existing test file, so there’s nothing on the lefthand side). You can choose to accept, discard, or regenerate the test file.

![First Attempt at Test Generation](/assets/img/posts/agentforce-test-generator/first-attempt-test-gen.png)

The test file is set up well at first glance. We have a new \__tests__ folder in the component directory, and we have a test file with the appropriate naming convention. Right off the bat, though, when you run the test, you can see there’s some issues. We don’t have a component called `counter` - it should be `caseBusinessAgeConfigurationPage`. Additionally, the tests themselves don't actually have anything to do with our component. Regenerating doesn’t help either, it just adds another test and doesn’t resolve the issue. 

![First Test After Running](/assets/img/posts/agentforce-test-generator/first-test-after-run.png)

### **Making Some Tweaks and Trying Again**

Let’s make some changes and try again. This time I deleted the the test so we're just left with the test setup. I also removed all references to the `counter` component and replaced them with the name of our component, `caseBusinessAgeConfigurationPage`.

![Gutted First Test Generation](/assets/img/posts/agentforce-test-generator/gutted-first-test-gen.png)

Once these changes are made, I tried generating the test again. While we no longer have any references to the `counter` component, we're still getting a failing test that's referencing a non-existent function.

![Second Attempt at Test Generation](/assets/img/posts/agentforce-test-generator/second-attempt-test-gen.png)

### **Generating Tests After Providing Setup**

So I decided to try to provide even more of the test setup to point the Dev Assistant in the direction I want it to go. I started by adding a mock data file, since it doesn't seem like the Dev Assistant will do this for you.

![Mock Data File](/assets/img/posts/agentforce-test-generator/mock-data-file.png)

I also went ahead and imported my Apex wire adapter and mocked that function within my Jest file. Again, it doesn't seem like the Dev Assistant will set this up for you. Finally, I added an "outline" for a test that I wanted to generate. I was hoping the Dev Assistant would take in all this new information (a mock data file, a mock Apex wire adapter, an outline for a test) and generate something I could work with. 

![Setup Mock Wire Adapter](/assets/img/posts/agentforce-test-generator/setup-mock-wire-adapter.png)

This time, we made a bit more progress! While the test generator ignored my "outline" test, it created a new test that has the same general idea. 

![Test Generation After Setup](/assets/img/posts/agentforce-test-generator/test-gen-after-setup.png)

The one thing this test does well is that it actually emits data from the wire adapter. Unfortunately, it does not verify that data in any meaningful way. We can make changes to the assertions in this test to generate our first passing test. 

![First Passing Test](/assets/img/posts/agentforce-test-generator/first-passing-test.png)

Now that I had a passing test (that hopefully gives the Dev Assistant some hints about how to work with my LWC), I wanted to try generating a new test one more time. Let's see how it does.

![Final Test Generation Attempt](/assets/img/posts/agentforce-test-generator/final-test-gen-attempt.png)

Unfortunately, this test is meaningless. There are no "default values" aside from the custom settings themselves, so this test fails to check anything that actually related to our component. 

### **Where the Dev Assistant Could Be Helpful**

After wrapping up my thoughts in this article and buckling down to write my tests from scratch, I discovered one area where the Dev Assistant's test generator could actually be useful. Take the following example:

![Hides Schedule Hour Test](/assets/img/posts/agentforce-test-generator/hides-schedule-hour-test.png)

This test ensures that when the user inputs `hours` as the units option on the page, the `Schedule Hour` input section of the page is hidden. When we run the test generator after writing this test, it gives us the following:

![Shows Schedule Hour Test](/assets/img/posts/agentforce-test-generator/shows-schedule-hour-test.png)

This test is written correctly and passes! The Dev Assistant also generated a couple of other tests for `weeks` and `months` options, which are not meaningful in our case, since the only unit options are `days` and `hours`. Regardless, this is the first fully-functioning test that we've receieved from the Dev Assistant that doesn't need any tweaks to pass.

### **Final Takeaways**

In conclusion, I think AgentForce Dev Assistant can be a really helpful tool for the following parts of testing:
- Generating a test file from scratch so that you have something to start with
- Writing "inverse" tests after you have already tested one side of a condition

It was _not_ helpful with the following parts of testing:
- Generating tests from scratch using information from the Javascript and HTML files
- Writing mock data files
- Mocking apex wire adapters within the test

### **Helpful Resources**

Since I was approaching this project as a beginner and the Dev Assistant wasn't able to help me through some of the more technical parts of writing tests, I had to do some other research to piece together my Jest tests. The following two resources were super helpful and I think any beginner could learn a lot starting here:
- [Lightning Web Component Tests](https://trailhead.salesforce.com/content/learn/modules/test-lightning-web-components){:target="_blank"} - this module on trailhead has all you need to get started with understanding Jest tests for LWCs, with a couple of basic examples.
- [Lightning Web Components Recipes](https://github.com/trailheadapps/lwc-recipes){:target="_blank"} - this repository has loads of LWC examples, and many of them have test files included as well. This is a great place to search for a wire adapater or any other built-in function that you want to test ([`getRecord`](https://developer.salesforce.com/docs/platform/lwc/guide/reference-wire-adapters-record.html){:target="_blank"}, for example) and follow how the test is generally set up.
