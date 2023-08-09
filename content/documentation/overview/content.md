+++
fragment = "content"
title = "Overall Architecture and Components"
weight = 110
[sidebar]
  sticky = true
+++

At the center of EMF.cloud, there is a model management component, which provides consistent model access, as well as an interface for manipulating models and listening to model changes across various editors, views, and components interacting with a model.
This model management component is called [*Model Hub* (Typescript)]({{< relref "modelHub" >}}).

<div style="text-align:center; margin-bottom:20px">
  <img src="../../images/modelhub.svg" alt="Overview of the Model Hub" width="70%" />
</div>

The Model Hub is an extensible central model management component, which offers a common API to all clients for accessing and interacting with models.
As such, the model hub provides plenty of generic functionalities, such as dirty state handling of models, undo/redo, notification mechanisms for various events, etc.
It supports clients living in the frontend (browser), as well as clients that are running in a backend (e.g. node.js).

For each modeling language or format, you can register a *modeling language contribution*, which defines how certain model-specific capabilities are implemented, such as persisting models, model-specific operations or APIs that can be invoked for your model, how cross-references are to be resolved within a workspace, validation rules, etc.
This gives you full control and customizability in all relevant aspects of the model management for your modeling language, format, or data source, whether it is a JSON file, custom file format, database, or REST service that you want to make available to your clients by integrating them with the Model Hub.

To make your life easier, you don't have to implement all of those Model Hub capabilities from scratch for every modeling language.
Instead EMF.cloud provides reusable libraries and integration code for third-party components to cover the most common choices and formats, such as JSON files.
Also, EMF.cloud contains libraries that make it easy to connect and interact with the Model Hub.

➡️ Best to [get started]({{< relref  "gettingStarted" >}}) with the Coffee Editor NG!

**Note on EMF:** If you need to support EMF-based models, there is a dedicated Java-based model management component for EMF models. For more information, please head over to the [EMF support documentation]({{< relref  "emf" >}}).
For all other use cases, we recommend to use the *Model Hub* as it provides a more homogeneous developer experience based on Typescript throughout the client and the server.