+++
fragment = "content"
title = "Model Hub"
weight = 140
[sidebar]
  sticky = true
+++

The EMF.cloud Model Hub is a central model management component that coordinates multiple clients, such as different editors, in their interaction and manipulation with models.
The Model Hub not only provides a generic API to access models, but is extensible with respect to different modeling languages.
Below we cover not only how clients can access models but also how new modeling languages can be registered

### Interacting with models

TODO more detailed look at the API available to clients and how to connect to the model hub

#### Loading and saving models
#### Resolving references
#### Changing models
#### Validating models

### Contributing modeling languages

TODO more detailed look at the API for registering a language contribution for the capabilities listed below

TODO mention that these capabilities don't have to be implemented one by one from scratch, if you use the langium integration as done in the Coffee Editor NG

#### Persistence
#### Cross References
#### Editing Domain
#### Validators
#### Custom APIs
