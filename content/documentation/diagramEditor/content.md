+++
fragment = "content"
title = "Diagram Editors"
weight = 190
[sidebar]
  sticky = true
+++

<div style="text-align:center; margin-top:50px; margin-bottom:50px">
  <img src="../../images/diagramanimated.gif" alt="Overview of the Model Hub for the Coffee Editor NG" width="80%" />
</div>

## Introduction

The foundation of implementing diagram editors is the [GLSP framework](https://eclipse.dev/glsp/).
This guide will give an introduction to connect GLSP diagram editors with the [ModelHub](https://github.com/eclipsesource/coffee-editor/tree/main/emfcloud-modelmanager/model-service-theia).
Specifically, the GLSP diagram editor makes use of the frontend `ModelHub` and its provided interfaces like the `CoffeeModelServer` or the `CoffeeModelService` for seamless integration and outsourced data management.
Furthermore, specific examples of the [Coffee Editor NG implementation](#coffee-editor-ng-example) will be given as a guideline.

## ModelHub Integration

</br>

### SourceModelStorage

A source model storage handles the persistence of source models, i.e. loading and saving it from/into the model state.
A `source model` is an arbitrary model from which the graph model of the diagram is to be created.
A source model loader obtains the information on which source model shall loaded from a
RequestModelAction; typically its client options. Once the source model is loaded, a model loader is expected
to put the loaded source model into the model state for further processing, such as transforming the loaded model
into a graph model (see [GModelFactory](#gmodelfactory) below).
On `saveSourceModel(SaveModelAction)`, the source model storage persists the source model from the model state.

In our case all of these responsibilites are outsourced to the `ModelHub`, namely the model loading, subscribing to model changes, dirty state information and triggering the save of the model.

### GModelFactory

A graph model factory produces a graph model from the source model contained in the model state.

In this case, we also use the source model from the model state to translate the source model into `GModelRoot`. For more complex translations, e.g. for creating edges, the `CoffeeModelServer` is used to resolve references.

### Model Operations

To outsource the responsibility of executing operations directly on the model, operations should be forwarded to the [CoffeeModelService]({{< relref  "/editingDomain#model-service-implementation" >}}).
The `CoffeeModelService` offers tailored functions to manipulate the model, e.g. dedicated create or delete functions that take as arguments the specific diagram data like the position on the canvas.

## Coffee Editor NG Example

</br>

### WorkflowModelStorage

The `WorkflowModelStorage` is responsible for loading and saving source models via the `ModelHub`.

#### Load the source model

To load the source model for further usage and transformation into the graphical GModel, we fetch the `CoffeeModelRoot` from the `modelHub` as follows:

```typescript
this.modelHub.getModel<CoffeeModelRoot>(modelUri);
```

<details><summary>Detailed implementation</summary>

```typescript
  async loadSourceModel(action: RequestModelAction): Promise<void> {
    const modelUri = this.getUri(action);
    const coffeeModel = await this.modelHub.getModel<CoffeeModelRoot>(modelUri);
    if (!coffeeModel || !isMachine(coffeeModel) || (isMachine(coffeeModel) && coffeeModel.workflows.length < 1)) {
      throw new GLSPServerError('Expected Coffee Model with at least one workflow');
    }
    this.modelState.setSemanticRoot(modelUri, coffeeModel);
    this.subscribeToChanges(modelUri);
  }
```
</details>

</br>

#### Subscribe to model changes

To subscribe to model changes, `modelHub` offers a `subscribe` function:

```typescript
this.subscription = this.modelHub.subscribe(modelUri);
this.subscription.onModelChanged = async (modelId: string, newModel: object) => {
  // handle model update
}
```

<details><summary>Detailed implementation</summary>

```typescript
private subscribeToChanges(modelUri: string): void {
    this.subscription = this.modelHub.subscribe(modelUri);
    this.subscription.onModelChanged = async (modelId: string, newModel: object) => {
      if (!newModel || !isMachine(newModel) || newModel.workflows.length < 1) {
        throw new GLSPServerError('Expected Coffee Model with at least one workflow');
      }
      this.modelState.setSemanticRoot(modelId, newModel);
      const actions = await this.submissionHandler.submitModel();
      const dirtyStateAction = actions.find(action => action.kind === SetDirtyStateAction.KIND);
      if (dirtyStateAction && SetDirtyStateAction.is(dirtyStateAction)) {
        dirtyStateAction.isDirty = this.modelHub.isDirty(this.modelState.semanticUri);
      }
      this.actionDispatcher.dispatchAll(actions);
    };
  }
```
</details>

</br>

#### Save model

To trigger model saving, `modelHub` offers a `save` function:

```typescript
this.modelHub.save(modelUri)
```

<details><summary>Detailed implementation</summary

```typescript
async saveSourceModel(action: SaveModelAction): Promise<void> {
    const modelUri = action.fileUri ?? this.modelState.semanticUri;
    if (modelUri) {
      await this.modelHub.save(modelUri);
    }
  }
```
</details>

</br>

### CoffeeGModelFactory

To resolve references, such as edge sources or targets, we use the `CoffeeModelServer` to get the resolved data needed to properly translate the source model into a `GModelRoot`.


This shows one flow from the example source model `superbrewer3000.coffee`:

```json
  ...
  {
    "id": "checkWaterToDecision",
    "type": "Flow",
    "source": "SuperBrewer3000.BrewingFlow.Check Water",
    "target": "SuperBrewer3000.BrewingFlow.MyDecision"
  },
  ...
```

The following snippet shows how to create a GEdge from such a flow object:

```typescript
@inject(CoffeeModelServer)
  protected modelServer: CoffeeModelServer;

  ...

  protected async createEdge(edge: Flow): Promise<GEdge> {
    const [sourceNode, targetNode] = await Promise.all([
      this.modelServer.resolve<Node>(edge.source as Required<NodeReferenceInfo>),
      this.modelServer.resolve<Node>(edge.target as Required<NodeReferenceInfo>)
    ]);
    return GEdge.builder()
      .id(edge.id)
      .sourceId(sourceNode.id)
      .targetId(targetNode.id)
      .build();
  }

```

### Model Operations via the CoffeeModelService

To showcase how to forward model operations to the [CoffeeModelService]({{< relref  "/editingDomain#model-service-implementation" >}}),
let's take a look at the `CreateWorkflowNodeOperationHandler`.
This handler is an abstract node creation operation handler that is used to create all possible types of nodes supported by the Coffee diagram editor, e.g. manual tasks or fork nodes.

We wrap the model operation in a `GModelRecordingCommand` - in our case a customized `CoffeeModelCommand` that aligns the customized types of the `GModelState` for example.
The model operation itself is basically the operation data collection we get from the diagram editor, which we hand over to the `CoffeeModelService`.
The modelService provides a dedicated function to create a new node, expecting the diagram editor's specific data like position and type of node.

```typescript
  override createCommand(operation: CreateNodeOperation): MaybePromise<Command | undefined> {
    return new CoffeeModelCommand(
      this.modelState,
      this.serializer,
      () => this.createWorkflowNode(operation)
    );
  }

  async createWorkflowNode(operation: CreateNodeOperation): Promise<void> {
    const container = this.modelState.semanticRoot;
    const modelService = this.modelHub.getModelService<CoffeeModelService>(COFFEE_SERVICE_KEY);
    const modelId = this.modelState.semanticUri;
    await modelService?.createNode(modelId, container.workflows[0], {
      posX: this.getLocation(operation)?.x ?? Point.ORIGIN.x,
      posY: this.getLocation(operation)?.y ?? Point.ORIGIN.y,
      type: this.getNodeType(operation)
    });
  }
```

Similarly, this is the way to implement model operations, also for other diagram editor use cases like deleting model elements, resizing or repostioning model elements or edit model element labels.


