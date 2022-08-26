# Workflow entities

The naming and concept of workflow entities is inspired by the [Workflow Patterns Initiative](http://www.workflowpatterns.com/).

**Workflow** defines the structure of a business process. It is built up of tasks which are connected by forward-referencing flows.

**Task** is a single action that occurs in a business process.

**Flow** defines the ordering and dependencies between tasks.

**Case** is a specific instance of a workflow. It has an internal state which represents it's progress.

**WorkItem** is a single unit of work that needs to be completed in a specific stage of a case.

##
