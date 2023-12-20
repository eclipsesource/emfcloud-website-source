+++
fragment = "content"

title = "EMF Support"
weight = 998

[sidebar]
  sticky = true
+++

## EMF Model Server

If you are not creating your modeling tool from scratch, but need to migrate an existing EMF-based model to a modern web-based modeling tool, EMF Cloud provides a dedicated model management component, called [*EMF Model Server*](https://github.com/eclipse-emfcloud/emfcloud-modelserver). The EMF Model Server is written in Java and provides access to your EMF models, including manipulation, state management, undo and redo, via a generic REST API, as well as a JSON-RPC channel.
This not only opens up accessing EMF models from web-based frontends and other components that aren't written in Java, but also encapsulates your EMF dependency for future migrations.

Alongside the EMF Model Server, there are also several components that simplify interacting with an EMF Model Server:
  * [Java-based EMF Model Server Client](https://github.com/eclipse-emfcloud/emfcloud-modelserver)
  * [Typescript-based EMF Model Server Client](https://www.npmjs.com/package/@eclipse-emfcloud/modelserver-client)
  * [Eclipse Theia integration of the EMF Model Server](https://github.com/eclipse-emfcloud/emfcloud-modelserver-theia)
  * [Eclipse GLSP Integration](https://github.com/eclipse-emfcloud/modelserver-glsp-integration)

## EMF Coffee Editor

The EMF Coffee Editor provides a comprehensive example modeling tool that combines the EMF Model Server as well as all components mentioned above.
The [sources of the Coffee Editor](https://github.com/eclipsesource/coffee-editor) are available under an open-source license and thus makes a great blueprint and starting point for your modeling tool project.

<div style="text-align:center; margin-bottom:20px">
  <img src="../../images/coffeeeditordemo.gif" alt="Coffee Editor (Java)" width="70%" />
</div>

This example provides several features:
* A custom Theia application frame
* Tree/form-based property editor
* Diagram editor
* Textual DSL
* Model analysis and visualization
* Code generation

Go ahead and [try out the coffee editor online](https://eclipsesource.com/coffee-editor)!

## Getting Started

To get you started quickly, we also provide project templates for the most popular choices including EMF Cloud and [GLSP](https://www.eclipse.dev/glsp/documentation/gettingstarted/) components.

Please see the following project-template and follow its README file.

[💾 Model Server ● 🖥️ Java ● 🗂️ EMF ● 🖼️ Theia -- `modelserver-glspjava-emf-theia`](https://github.com/eclipse-emfcloud/modelserver-glsp-integration/tree/main/project-templates/modelserver-glspjava-emf-theia)

If you need help, please raise a question in the [Github discussions](https://github.com/eclipse-emfcloud/emfcloud/discussions) or look at our [support options]({{< relref  "/support" >}}).
