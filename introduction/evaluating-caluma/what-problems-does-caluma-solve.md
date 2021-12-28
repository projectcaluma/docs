# What problems does Caluma solve?

The functionality of Caluma can be divided into two parts: **Forms** and **Workflows**.

#### Forms

While it is trivial to implement simple, one-page forms in most general-purpose frameworks, things get more complicated when you have several of the following requirements:

* Multi-page forms with the possibility to freely navigate between the pages
* Dynamic visibility (show/hide) or requiredness of questions depending on the answers to other questions
* Changes of the form should be possible through a dedicated user interface without requiring technical skills ("form builder")
* Possibility to add multiple answers (or "rows") to a single question (called "table questions" in Caluma)
* Dynamic validation of inputs, completeness indication of subpages
* File uploads
* Multilingual support

Caluma forms offer support for all of the above features, making them a great choice if you need to implement complex forms with great user experience.

#### Workflows

Usually business processes are defined by a list of tasks which are supposed to be executed by different actors in a specific order or with specific dependencies on each other. They can be represented visually, e.g. using BPMN:

![](https://codimd.adfinis.com/uploads/upload\_3e05b9e68107a2b587f08290de889781.png)

Caluma includes a simple workflow engine, which

* supports many common workflow patterns,
* is well integrated with the forms module,
* can be flexibly extended using various extension points,

The workflows are currently defined using the GraphQL API, but a "workflow builder" user interface is something we'd like to add to Caluma in the future.

Next to forms and workflows, most apps need other standard components to be complete. While Caluma's scope is limited to those two core features, there are several other standard components that built to work well with Caluma and can be used in a plug-and play fashion. Details about this follow in the next chapter.
