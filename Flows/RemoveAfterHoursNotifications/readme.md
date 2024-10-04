# Remove After Hours Notifications

## Description

So, you work at an international company and you get meeting invites for 2am meetings?
Time to remove those annoying popups before they steal your sleep!

This is a fundamentally simple flow:

+ Trigger: Outlook - When a new event is created
+ Action 1: Outlook - Update event

Using TriggerConditions we make sure only calendar entries outside of business hours are affected.
We also trigger only on creation of a new event, so we can still manually add that back for events we care about.

## Trigger

So here's the trigger definition:

```text
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "table": "AQMkAGQ2NgA4ZGY1ZC04OTc4LTQzZTUtODE2YS00NWE4M2FkYzJjNTIARgAAA2GRZj69MutEmUAsK9ZYZUcHAA_xxxxxxxxxxxxxx_L1z7UAAAIBBgAAAA_4ckB2ou1CoXm5U_L1z7UAAAIVTgAAAA=="
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connection": "shared_office365",
      "operationId": "CalendarGetOnNewItemsV3"
    }
  },
  "recurrence": {
    "frequency": "Minute",
    "interval": 1,
    "startTime": null,
    "timeZone": null
  },
  "conditions": [
    {
      "expression": "@equals(triggerOutputs()?['body/isReminderOn'], true)"
    },
    {
      "expression": "@or(less(int(formatDateTime(triggerOutputs()?['body/start'], 'HH')),6), greater(int(formatDateTime(triggerOutputs()?['body/start'], 'HH')),16))"
    }
  ],
  "splitOn": "@triggerOutputs()?['body/value']"
}
```

When created via the UI ("office365/Outlook > When a new event is created (v3)"), select your calendar from the parameters drop-down, then select the "Settings" tab.
There you need to add two Trigger conditions:

+ `@equals(triggerOutputs()?['body/isReminderOn'], true)` (Event ha a notification to begin with)
+ `@or(less(int(formatDateTime(triggerOutputs()?['body/start'], 'HH')),6), greater(int(formatDateTime(triggerOutputs()?['body/start'], 'HH')),16))` (Event is outside of business hours)

The numbers - "6" and "16" in the second filter are the hour boundaries in UTC.
In Central European Time, this would only apply to meetings that start before 08:00 or after 18:00.
If your working hours end up being across the date boundary, you may need to go a bit more complex on the time boundaries.

## Action 1: Update event

So, let's remove those notifications:

```text
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "table": "AQMkAGQ2NgA4ZGY1ZC04OTc4LTQzZTUtODE2YS00NWE4M2FkYzJjNTIARgAAA2GRZj69MutEmUAsK9ZYZUcHAA_xxxxxxxxxxxxxx_L1z7UAAAIBBgAAAA_4ckB2ou1CoXm5U_L1z7UAAAIVTgAAAA==",
      "id": "@triggerOutputs()?['body/id']",
      "item/subject": "@triggerOutputs()?['body/subject']",
      "item/start": "@triggerOutputs()?['body/start']",
      "item/end": "@triggerOutputs()?['body/end']",
      "item/timeZone": "@{triggerOutputs()?['body/timeZone']}",
      "item/reminderMinutesBeforeStart": 0,
      "item/isReminderOn": false
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connection": "shared_office365",
      "operationId": "V4CalendarPatchItem"
    }
  },
  "runAfter": {}
}
```

Really not much to do here - when creating the Update event action (Again under "office365/Outlook"), just select the same calendar again.
It requires you to specify a few properties, but most of them you can just select data from the trigger.
The only exception is the Time Zone field, which wants you to do a drop-down menu, which we do _not_ want.
If it does not offer the property-selector as in the other fields, you can still chose to provide a custom value and insert this entry: `@{triggerOutputs()?['body/timeZone']}`.

Under advanced parameters, you can add in Reminder and "Is Reminder On", setting it to `0` and `No` respectively.

## Done

That's it, nothing else to say here.
The flow is simple enough and will destress much of the calendar stress in an organization spanning timezones.
