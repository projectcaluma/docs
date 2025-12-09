# Workflow

This guide explains the most important aspects of the _workflow_ capabilities of Caluma. First, we'll look at the concepts the workflow system is based on, and after at how these concepts can be applied in practice by using Caluma's GraphQL API.

## Concepts

When thinking about workflows, it helps to differentiate between two different steps of a workflow's lifecycle:

1. _Design_: First, the desired workflow is _designed_ by constructing it from its basic building blocks. For workflows those building blocks are called _tasks_ and the connections between them are called _flows_. Only when all the needed tasks and flows are specified, the design of a workflow is complete.
2. _Execution_: After the workflow is designed, it can be executed by creating an instance of it. An Instance of a workflow is called a _case_ and consists of _work items_, which are the instances of _tasks_.

The following picture summarizes the relationship between the various workflow entities and the phases they are concerned with:

![Workflow entities diagram](../.gitbook/assets/workflow-entities.svg)

In order to illustrate how the workflow entities are used, let's imagine a simple workflow that describes an exam in a school:

![Exam workflow with two tasks](../.gitbook/assets/exam-design.svg)

The "exam" workflow states that every exam consists of two steps:

1. The pupil fills out the exam
2. The teacher corrects the exam

The way the tasks are connected defines the order in which they are executed, i.e. that without a filled out exam there can also be no correction; or after the correction has started it can't be filled out any more.

Now that we know about the workflow's design, let's imagine this workflow being executed and visualize the different steps it passes through:

1.  When a new "exam" case is started, Caluma creates a new "fill out exam" work item.

    <img src="../.gitbook/assets/exam-execution-1.svg" alt="Exam case with one work item" data-size="original">
2.  The exam is filled out, and the corresponding work item is completed. Caluma creates a new work item for the "correct exam" task.

    <img src="../.gitbook/assets/exam-execution-2.svg" alt="Exam case with one work item" data-size="original">
3.  The exam is corrected, and just as before the corresponding work item is completed. Since there are no more tasks following in the workflow's design, the whole case is marked as "complete".

    <img src="../.gitbook/assets/exam-execution-3.svg" alt="Exam case with one work item" data-size="original">

The separation of _Design_ and _Execution_ allows having many "exam" cases in different states at the same time:

![Three exam cases in different states](../.gitbook/assets/exam-execution-4.svg)

Most workflows needed in real applications are more complex than the "exam" workflow we looked at in this example. To handle complex workflows, more building blocks are needed. The building blocks for workflows are called _Worfklow Patterns_, and you'll find all the patterns which are supported in Caluma in the docs (currently WIP).

***

