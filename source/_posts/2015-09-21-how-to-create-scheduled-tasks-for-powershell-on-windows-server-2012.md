title: "How to create Scheduled Tasks for Powershell on Windows Server 2012"
date: 2015-09-21 18:45:43
categories:
  - Microsoft
tags:
- microsoft
- server-2012
---
Instructions on how to set up properly configured Scheduled Tasks for Powershell
scripts on Windows Server 2012.

## Creating the service account

Assuming the account only requires local administrator permissions:

- open the Local Users and Groups console (``lusrmgr.msc``)
- create a new local user (service) account used to execute the scheduled task
- add the service account to the local Administrators group
- open the Local Security console (``gepdit.msc``) and expand:
  - Local Computer Policy
  - Windows Settings
  - Security Settings
  - Local Policies
  - User Rights Assignment
- open the ``Log on as a service`` policy and make sure to add the service account

## Creating the Scheduled Task

Open the Task Scheduler console (``taskschd.msc``) and create a new task.

### General tab

- use a meaningful name
- specify your service account using the ``Change User or Group`` button
- select ``Run whether user user is logged in or not``
- select the ``hidden`` option
- set the ``Configure for`` option to Windows Server 2012

### Triggers tab

Create two triggers:

- one using begin ``At startup`` so the scheduled task will survive reboots
- one using begin ``At task creation/modification`` so it will run from the
moment the task is created

Make sure both triggers share the same configuration:

- select ``Repeat task at every`` and set to e.g. 30 minutes
- change the duration to ``Indefinitely``
- be proactive; stop the task if it runs longer than ``1 hour``
- select the ``Enabled`` checkbox

### Actions tab

Create a new action:

- select ``Start a program``
- set the ``Program/script`` field to (just) ``Powershell``
- set the ``Arguments`` field to ``-noprofile -executionpolicy bypass -file "c:\your-script.ps1"``

> **Note:** do NOT use the ``-noexit`` argument as this will keep your task running
> indefinitely, preventing it to reach ``Ready`` state after completion.

### Conditions tab

Nothing to do here

### Settings tab

Make sure to set the bottom most dropdown to ``Stop the existing instance``.

## Round up

On saving the task it will:

- start immediately (status ``Running``)
- change to status ``Queued`` after it has completed
- will be executed executed again every other 30 minutes
