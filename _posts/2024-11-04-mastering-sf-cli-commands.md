---
layout: post
title:  "Mastering the New Salesforce sf CLI Commands"
date:   2024-11-04 07:00:00 -0400
categories: salesforce cli sfdx sf deploy
author: Tamara Chance
comments: true
image: assets/img/stockImages/sf-cli-commands.png
---
The Salesforce CLI is a cornerstone tool for Salesforce developers, facilitating tasks such as building, testing, deploying, and more. As of July 2023, Salesforce released a new version of the CLI, known as sf (v2). This update marks a significant evolution in how developers will interact with Salesforce's command-line tools.

But many developers still haven't made the transition over a year later. The sfdx commands (while deprecated) continue to work, which has led a lot of developers to procrastinate the switch. Following the Summer '24 release and the 1-year anniversary of sf (v2), however, Salesforce announced that the deprecated sfdx source/mdapi/org commands will be removed as of November 6th, 2024. So now is as good a time as any to learn the new commands. The goal of this blog post is to guide you through the transition to help you quickly get up to speed with the new commands and make the most of the enhanced features.
## Why Salesforce CLI Matters
The CLI is integral to modern Salesforce development, especially in environments where continuous integration and continuous deployment (CI/CD) practices are employed. Its ability to automate tasks, manage orgs, and deploy changes quickly and efficiently makes it an indispensable tool in the Salesforce ecosystem.
## Setting Up the New CLI
To get started with sf (v2), you'll first need to uninstall sfdx (v7) if it's currently installed on your machine. Here's a link to help you [uninstall sfdx (v7)](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_uninstall.htm). And be sure to check for multiple installations on your computer. You may need to manually delete `.sfdx` file paths as well as errant `sfdx` subfolders. This is because sf (v2) uses the sfdx alias, and having both installed simultaneously will cause conflicts. 

Once sfdx is uninstalled, you can proceed with installing sf (v2) by running the `npm install -g @salesforce/cli` command. Here's a link to the full instructions for [moving from sf (v2)](https://developer.salesforce.com/docs/atlas.en-us.252.0.sfdx_setup.meta/sfdx_setup/sfdx_setup_move_to_sf_v2.htm) using NPM, Windows, or Linux. The installation process is pretty straightforward with NPM if you have Node.js already downloaded onto your computer. If you encounter any issues related to dependencies, follow the prompts in the CLI to audit and fix any issues.
## Top 10 Most Frequently Used Commands and Their Use Cases
Once you've successfully installed sf (v2), it's time to start using the new commands. Here, I've compiled a list of _my_ top 10 most frequently used commands, along with the corresponding flags. These commands cover a range of essential tasks from org management to deployments, making them invaluable tools for any Salesforce developer.
### 1. sf org open
Opens your default org. Use the `-o` or `--target-org` flags to open an org other than the default or under a specific profile. This replaces the previous `-u` flag.
### 2. sf org list
Quickly view a list of all authorized orgs. This command is helpful for managing multiple environments, especially when switching between scratch orgs. There's even little icons to denote DevHubs and Default orgs. Use the `--all` flag to see a full list of authenticated orgs and all scratch orgs.
### 3. sf project deploy start
Deploy metadata from your local project to a target org. I often use this command for deploying changes I've made in the .xml files into my scratch org, but this is also the command to deploy between environments. Add the `-m`, `--metadata`, `-d`, or `--source-dir` flags when you want to deploy specific components or files. This command replaces the old `force:source:push` and `force:source:deploy` commands. Add `-c` to ignore conflicts and `-g` to ignore warnings.
### 4. sf project retrieve start
Retrieve metadata from a target org into your local project. This command is the compliment for the above command for pulling down changes made directly in the org, ensuring that your local environment is up to date. It replaces the `force:source:pull` and `force:source:retrieve` commands. Use the same flags listed above for retrieving specific metadata.
### 5. sfdx org create scratch
Spin up a new scratch org for development or testing. This command creates isolated environments for different features or user stories. Use the following flags: `-f/--definition-file`, `-d/--set-default`, `-y/--duration-days`, `-a/--alias`.
### 6. sf org login web
Authenticate with an org. The `--instance-url` flag still applies.
### 7. sf config set target-org <alias>
Switch default orgs. If you don't specify a target org, the sf (v2) commands use the default org. But you can use this command to switch which org is set as your default. No flags required.
### 8. sf apex run test
Run Apex tests in a target org. I often run this command before deployments to ensure that my code changes don't break existing functionality. Use the `-n` or `t` flag to speficy the class names or specific tests. The catch here is that you can't use the array format anymore. Instead, you'll need to add the flag before each class name or test name. If you want to see the results displayed in your terminal, use the `-r <format>` and the `y/--synchronous` flags.
### 9. sf org delete scratch
Delete a scratch org that is no longer needed. This command helps keep my list of scratch orgs clean and manageable by removing outdated or unused environments. Append the `-o <alias>` flag to the command to specify which org to delete.
### 10. sf package install
Run this command to install packages. Use the same flags to add the `-p/` package and `-k/--installation-key`. To see the list of installed packages run `sf package installed list`. 

For a comprehensive list of commands, see this [sfdx to sf command mapping guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_old_new_command_mapping.htm) from Salesforce. I recommend using CTRL+F to search for the sfdx command that you need. Just remember that this page doesn't include the flags.

## Tips for Transitioning
Transitioning from sfdx to sf (v2) is designed to be as smooth as possible. You can still use the `sfdx` in front of all of the sf (v2) commands, and it will still recognize and execute them as expected. The same is true for adding the colon vs the space: `sf org open` or `sf org:open`. Over time, as you become more comfortable with the new structure, you will start to build up a muscle memory of the new commands.

Perhaps you've been seeing the deprecation warnings, already. When you run an `sfdx` command, the `sf` equivalent is displayed in a warning. You can use these warnings to start practicing the new versions of commands that you regularly use. If the deprecation warnings are no longer showing up for you, you can also run any `sfdx` command with the `--help` flag to see the `sf` equivalent.

## Recap
The release of sf (v2) marks a significant advancement in Salesforce CLI capabilities. It's here to stay, and it's a tool that every Salesforce developer should embrace. As you begin working with sf (v2), make sure to bookmark this page. It’s a great resource to keep handy as you explore the new commands and transition your workflows. Stay tuned for future updates and enhancements to sf (v2). Salesforce is committed to continuously improving the CLI, and keeping up with these changes will ensure you’re always using the best tools available.