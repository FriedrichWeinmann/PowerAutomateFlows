# Trigger Conditions

Looking back, this seems like an obvious one, but ... _Work With Trigger Conditions Whenever You Can._
Seriously, this can save so much processing costs and makes it easier to separate flows into smaller units.

Example Scenario:
We want to use Power Automate to manage our office calendar (something mailbox rules cannot).
Now, there may be many things we want to automate - remove reminders from emails outside of working hours, disregard events from certain sources, automatically forward certain invites to another account, ... the possibilities are endless.
With Power Automate, we can now do one of the following:

+ Create a Flow for each scenario and use conditions within the flow. This means every single meeting invite is going to trigger potentially dozens of flows and cost quite a bit.
+ Create a single Flow and have conditions for each case inside. One execution per meeting invite, but a nightmare to maintain
+ Create a Flow for each scenario and use Trigger Conditions. These will be evaluated before the flow is started, so only applicable flows execute while keeping our flows very clean.

So ... while they may not be the most convenient to design, Trigger Conditions offer massive advantages over the alternatives and I strongly recommend investing into them.
