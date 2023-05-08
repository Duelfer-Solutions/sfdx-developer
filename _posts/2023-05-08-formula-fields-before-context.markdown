---
layout: post
title:  "Working with Formula Fields in the Before Context of an Apex Trigger - Don't Make This Mistake!"
date:   2023-05-08 10:30:00 -0400
categories: salesforce apex
comments: true
---

Formula fields in Salesforce can be used to quickly calculate values from other fields on an object and to create custom values based on specific conditions, thus making Salesforce more powerful and flexible. However, if you're not careful when working with them in the before-context of an Apex trigger, you may experience unexpected results due to the runtime of formula fields.

In this blog post, we'll walk you through a case study that highlights how to get around the issue of formula fields in Apex triggers not evaluating correctly when values aren’t yet committed to the database.

Let's get started!

## Case Study Scenario:

Let's say that a retail business needs to populate a `Total_Value__c` field on each Opportunity based on a calculation that adds together the values from three other fields also on the Opportunity object. This calculation happens in the before context of an update trigger on the Opportunity object.

Prior to the calculation, the fields on the Opportunity object are overwritten by a custom object that tracks proposals, called Proposal__c. Whenever a proposal is saved, it triggers the update for the Opportunity. 

This trigger calls another class that contains the logic to overwrite the three field values on the Opportunity based on data from a related proposal. Here’s an example of what the class in our ficitional scenario might look like:

```apex
/*
 * This class's purpose is to automate Opportunity values based on proposal data.
 */

public class CalculateValues {
    
    public static void setProposalData(OpportunityTriggerHandler.OpportunityContext ctx) {
        setDataFromProposal(ctx); //Fields are mapped here, then
        setTotalValueMetric(ctx); //Total Value is calculated.
    }

    private static void setDataFromProposal(OpportunityTriggerHandler.OpportunityContext ctx) {

        ctx.opp.Unit_Price__c = ctx.proposal.Unit_Price__c;
        ctx.opp.Quantity__c = ctx.proposal.Quantity__c;
        ctx.opp.Comped_Quantity__c = ctx.proposal.Comped__c;
        ctx.opp.Company_Transportation_Cost__c = ctx.proposal.Company_Transportation_Cost__c;
        ctx.opp.Uncategorized_Transportation_Costs__c = ctx.proposal.Uncategorized_Transportation_Cost__c;
        ctx.opp.Waste_Cost_Margin__c = ctx.proposal.Waste_Cost_Margin__c;
        
    }

    public static void setTotalValueMetric(OpportunityTriggerHandler.OpportunityContext ctx) {
        ctx.opp.Total_Value__c = calculateTotalValue(ctx.opp);
    }

    private static Decimal calculateTotalValue(Opportunity opp) {
        Decimal totalValue = 0.0;
        if (opp.Total_Order__c != null && opp.Unit_Price__c != null) { 
            totalValue += opp.Total_Order__c;
        }
        if (opp.Total_Transportation_Costs__c != null) {
            totalValue -= opp.Total_Transportation_Costs__c;
        }
        if(opp.Waste_Cost_Margin__c != null) { 
            totalValue -= (totalValue * opp.Waste_Cost_Margin__c);
        }
        return totalValue;
    }
}
```

Whenever a Proposal__c record is saved, the Opportunity trigger runs and maps the values from the Proposal__c to the Opportunity object in the setDataFromProposal method.

Then, the context is passed to the setTotalValueMetric method. The calculateTotalValue method returns the value that will be set to the Total_Value__c field on the opportunity. 

The first two fields in the calculateTotalValue calculation, Total_Order__c and Total_Transportation_Cost__c, are both formula fields. 


![Depicts the formulas for the Total Order and Total Transportation Cost fields in the calculateTotalValue method.](/assets/img/posts/formula-fields-before-context/calcTotalValue.png)

Here’s an example test class for this calculation that asserts what you might expect the results for the Total_Value__c fields to be based on the above code:

```apex
@isTest
private class CalculateValues_Test {
    
    private static Account acc;
    private static Opportunity opp;
    
    private static void init() {
        acc = new Account(Name = 'TestAccount');
        insert acc;
        opp = new Opportunity(Name = 'Test Opp', AccountId = acc.Id, StageName = 'Qualification', CloseDate = Date.today().addMonths(2));
        insert opp;
    }
    
    @isTest
    private static void calculateValuesTest() {
        
        init();
        
        Test.startTest();
        
        Proposal__c prop1 = new Proposal__c(Opportunity__c = opp.Id);
        prop1.Unit_Price__c = 1.50;
        prop1.Quantity__c = 50000;
        prop1.Comped__c = 1400;
        prop1.Company_Transportation_Cost__c = 4589.76;
        prop1.Uncategorized_Transportation_Cost__c = 780.22;
        prop1.Waste_Cost_Margin__c = 0.005;
        insert prop1;
        
        opp.Accepted_Proposal__c = prop1.Id;
        update opp;
        opp = getOpp(opp.Id);
        System.assertEquals(1.50, opp.Unit_Price__c);
        System.assertEquals(50000, opp.Quantity__c);
        System.assertEquals(1400, opp.Comped_Quantity__c);
        System.assertEquals(4589.76, opp.Company_Transportation_Cost__c);
        System.assertEquals(780.22, opp.Uncategorized_Transportation_Costs__c);
        System.assertEquals(0.005, opp.Waste_Cost_Margin__c);
        System.assertEquals(((50000 - 1400) * 1.50) - (4589.76 - 780.22) - ((((50000 - 1400) * 1.50) - (4589.76 - 780.22))*0.005), opp.Total_Value__c);
        
        Test.stopTest();
    }
    
    private static Opportunity getOpp(Id oppId) {
        return [SELECT Id, Total_Order__c, Total_Value__c, Total_Transportation_Costs__c, Quantity__c, 
                Comped_Quantity__c, Company_Transportation_Cost__c, Uncategorized_Transportation_Costs__c, 
                Waste_Cost_Margin__c, Unit_Price__c
                FROM Opportunity WHERE Id = :oppId];
    }
}
```

