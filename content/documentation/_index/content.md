+++
fragment = "content"
title = "Documentation"
weight = 100
[sidebar]
  sticky = true
+++

EMF.cloud is a set of open-source technologies for building next-generation modeling tools based on a modern web technology stack.
In combination with other components and platforms of the open-source ecosystem, EMF.cloud simplifies developing all kinds of modeling tools, whether they are custom modeling suites for domain-specific languages with Eclipse Theia, adaptable general purpose UML-like modeling extensions for VS Code, or tailored configurators for custom data formats in plain web applications.
You can also use EMF.cloud technologies to enrich your custom IDE with certain modeling tool features, such as integrated form-based editors, graphical diagram editors, and textual DSLs.

<div style="text-align:center; margin-bottom:20px">
  <img src="../images/overview.svg" alt="Overview of EMF.cloud features" width="70%" />
</div>

Most of the EMF.cloud components are frameworks written in Typescript either running in the browser or on a node-based backend.
They range from model management components to provide consistent access to underlying models from mulitple types of editors within a tool to integration code with third-party technologies and platforms, such as [Eclipse Theia](https://theia-ide.org), VS Code, [Langium](https://langium.org), [Eclipse GLSP](https://eclipse.dev/glsp/), or [JSON Forms](https://jsonforms.io).
If you need to reuse EMF-based tools and thus require Java on the backend, EMF.cloud also provides dedicated components to allow accessing and manipulating EMF models a modern web-based tool environment.

➡️ To get an overview of all EMF.cloud components, please head over to the [Overview]({{< relref  "overview" >}}) section.

We are continuously working on improving this documentation. If you feel something is missing, please file an issue and open a PR at the [documentation repository](https://github.com/eclipse-emfcloud/emfcloud-website-source).
If you need more help, please have a look at our [available support options]({{< relref  "/support" >}}).
