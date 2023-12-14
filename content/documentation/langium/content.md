+++
fragment = "content"
title = "Langium Integration"
weight = 160
[sidebar]
  sticky = true
+++

To extend the Model Hub with your specific language, you need to provide a `ModelServiceContribution`.
A model service contribution can provide a dedicated Model Service API and extend the persistence and validation capabilities of the Model Hub.

One of the core principles in our Model Hub architecture is re-use and we therefore aim to re-use as much of the language infrastructure and support that Langium generates for us.
This is reflected in several design decisions:
- Since the language server that contains the modules with all the languages already starts in a dedicated process, we will start our own Model Hub server in the same process to ease access.
- We are re-using the dependency injection framework from Langium to bind our own Model Hub-specific services, such as the core model hub implementation, the model manager that holds the model storage, the overall command stack and the model subscriptions or the validation service that ensures that all custom validations are run on each model.

The bridge that connects the Model Hub world with the Langium world is our generic _EMF.cloud Model Hub Langium integration library_.
That library has two main components:
1. The Abstract Syntax Tree (AST) server that serves as a facade to access and update semantic models from the Langium language server as a non-LSP client. It provides a simple open-request-update-save/close lifecycle for documents and their semantic model.
2. A converter between the Langium-based AST model and the client language model. The biggest difference between those two models is that the client language model needs to be serializable as we intend to send it to the model hub client which might run in a differenct process. This core work of this transformation is the resolution of cycles and the proper representation of cross references so that when we get a language model back from the client we can restore a full Langium-based AST model again, i.e., to have a full bi-directional transformation when it comes to the semantic model.

Using the generic AST server and the AST-language model converter we can easily implement a language-specific, typed `ModelServiceContribution` that the Model Hub can pick up and use as all we need to do is to connect our Langium services with the respective Model Hub services.
Any additional functionality that we want to expose for our language can be exported as Model Service API in the contribution and re-used in the model hub or even a dedicated server.