> **Workflow Patterns and Standardization**
>
> Workflow Patterns in software applications are their very own field of research: The [Workflow Patterns initiative](http://www.workflowpatterns.com/) categorizes, visualizes and explains known patterns and compares their support in various software applications.
>
> When developing Caluma we try to incorporate as much of this research into our work as possible: The naming of our entities ("Workflow", "Case", "Task", "Work Item") stems directly from the workflow pattern initiative, and the workflow patterns we support are named according to the existing patterns.
>
> If you find that a specific workflow can't be implemented with the patterns that Caluma currently supports, then identifying and mentioning the workflow you need on the Workflow Patterns initiative is a great way to put your use-case in the right context and kick-start a possible implementation in Caluma. We're interested to support as many documented Workflow Patterns as possible, especially if their implementation doesn't add a great deal of complexity to the codebase - if you'd like to contribute to Caluma development by adding support for a new pattern, please reach out and we'll do our best to support you!

***



## Implementation

Now that we know about the concepts behind Caluma's workflow system, it's time to get our hands dirty and implement the "exam" workflow. We'll do so by executing mutations on the Caluma GraphQL API - to get most out of this guide, we recommend to follow along using the interactive GraphiQL console of your local Caluma installation (see [#installation](guide.md#installation "mention") for instructions).

### Step 1: Workflow Design

In this first step, we'll create the "exam" workflow including its tasks and flows. Although it would feel most natural to start by creating the workflow itself, we'll start by creating the "fill out exam" task - the reason for this is that when creating a new workflow, we have to tell caluma which task(s) to start when the workflow is started.

#### 1a) Create "fill out exam" task

The following GraphQL snippet runs the `saveSimpleTask` mutation. We have to specify the tasks name and slug - the slug is the technical identifier of the task, and can only contain lower-case letters and a limited set of special characters.

```
mutation {
  saveSimpleTask(input: {
    slug: "fill-out-exam",
    name: "Fill out form"
  }) {
    task {
      slug
    }
  }
}
```

After executing the mutation, Caluma should confirm the successful creation of the task with a response like this:

```json
{
  "data": {
    "saveSimpleTask": {
      "task": {
        "slug": "fill-out-exam"
      }
    }
  }
}
```

> **Task types**
>
> _Why did we use the `saveSimpleTask` mutation? There are also other mutations that save tasks._
>
> Caluma supports different types of tasks, each of which has its own set of mutations. The following types are supported:
>
> * **Simple Tasks** can be completed without any requirements. _Simple_ doesn't mean that they are _easy_ to complete, but that Caluma doesn't (or rather _can't_) do any validations if it's really possible to complete a work item of this type. An example for this type of task might be an "Approval" task in a workflow that includes requests for vacation in a company: The approval can be given without any further information if a given request should be approved.
> *   **Workflow Form Tasks** and **Task Form Tasks** both deal with forms: Both require a document to be filled out before the task (or, more precisely, it's associated work item) can be completed. Caluma's data model features two places where forms (and therefore documents) can be found:
>
>     TODO diagram
>
>     As you can see, a form can either be attached to a specific task, or to the entire workflow. In the latter case, the form is called the workflow's _main form_.
>
>     _When should I use a Workflow Form instead of a Task Form?_
>
>     Often, workflows have data which is central to their execution: Everyone involved in a specific case needs access to this information in order to work productively with the case. For example, in a workflow involving an application the application form is typically so central that it's used as the workflow's _main form_. On the other hand, if form data is not central to the workflow's "identity", a _task form_ should be used.
>
>     Also note that technically, it would be possible to only use _task forms_ - _workflow forms_ are merely a convenient way to store the most important form data at a central location.
>
> For the first task in our example "exam" workflow, a _workflow form task_ would probably be most appropriate - we decided to use a _simple task_ here anyway, because in this guide we'd like to show workflow concepts without the need to set up a form beforehand.

#### 1b) Create workflow

Now that we have the first task in place, let's create the "exam" workflow:

```
mutation {
  saveWorkflow(input: {
    slug: "exam",
    name: "Exam",
    startTasks: ["fill-out-exam"]
  }) {
    workflow {
      slug
    }
  }
}
```

Conceptually, the current state of the workflow looks like this:

TODO diagram

To complete the "exam" workflow according to our specification, we still need to add the second task as well as a _flow_ connecting the two tasks.

#### 1c) Create "correct exam" task

We've already seen how to do this in step 1a - all we need to change is the tasks' name and slug:

```
mutation {
  saveSimpleTask(input: {
    slug: "correct-exam",
    name: "Correct exam"
  }) {
    task {
      slug
    }
  }
}
```

#### 1d) Create flow

A _flow_ is a connection between two (or more) tasks in a workflow. Creating a flow looks like this:

```
mutation {
  addWorkflowFlow(input: {
    workflow: "exam",
    tasks: ["fill-out-exam"],
    next: "'correct-exam'|task"
  }) {
    workflow {
      flows {
        edges {
          node {
            tasks {
              slug
            }
            next
          }
        }
      }
    }
  }
}
```

```json
{
  "data": {
    "addWorkflowFlow": {
      "workflow": {
        "flows": {
          "edges": [
            {
              "node": {
                "tasks": [
                  {
                    "slug": "fill-out-exam"
                  }
                ],
                "next": "'correct-exam'|task"
              }
            }
          ]
        }
      }
    }
  }
}
```
