---
layout: post
title:  "Salesforce Code Analyzer: From Local Checks to GitHub Automation"
date:   2025-05-01 07:00:00 -0400
categories: salesforce cli sf code analyzer github actions
author: Tamara Chance
comments: true
image: assets/img/stockImages/salesforce-github.svg
---
Salesforce Code Analyzer is a static analysis tool that scans your Apex, Lightning Web Components (LWC), and Visualforce code to detect potential issues, enforce coding standards, and improve code quality. By using it with your development workflow, you can catch problems early and ensure consistent code quality across your team.

It’s also a requirement for [developing apps for the AppExchange](https://duelfersolutions.com/salesforce/agentforce/ai/2025/04/30/lessons-learned-road-to-appExchange.html), which is how I was first introduced to this tool. This guide will walk you through setting up and using Salesforce Code Analyzer both locally in Visual Studio Code (VS Code) and integrating it into your GitHub workflow for automated analysis.
### **Prerequisites**
Before getting started, ensure you have the following:
- An SFDX project opened in VS Code
- GitHub Actions enabled in GitHub
- Familiarity with branches and basic GitHub navigation

### **Running Salesforce Code Analyzer Locally in VS Code**
Local code quality checks help you catch issues before committing or pushing to your repository. _The Salesforce Code Analyzer plugin + VS Code = fast feedback!_ Here are the steps to get started running scans locally and interpreting results in VS Code.
#### **Installation**
- **Install Salesforce CLI:** Download and install from the [Salesforce CLI website](https://developer.salesforce.com/tools/sfdxcli). If you already have the Salesforce CLI, note that the following instructions are using the sf(V2) commands. Need help getting set up with sf(V2)? [Check out my simple setup guide](https://sfdxdeveloper.com/salesforce/cli/sfdx/sf/deploy/2024/11/04/mastering-sf-cli-commands.html). 
- **Install the Code Analyzer Plugin:** Open the terminal in VS Code and run: `sf plugins install @salesforce/sfdx-scanner`

This command installs the Code Analyzer plugin globally, enabling you to run code scans directly from the CLI.
#### **Running a Code Scan**
With the setup complete, you can now run code scans.

To scan the entire project, run: `sf scanner run --target force-app --format table --engine pmd --engine eslint`

This command scans the force-app directory using both ESLint and PMD engines, displaying the results in a table format.
#### **Interpreting Results**
After running a scan, the results will be displayed in the terminal in VS Code. Each issue will include details such as the rule violated, severity, and a description to help you understand and fix the problem.
#### **Customizing the Analyzer**
You can customize the behavior of the Code Analyzer by using additional flags. For example, to focus on security-related rules, set the category to "Security":
- `--category`: limit rules by category (e.g., Security, Performance, etc.)
- `--engine:` choose specific rule engines like `eslint`, `pmd`, `retire-js`
- `--ruleset`: point to a custom `.xml` ruleset file (for PMD)
- `--format`: output type — like `table`, `json`, or `csv`
- `--target`: specify which files/folders to scan

### **Automating Code Analysis with GitHub Actions**
Now that we’ve covered local development, let's look at how you can use this tool with your whole team. Integrating Salesforce Code Analyzer into your GitHub workflow can help you _level-up_ your CI/CD devOps. It ensures that code is automatically analyzed during pull requests and merges.

Note that there is no need to install Salesforce Code Analyzer locally for this setup!

#### **Setting Up the GitHub Workflow**
1. **Create a Workflow File:** In your repository, create a new file at `.github/workflows/code-analyzer.yml`.
2. **Add the Workflow Configuration:**

Yaml
```
name: Salesforce Code Analyzer
on:
  workflow_dispatch:
  pull_request:
    types: [opened, edited, synchronized, reopened]
jobs:
  salesforce-code-analyzer-workflow:
    runs-on: ubuntu-latest
    steps:
      - name: Check out files
        uses: actions/checkout@v4


      - name: Install Salesforce CLI
        run: npm install -g @salesforce/cli@latest


      - name: Install Salesforce Code Analyzer v4.x CLI Plugin
        run: sf plugins install @salesforce/sfdx-scanner@latest


      - name: Run Salesforce Code Analyzer
        id: run-code-analyzer
        uses: forcedotcom/run-code-analyzer@v1
        with:
          run-command: run
          run-arguments: --normalize-severity --target . --outfile results.html
          results-artifact-name: salesforce-code-analyzer-results


      - name: Check the outputs to determine whether to fail
        if: |
          steps.run-code-analyzer.outputs.exit-code > 0 ||
          steps.run-code-analyzer.outputs.num-sev1-violations > 0 ||
          steps.run-code-analyzer.outputs.num-violations > 10
        run: exit 1
```
This workflow triggers on every pull request, checks out the code, installs the Code Analyzer, and runs the analysis. You can set the trigger to whatever is most useful (on push, on pull request, only on specific branches, etc).
#### **Running the Workflow**
Once the workflow file is added:
- **Automatic Triggers:** The workflow, in this case, will run automatically on every pull request to the repository. 
- **Manual Triggers:** You can also manually trigger the workflow from the "Actions" tab in your GitHub repository. This allows you to test the workflow without having to open a PR as long as the workflow exists on the branch you are testing. Ensure you have the `workflow_dispatch` trigger listed in your yaml file if you want to run the manual trigger.

#### **Viewing Results**
After the workflow runs, you can view the results:
1. Navigate to the "Actions" tab in your GitHub repository.
2. Select the latest workflow run.
3. Review the logs and any artifacts generated by the Code Analyzer. I prefer to download the artifact and open it in my browser for added readability.

### **Comparing Local and CI Workflows**
Using both local and CI workflows ensures comprehensive code quality checks. Local analysis provides immediate feedback during development, while GitHub Actions enforce standards across the team. Here’s a breakdown of the benefits of both tools.

| **Feature** | **Local (VS Code)** | **GitHub Actions** |
| :-----------------------------: | :----------------: | :-----------------: |
| Immediate Feedback | ✅ | ❌ |
| Integration with Editor | ✅ | ❌ |
| Automated on Pull Requests | ❌ | ✅ |
| Ensures Team-wide Code Quality | ❌ | ✅ |
| Customizable Analysis Settings | ✅ | ✅ |

And that’s it! You’re all set to start using Salesforce Code Analyzer to enhance your code quality and security. By integrating it into your local development environment and your CI/CD pipeline, you can catch issues early and maintain high standards across your projects.

Start by setting up the analyzer in VS Code for immediate feedback, and then integrate it into your GitHub workflows.
### **Additional Resources**
- [Salesforce Code Analyzer Documentation](https://developer.salesforce.com/docs/platform/salesforce-code-analyzer/overview)
- [Salesforce CLI Installation Guide](https://developer.salesforce.com/tools/sfdxcli)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Run Salesforce Code Analyzer GitHub Action](https://github.com/marketplace/actions/run-salesforce-code-analyzer)