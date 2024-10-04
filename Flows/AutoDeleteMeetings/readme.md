# Auto-Delete Meetings

## Description

Welcome, dear reader.
Given that you opened this file, I am probably correct in the assumption, that you receive way too many meeting invites and want to clear them off your calendar.
Whether it's just that persistent sender who keeps inviting the entire company, a distribution list you cannot really leave, but gets invited all the time ... or that coworker sending team meetings at 22:00.

The reason probably doesn't really matter, but those meetings must be gone!
Mailbox rules can handle the mail body that brought the meeting, now lets free up our calendar...

This is a fundamentally simple flow:

+ Trigger: Outlook - When a new event is created
+ Action 1: Outlook - Delete event

Using TriggerConditions we make sure, only calendar entries from the specific organizer are affected.

## Trigger

This is the raw view of the trigger:

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
    "interval": 1
  },
  "conditions": [
    {
      "expression": "@equals(triggerOutputs()['body/organizer'], 'foo@bar.com')"
    }
  ],
  "splitOn": "@triggerOutputs()?['body/value']"
}
```

When created via the UI ("office365/Outlook > When a new event is created (v3)"), select your calendar from the parameters drop-down, then select the "Settings" tab.
Then go to the "Settings" tab and add a Trigger Condition, then insert `@equals(triggerOutputs()['body/organizer'], 'foo@bar.com')` (replace the address as suitable).

## Action 1: Delete Event

Kill it with fire:

```text
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "calendar": "AQMkAGQ2NgA4ZGY1ZC04OTc4LTQzZTUtODE2YS00NWE4M2FkYzJjNTIARgAAA2GRZj69MutEmUAsK9ZYZUcHAA_xxxxxxxxxxxxxx_L1z7UAAAIBBgAAAA_4ckB2ou1CoXm5U_L1z7UAAAIVTgAAAA==",
      "event": "@triggerOutputs()?['body/id']"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connection": "shared_office365",
      "operationId": "CalendarDeleteItem_V2"
    }
  },
  "runAfter": {}
}
```

Really not much to do here - when creating the Delete event action (Again under "office365/Outlook"), just select the same calendar again.
Then for the ID field, use the Data Picker tool (click into the text field, it should show up as a lightning bolt on the right) to select the ID property from the list of properties offered.

That's it - save and done.