But when the test class runs….

![A screenshot of the test class failing. Assertion expected 68745.00770, but the actual value was 0.00.](/assets/img/posts/formula-fields-before-context/failed-assertion.png)

…the assertion fails. So what’s the problem?

The newly mapped opportunity fields are required as inputs to the formula fields being used in the calculateTotalValue calculation. 

The formula fields for Total_Order__c and Total_Transportation_Cost__c ran when the trigger was first called. Before the mapping occurred and was committed to the database. 

Since formulas rely on data that’s already been committed to the database, the formula fields can’t see the new values from the proposal object until after save.

The Total Value Calculation evaluated correctly, but with incorrect data so that our assumptions were incorrect.

After update, if you were to run the calculation again with no additional changes made to the underlying fields, the formula fields would reflect correct values and the Total Value calculation would be as expected. 

While it’s great to know that the formula does work, it doesn’t help us to make sure that our calculation is evaluating as expected in the before-context.

## The Solution:

The good news is there are some strategies you can use to get around this limitation of the formula fields. Here’s an example of a manual calculation strategy to do just that:

```apex
/*
 * This class's purpose is to automate Opportunity values based on proposal data.
 */

public class CalculateValues {
    
    public static void setProposalData(OpportunityTriggerHandler.OpportunityContext ctx) {
        setDataFromProposal(ctx);
        setTotalValueMetric(ctx);
    }

    private static void setDataFromProposal(OpportunityTriggerHandler.OpportunityContext ctx) {
        ctx.opp.Unit_Price__c = ctx.proposal.Unit_Price__c;
        ctx.opp.Quantity__c = ctx.proposal.Quantity__c;
        ctx.opp.Comped_Quantity__c = ctx.proposal.Comped__c;
        ctx.opp.Company_Transportation_Cost__c = ctx.proposal.Company_Transportation_Cost__c;
        ctx.opp.Uncategorized_Transportation_Costs__c = ctx.proposal.Uncategorized_Transportation_Cost__c;
        ctx.opp.Waste_Cost_Margin__c = ctx.proposal.Waste_Cost_Margin__c;        
    }

    public static void setTotalValueMetric(OpportunityTriggerHandler.OpportunityContext ctx) {
        ctx.opp.Total_Value__c = calculateTotalValue(ctx.opp);
    }

    private static Decimal calculateTotalValue(Opportunity opp) {
        Decimal totalValue = 0.0;
        if (opp.Total_Order__c != null && opp.Unit_Price__c != null) { 
            totalValue += calcTotalOrder(opp);
        }
        if (opp.Total_Transportation_Costs__c != null) {
            totalValue -= calcTotalTransportationCost(opp);
        }
        if(opp.Waste_Cost_Margin__c != null) { 
            totalValue -= totalValue * opp.Waste_Cost_Margin__c;
        }
        return totalValue;
    }

    private static Decimal calcTotalOrder (Opportunity opp){
        Decimal totalOrder = 0.0;
        if(opp.Comped_Quantity__c != null){
            totalOrder += (opp.Quantity__c - opp.Comped_Quantity__c) * opp.Unit_Price__c;
        }else{
            totalOrder += opp.Quantity__c * opp.Unit_Price__c;
        }
        return totalOrder;
    }

    private static Decimal calcTotalTransportationCost (Opportunity opp){
        Decimal totalTransportationCost = 0.0;
        if(opp.Uncategorized_Transportation_Costs__c != null){
            totalTransportationCost += opp.Company_Transportation_Cost__c - opp.Uncategorized_Transportation_Costs__c;
        }else{
            totalTransportationCost += opp.Company_Transportation_Cost__c;
        }
        return totalTransportationCost;
    }
}
```

While the formula fields rely on data that has already been committed to the database, the new values mapped by the setDataFromProposal method are available to use in the calculateTotalValue calculation.

Instead of using the formula fields, the manual calculation breaks out the fields inside of the formula field and performs the calculation from each formula inside of two helper functions. 

Now if we take that same test class from earlier and run it with our manually calculated Total Value calculation…

![A screenshot of the debug log showing the correct Total Value of 68745.00770](/assets/img/posts/formula-fields-before-context/correctTotalValue.png)

…the test now passes and the Total_Value__c field shows an accurate value.

## Closing Thoughts

It's essential to be mindful when working with formula fields in Salesforce, especially within triggers. Here are some tips and best practices to keep in mind when you're working in the before context and using formula fields:

- Formula fields rely on data that is already committed to the database.
- Make sure to perform testing to ensure your calculations are accurate and don't rely on any incorrect data.
- If you run into an unexpected result, try breaking out your formula into its individual fields and performing the calculation manually.

​
I hope this blog post was helpful in understanding how to work around issues with formula fields in the before context. With the right approach, you'll be able to achieve accurate results in not time! I’ve included only the necessary code snippets here to illustrate the problem. If you’d like to look at the full code including the trigger handler, here’s a link to the [gist](https://gist.github.com/tamarachance/e44db2f1ffcc04e1a47a10507d0f1f9b.js). If you're looking for more information on Salesforce best practices and development, be sure to check out our other [blog posts](sfdxdeveloper.com).

Ready to grow your business? For custom Salesforce solutions tailored to your business needs, contact our team of experts today!