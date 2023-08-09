+++
fragment = "content"
title = "Coffee Editor NG"
weight = 130
[sidebar]
  sticky = true
+++

The Coffee Editor NG is a comprehensive example modeling tool based on EMF.cloud technologies and can also act as an architecture blueprint for your custom modeling tool.
It thus brings together a set of best practices in architecture and technology selection from both inside and outside of EMF.cloud.
This full-featured example tool is written entirely in Typescript and includes a central Model Hub serving a sample modeling language to a variety of editors, such as a diagram editor, a form-based editor, as well as a textual DSL editor.

<div style="text-align:center; margin-bottom:20px">
  <img src="../../images/coffeeeditormodelhub.svg" alt="Overview of the Model Hub for the Coffee Editor NG" width="70%" />
</div>

The core of the Coffee Editor NG is a model language contribution to the Model Hub
With this language contribution, we register the modeling language alongside the capabilities for handling this modeling language in the Model Hub.
The contributed Coffee Editor NG Model is defined with Langium, an open-source language toolkit for textual languages.
The Langium-based language is then integrated into the Model Hub API with the generic *EMF.cloud Model Hub Langium* integration library, so that Model Hub clients can interact with the language on model level and benefit from Langium's efficient persistence, cross-reference management and validation mechanism.

In order to also simplify the editing capabilities, including undo and redo, on model level -- rather than on text level -- we use the *EMF.cloud Editing Domain*, which provies state management, as well as a command API and a command stack for arbitrary JSON models.

Finally, the Coffee Editor NG language contribution adds a custom coffee-model-specific API to be used by clients in order to provide reusable model queries and manipulation functions to the clients.

➡️ TODO Link to try now

➡️ TODO Link to source code and a good entry point to look at the language contribution

In the remainder of this documentation, we use the Coffee Editor NG as an example to demonstrate how certain capabilities can be customized, extended or exchanged with other implementations.
Please use the table of contents on the left to navigate to the respective topic of interest.
