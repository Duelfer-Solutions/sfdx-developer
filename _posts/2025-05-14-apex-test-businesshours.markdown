---
layout: post
title:  "Testing BusinessHours in Apex"
subtitle: "The Painless Way Salesforce Forgot to Tell You"
date:   2025-05-14 07:00:00 -0400
categories: salesforce apex test businesshours
author: Tamara Chance
comments: true
image: assets/img/stockImages/clock-calendar.svg
---
Salesforce gives us a lot to be thankful for when it comes to working with time-based logic—and the BusinessHours class is one of those hidden gems. It’s built right into the platform, ready to help you calculate business time across all kinds of use cases: SLA tracking, escalation logic, aging calculations, and more. No need to reinvent the wheel.

The BusinessHours class includes a few key static methods:

- [`BusinessHours.add(businessHoursId, startDate, intervalMilliseconds)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_add)
- [`BusinessHours.addGmt(businessHoursId, startDate, intervalMilliseconds)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_addGmt)
- [`BusinessHours.diff(businessHoursId, startDate, endDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_diff)
- [`BusinessHours.isWithin(businessHoursId, targetDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_isWithin)
- [`BusinessHours.nextStartDate(businessHoursId, targetDate)`](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm#apex_System_BusinessHours_nextStartDate)

Each one handles the heavy lifting of Salesforce’s Business Hours configuration for you—including time zone alignment, holidays, and working hours. 

This blog post is for Salesforce developers who find themselves wrangling with BusinessHours methods in Apex tests—and want a clean, reliable, and drama-free approach to making it testable. I'll be focusing on the `BusinessHours.diff()` method, but the solution is applicable to all of the methods.

### _Why Testing BusinessHours in Apex is So Hard_
At first glance, `BusinessHours.diff()` seems like a dream—it returns the number of business milliseconds between two dates. Just pass in your values, and boom: your math is done.

But then you try to write an Apex test.

Suddenly, that simple, beautiful method becomes a black box. You search the [Salesforce Developer Documentation for BusinessHours](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_classes_businesshours.htm), hoping for guidance on how to test this. You scroll. You CTRL+F. You check again.

Nothing. No section on testing. No warning that you can’t create custom Business Hours records in tests.

You try injecting fake business hours, since BusinessHours technically supports the `create()` call. 
```
BusinessHours bh = new BusinessHours();
bh.Name = 'Weekday Business Hours';
bh.IsActive = true;
bh.IsDefault = false;
bh.TimeZoneSidKey = 'America/New_York';
bh.MondayStartTime    = Time.newInstance(9, 0, 0, 0);
bh.MondayEndTime      = Time.newInstance(17, 0, 0, 0);
bh.TuesdayStartTime   = Time.newInstance(9, 0, 0, 0);
bh.TuesdayEndTime     = Time.newInstance(17, 0, 0, 0);
bh.WednesdayStartTime = Time.newInstance(9, 0, 0, 0);
bh.WednesdayEndTime   = Time.newInstance(17, 0, 0, 0);
bh.ThursdayStartTime  = Time.newInstance(9, 0, 0, 0);
bh.ThursdayEndTime    = Time.newInstance(17, 0, 0, 0);
bh.FridayStartTime    = Time.newInstance(9, 0, 0, 0);
bh.FridayEndTime      = Time.newInstance(17, 0, 0, 0);

insert bh;
```
_"Nope. You can’t insert those records in a test context."_ Totally useless. Salesforce blocks it.

You scour forums. Trailblazer posts hint at answers, but most are incomplete. You try stubbing the `BusinessHours.diff()` method. _But_, all of the BusinessHours methods are **static methods**—and static methods in Apex **can’t be stubbed**.

You even start wondering if the moon phase is affecting your assertions. Your confidence in your code erodes. You pause and think: _“Maybe this just isn’t testable.”_

🙈 _Spoiler alert:_ The problem wasn’t with stubbing. You just can’t stub a static method.

**But you can stub an instance of your own service class method—one that wraps the BusinessHours static method call.**

That’s the twist. That’s the way forward. You just need to _wrap_ the static method in a custom class.
### _Stub Your Own Service Class, Not Salesforce's_
The turning point is this: stop trying to test Salesforce’s static logic. Start testing your own.
#### **Step 1: Create a service class wrapper**
Create a simple service class wrapper that will call the BusinessHours static method like this:

```apex
public class BusinessHoursService {
    public Long diff(Id businessHoursId, Datetime startDate, Datetime endDate) {
        return BusinessHours.diff(businessHoursId, startDate, endDate);
    }
}
```

Then, instead of calling BusinessHours.diff() directly in your core logic like this:

```apex
public class MyHelperClass {
    public static void doSomethingOverThreshold(Case c) {
        Integer threshold = 86400000; //24hrs
        Long caseAge = BusinessHours.diff('0000000000000', c.CreatedDate, Datetime.now());
        if (caseAge > threshold) {
            // do something
        }
    }
}
```
...wrap it in an instantiation of your service class like this:

```apex
public class MyHelperClass {
    BusinessHoursService bhService;

    public MyHelperClass(BusinessHoursService bhService) {
        this.bhService = bhService;
    }

    public static void doSomethingOverThreshold(Case c) {
        Integer threshold = 86400000; //24hrs
        Long caseAge = bhService.diff('0000000000000', c.CreatedDate, Datetime.now());
        if (caseAge > threshold) {
            // do something
        }
    }
}
```
Now you have something you _can_ stub—because Apex supports stubbing instance methods, not static ones. 

Here’s how you do it:
#### **Step 2: Create your Stub**
As the [Apex documentation](https://developer.salesforce.com/docs/atlas.en-us.254.0.apexcode.meta/apexcode/apex_testing_stub_api.htm) explains:

> “We want to return a constant, predictable value to isolate our testing . . . rather than writing a 'fake' version of the class, where the method returns a constant 
> value, we create a stub version of the class. The stub object is created dynamically at runtime, and we can specify the 'stubbed' behavior of its method.”

```apex
public class BusinessHoursServiceStub implements System.StubProvider {
    public Object handleMethodCall(Object stubbedObject, String stubbedMethodName, System.Type returnType, 
        List<System.Type> listOfParamTypes, List<String> listOfParamNames, List<Object> listOfArgs) {
        if (returnType.getName() == 'Long') {
            return 86400000; // 24 hours in milliseconds
        } else {
            return null;
        }
    }
}
```
In this case, we want to return a constant, predictable value so we can isolate our testing to the business logic—not the behavior of BusinessHours.diff() itself. This approach gives us clean, controlled test conditions.
#### **Step 3: Stub the service in your test**
Now in your test class you can call your stub.
```apex
Case c = new Case(Subject = 'Test');
BusinessHoursService stub = (BusinessHoursService) Test.createStub(
    BusinessHoursService.class,
    new BusinessHoursServiceStub()
);
MyHelperClass mhc = new MyHelperClass(stub);
mhc.doSomethingOverThreshold(c);
System.assert(true, 'Verify any threshold side-effects occur.');
```
No more trying to fake BusinessHours records. No more relying on _real data_ in tests. No more brittle tests held together with duct tape and hope.

- ✅ You test the logic that matters
- ✅ You return predictable values
- ✅ You stop testing Salesforce internals

You _can_ actually test your BusinessHours dependent logic--without pain. For more tips & tricks check out some of our [other articles]({{ site.url }}). Happy Coding!