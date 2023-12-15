+++
fragment = "content"
title = "Editing Domain"
weight = 170
[sidebar]
  sticky = true
+++

#### General principles

Edition of models is handled by the ModelManager, by executing Commands on one or several CommandStacks. The ModelManager is not directly exposed by the ModelHub API, so it is not (easily) possible to execute arbitrary commands on arbitrary models. Instead, the ModelManager is passed to ModelServiceContributions, which can then use it in their custom ModelService implementation to execute commands. This way, clients do not have to deal with Commands directly, but simply interact with the custom API.

#### Model Service implementation

As seen in the [ModelHub section]({{< relref  "modelhub" >}}), the ModelServiceContribution is the entry point for model-specific contributions. Upon initialization of the ModelHub, each contribution will get access to the ModelManager, and should typically forward it to their ModelService implementation.

```ts
@injectable()
export class CoffeeModelServiceContribution extends AbstractModelServiceContribution {
  private modelService: CoffeeModelService;

  @postConstruct()
  protected init(): void {
    this.initialize({
      id: COFFEE_SERVICE_KEY
    })
  }

  getModelService<S>(): S {
    return this.modelService as unknown as S;
  }

  setModelManager(modelManager: ModelManager): void {
    super.setModelManager(modelManager);
    // Forward the model manager to our model service, so it can actually
    // execute some commands.
    this.modelService = new CoffeeModelServiceImpl(modelManager);
  }
}
```

Then, the ModelService implementation can access the model and execute Commands on the ModelManager:

```ts
export class CoffeeModelServiceImpl implements CoffeeModelService {
  constructor(private modelManager: ModelManager<string>) {}

  // clients could directly access the model from the Model Hub, but 
  // custom ModelService APIs can also provide a convenience method:
  async getCoffeeModel(modelUri: string): Promise<CoffeeModelRoot | undefined> {
    const key = getModelKey(modelUri);
    return this.modelManager.getModel<CoffeeModelRoot>(key);
  }

  // [...]

  async createNode(modelUri: string, parent: string | Workflow, args: CreateNodeArgs): Promise<PatchResult> {
    const model = await this.getCoffeeModel(modelUri);
    if (model === undefined) {
      return {
        success: false,
        error: `Failed to edit ${modelUri.toString()}: Model not found`
      };
    }

    // parent can be either the element itself, or its path. Resolve the
    // correct element.
    const parentPath = this.getParentPath(model, parent);
    const parentElement = getValueByPointer(model, parentPath);
    if (!isWorkflow(parentElement)) {
      throw new Error(`Parent element is not a Workflow: ${parentPath}`);
    }

    // create the new Node (regular JSON object)
    const newNode = createNode(args.type, parentElement, args);

    // create a JSON patch to edit the model
    const patch: Operation[] = [
      {
        op: 'add',
        path: `${parentPath}/nodes/-`,
        value: newNode
      }
    ];
    
    // get the command stack. In this case, we don't have multi-model command stacks,
    // so just use the modelUri as the command stack id.
    const stackId = getStackId(modelUri);
    const stack = this.modelManager.getCommandStack(stackId);

    // create the Command from the JSON Patch and execute it
    const command = new PatchCommand('Create Node', modelUri, patch);
    const result = await stack.execute(command);

    // Return a patch result that indicates success or failure, and applied changes
    // in case of success.
    const patchResult = result?.get(command);
    if (patchResult === undefined) {
      return {
        success: false,
        error: `Failed to edit ${modelUri.toString()}: Model edition failed`
      };
    }
    return {
      success: true,
      patch: patchResult
    };
  }
}
```

There are several ways to create patch commands. In the above example, we created the JSON Patch manually, using the JSON pointer path from the parent, and adding a value. However, using a JSON Patch Library (such as `fast-json-patch`, although any similar library can be used), one could generate the patch instead:

```ts
export class CoffeeModelServiceImpl implements CoffeeModelService {

  // [...]

  async createNode(modelUri: string, parentPath: string): Promise<PatchResult> {
      const model = await this.getCoffeeModel(modelUri);

      // [...]

      // We are not allowed to edit the `model` object directly. Make a copy, 
      // and then we'll use fast-json-patch to generate a diff-patch.
      const updatedModel = deepClone(model) as CoffeeModelRoot;
      const updatedWorkflow = getValueByPointer(updatedModel, parentPath);
      if (!isWorkflow(parentElement)) {
        throw new Error(`Parent element is not a Workflow: ${parentPath}`);
      }

      // Directly modify the updatedModel object, by adding the new node to it
      const newNode = createNode(args.type, parentElement, args);
      updatedWorkflow.nodes.push(newNode);

      // Generate a patch using fast-json-patch.compare() and create a command
      const patch: Operation[] = compare(model, updatedModel);
      const command = new PatchCommand('Create Node', modelUri, patch);

      // [...]

      // then get the command stack and execute the command as before
      const result = await stack.execute(command);

      // [...]
  }
}
```

The ModelManager framework also provides a convenience method to create a PatchCommand that will directly edit the JSON Model, using a Model Updater:

```ts
export class CoffeeModelServiceImpl implements CoffeeModelService {

  // [...]

  async createNode(modelUri: string, parentPath: string): Promise<PatchResult> {
      const model = await this.getCoffeeModel(modelUri);

      // create a PatchCommand using a Model Updater. This allows us to directly
      // edit the model, without having to deal with JSON Patches at all.
      const command = new PatchCommand('Create Node', modelUri, model => {
        const workflow = getValueByPointer(model, parentPath);
        if (!isWorkflow(parentElement)) {
          throw new Error(`Parent element is not a Workflow: ${parentPath}`);
        }
        // Directly modify the updatedModel object, by adding the new node to it.
        // Inside of the Model Updater, we are allowed to edit the model object
        // directly - no need to create our own working copy!
        const newNode = createNode(args.type, parentElement, args);
        workflow.nodes.push(newNode);
      });

      // [...]

      // then get the command stack and execute the command as usual
      const result = await stack.execute(command);

      // [...]
  }
}
```

#### Command Stack IDs

A Command can be executed on any CommandStack. The Command Stack ID defines how Undo/Redo will behave, especially when a Command affects multiple models. When undoing (or redoing) changes on a Command Stack, the latest command executed on this Stack will be undone, ignoring commands that were executed in different stacks.

The most typical use case is to have one command stack per editor, so editors are independent from each other: undoing changes in one editor (command stack) does not affect the state of other editors.

However, when models have cross-references, you may be able to open inter-related models in different editors. In this case, it can be necessary to use a shared command stack for all editors, to ensure all editors work on a consistent state of the model. In that case, you may want to use a Command Stack Identifier that represents the entire set of inter-related models, such as the parent folder path, or project name.

#### Model Hub Context

A Context is a string identifier that defines the scope of a Model Hub. One model hub instance exists per context. Each Model Hub has its own set of Model Service Contributions, Model Manager, and set of Command Stacks, which are completely independent from other Model Hub instances. A Command executed in a given context cannot be undone from another context.

It is up to each application to decide how these contexts are defined. For example, you may decide that you need to isolate changes on a per-project basis, in which case it can be useful to use the Project ID as the Model Hub context. Alternatively, if you're defining several modeling languages without any relationship to each other, you may choose to use the language ID as the context.

For simpler applications, using a single, constant context ID is usually recommended.