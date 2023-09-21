---
layout: post
title:  "Retrieving Dashboard Metadata"
date:   2023-09-11 05:00:00 -0400
categories: salesforce metadata
author: Tamara Chance
comments: true
image: assets/img/stockImages/metadata-doodles.png
---
Dashboards in Salesforce allow you to visualize your data and gain insights into your organization's performance. Even more valuable is the ability to deploy dashboards from one org to another. 

If you’re familiar with Salesforce’s Metadata API, this process seems simple enough. Retrieve the Dashboard and the dashboards folder using the `sfdx force:source:retrieve -m` command, right?

Let’s assume you have a dashboard named _MyDashboard_. You might try something like this: `sfdx force:source:retrieve -m Dashboard:MyDashboard`. You may even try just getting all the dashboards `sfdx force:source:retrieve -m Dashboard` in the org. But despite your attempts nothing works. 

If this scenario sounds all too familiar, here’s a simple technique for retrieving Dashboard Metadata. TL;DR: The name you gave your dashboard isn’t its real API name. The real API name is a weird, system generated hash.

### Step 1: Identify Your Dashboard

First things first, you need to identify the dashboard you want to pull. The big mistake here is thinking that the name you gave to your dashboard corresponded with its `DeveloperName`. 

It doesn’t. **The API name for dashboards are system generated strings that don’t show up anywhere in the UI.** The name you assigned to your dashboard is actually its `Title`.

In order to find your dashboard’s actual `DeveloperName`, run the following SOQL query:

`SELECT Id, DeveloperName, FolderName FROM Dashboard WHERE Title='MyDashboard';`

Now you can see your dashboard’s real `DeveloperName`. It will be something like 'WdD0FHnDDbudSkdJp'. Remember this, as you'll need it in the following steps.

### Step 2: Retrieve the Dashboard

Now comes the slightly tricky part. Not only do you need to know your dashboard’s real `DeveloperName`, you also need to include the `FolderName` as part of your retrieval command.

Assuming your dashboard is in a folder called 'MyDashboardFolder', here's how you can retrieve it:

`sfdx force:source:retrieve -m 'Dashboard:MyDashboardFolder/WdD0FHnDDbudSkdJp'`

Remember to remove any spaces in your `FolderName` or replace them with underscores. My Dashboard Folder becomes MyDashboardFolder or My_Dashboard_Folder. Either one will work.

### Step 3: Add to package.xml (Optional)

If you prefer working with a package.xml file, you can also add the dashboard to it. Open your package.xml and include the following lines:

``` xml
<types>
    <members>MyDashboardFolder</members>
    <members>MyDashboardFolder/WdD0FHnDDbudSkdJp</members>
    <name>Dashboard</name>
</types>
```

Again, replace  or remove any spaces in your dashboard's actual folder name.

### Step 4: Success!

Execute the SFDX command or deploy your changes using the updated package.xml, and you will have successfully moved your dashboard and the dashboard folder metadata.

### Special Considerations

 - Unfortunately, you cannot change the name of your dashboard. 

 - This example references the CLI commands one might use for source-driven deployments. If you are utilizing Change Sets, you won’t need to apply the retrieval commands, but you will still need to query for the `DeveloperName`.

I hope these steps save you a headache and help you to seamlessly integrate your Salesforce dashboards into your source code management process. 

Happy coding!
