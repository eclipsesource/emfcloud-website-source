+++
fragment = "content"
title = "Model Hub"
weight = 140
[sidebar]
  sticky = true
+++

The EMF.cloud Model Hub is a central model management component that coordinates multiple clients, such as different editors, in their interaction and manipulation with models.
The Model Hub not only provides a generic API to access models, but is extensible with respect to different modeling languages.
Below we cover not only how clients can access models but also how new modeling languages can be registered.

### Interacting with models

An application may contain several model hubs, each associated to its own "context". The context is a unique string identifier. If an application requires a single model hub context, it may use a constant string; but it is also possible to use more dynamic values that take the current editor into account (e.g. using a folder path, or an application ID, or any value relevant to the application being developped).

Each instance of model hub comes with its own set of contributions, services, models and states.

To access the model hub for a given context, we use the ModelHubProvider, which is registered as a Theia Extension:

```ts
import { ModelHubProvider } from '@eclipse-emfcloud/model-service-theia/lib/node/model-hub-provider';
import { ModelHub } from '@eclipse-emfcloud/model-service';

@injectable()
class ModelHubExample {
  @inject(ModelHubProvider)
  modelHubProvider: ModelHubProvider

  modelHub: ModelHub;

  async initializeModelHub() {
    this.modelHub = await modelHubProvider('my-application-context');
  }
}
```

#### Loading and saving models

Loading and saving models can be achieved by calling the corresponding methods on your model hub instance, assuming contributions have been registered that can handled the requested model IDs. Model IDs are string identifiers that represent a model. They are typically URIs, but can be any arbitrary strings, as long as a Persistence Contribution is able to handle them (See Persistence section below).

```ts
const modelId = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const model: object = await modelHub.getModel(modelId);
```

If you're certain about the type of your model, you can also directly cast it to the necessary type:

```ts
const modelId = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const model: CoffeeModelRoot = await modelHub.getModel<CoffeeModelRoot>(modelId);
```

**Note:**: If the model is already loaded, the in-memory instance will be immediately returned. Otherwise, the model hub will look for a Persistence Contribution that can handle the requested `modelId`, and load it before returning it. Since loading may require asynchronous operations, the `getModel()` method is itself asynchronous.

After applying some changes, you can save your model. For editing the model, the ModelHub uses Commands executed on a CommandStack, identified by a CommandStackId. When using a single model, the commandStackId can be the same value as the modelId. However, since Commands may affect multiple models in some cases, you may want to use a different CommandStackId. When saving this CommandStack, all models that have been modified by a Command executed on this CommandStack will be saved.

```ts
// In this example, we use a single model, so we can use the modelId as the commandStackId.
const modelId = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const commandStackId = modelId;
modelHub.save(commandStackId);
```

Alternatively, you can save all modified models on all available Command Stacks:

```ts
modelHub.save();
```

#### Resolving references

TODO: References are not supported by the ModelHub out of the box. See Langium integration.

#### Changing models

For the sake of isolation, the ModelHub doesn't expose methods to directly edit the models. Instead, Model Contributions are expected to register a Model Service, that will be responsible for handling all edition operations on the models it handles. Any application interesting in editing these models can request the corresponding Model Service, then call any of the exposed API methods to perform edit operations.

Edit Operations take the form of Commands, that are executed on a CommandStack. A Command may change one or several Models, and supports Undo/Redo operations.

Commands are usually handled directly by the Model Service, so they will not be visible to the client of the Model Service.

```ts
export interface CoffeeModelService {
  getCoffeeModel(modelUri: string): Promise<CoffeeModelRoot | undefined>;

  unload(modelUri: string): Promise<void>;

  edit(modelUri: string, patch: Operation[]): Promise<PatchResult>;

  createNode(modelUri: string, parent: string | Workflow, args: CreateNodeArgs): Promise<PatchResult>;
}

/**
 * This constant is used to register the CoffeeModelService and to retrieve
 * it from the ModelHub.
 */ 
export const COFFEE_SERVICE_KEY = 'coffeeModelService';

// Access and use the model service
const modelService: CoffeeModelService = modelHub.getModelService<CoffeeModelService>(COFFEE_SERVICE_KEY);
await modelService.createNode(modelId, '/workflows/0', { type: 'AutomaticTask' });
```

#### Validating models

Model Contributions may register Validators. These Validators will be invoked whenever the ModelHub is validated:

```ts
const modelId = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const diagnostic = await modelHub.validateModels(modelId);
```

Several models can be validated at the same time:

```ts
const modelId1 = 'file:///coffee-editor/examples/workspace/superbrewer2000.coffee';
const modelId2 = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const diagnostic = await modelHub.validateModels(modelId1, modelId2);
```

Or you can validate all models currently loaded, by omitting the `modelIds` argument:

```ts
const diagnostic = await modelHub.validateModels();
```

**Note:** In the latter case, only models currently *loaded* will be validated. Since the model hub relies on lazy-loading to identify existing models, it may ignore some models present in your workspace, if they have never been explicitly loaded beforehand.

The `validateModels` method will validate all requested models, then return the validation results, in the form of a Diagnostic.

If you're only interested in the latest known validation results, but don't want to wait for a full validation cycle, you can use `getValidationState` instead. This method doesn't trigger any validation, but returns the result from the latest validation:

```ts
const modelId = 'file:///coffee-editor/examples/workspace/superbrewer3000.coffee';
const currentDiagnostic = modelHub.getValidationState(modelId);
```

### Contributing modeling languages

The Model Hub can handle several aspects for each Modeling Language:

- Persistence (Save/Load)
- Edition (Via Model Services and Commands)
- Validation
- Triggers

All of these aspects can be registered using a `ModelServiceContribution`. The only mandatory aspect is Edition, via a Model Service.

```ts
/**
 * Our Model Service identifier. Used by clients to retrieve our Model Service.
 */
export const COFFEE_SERVICE_KEY = 'coffeeModelService';

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
    if (! this.modelService){
      this.modelService = new CoffeeModelServiceImpl();
    }
    return this.modelService as unknown as S;
  }
}
```

This minimal example lacks critical capabilities, that are required by most applications: persistence, and access to the Model Manager, in order to execute Commands. Here's a more complete and realistic example:

```ts
/**
 * Our Model Service identifier. Used by clients to retrieve our Model Service.
 */
export const COFFEE_SERVICE_KEY = 'coffeeModelService';

@injectable()
export class CoffeeModelServiceContribution extends AbstractModelServiceContribution {

  private modelService: CoffeeLanguageModelService;

  constructor(@inject(CoffeeLanguageModelService) private languageService: CoffeeLanguageModelService){
    // Empty constructor
  }

  @postConstruct()
  protected init(): void {
    this.initialize({
      id: COFFEE_SERVICE_KEY,
      persistenceContribution: new CoffeePersistenceContribution(this.languageService)
    });
  }

  getModelService<S>(): S {
    return this.modelService as unknown as S;
  }

  setModelManager(modelManager: ModelManager): void {
    super.setModelManager(modelManager);
    // Forward the model manager to our model service, so it can actually
    // execute some commands.
    this.modelService = new CoffeeModelServiceImpl(modelManager, this.languageService);
  }
}

class CoffeePersistenceContribution implements ModelPersistenceContribution {
  
  constructor(private languageService: CoffeeLanguageModelService) {
    // Empty
  }

  canHandle(modelId: string): Promise<boolean> {
    // This example handles file URIs with the '.coffee' extension
    return Promise.resolve(modelId.startsWith('file:/') 
      && modelId.endsWith('.coffee'));
  }

  async loadModel(modelId: string): Promise<object> {
    // Load our model from file...
  }

  async saveModel(modelId: string, model: object): Promise<boolean> {
    // Save the new model to file...
  }
}
```

#### Persistence

Persistence is handled by specifying a `ModelPersistenceContribution` in your `ModelServiceContribution`.

```ts
  @postConstruct()
  protected init(): void {
    this.initialize({
      id: COFFEE_SERVICE_KEY,
      persistenceContribution: new CoffeePersistenceContribution(this.languageService)
    });
  }
```

The Persistence contribution needs to implement three methods: `canHandle(modelId)` to indicate which models it supports, `load(modelId)` and `save(modelId, model)` for the actual persistence.

```ts
class CoffeePersistenceContribution implements ModelPersistenceContribution {

  constructor(private languageService: CoffeeLanguageModelService) {
    // Empty
  }
  
  canHandle(modelId: string): Promise<boolean> {
    // This example handles file URIs with the '.coffee' extension
    return Promise.resolve(modelId.startsWith('file:/') 
      && modelId.endsWith('.coffee'));
  }

  async loadModel(modelId: string): Promise<object> {
    // Load our model from file...
  }

  async saveModel(modelId: string, model: object): Promise<boolean> {
    // Save the new model to file...
  }
}
```

#### Cross References

TODO: References are not supported by the ModelHub out of the box. See Langium integration.

#### Editing Domain

TODO

#### Validators

A ModelServiceContribution can register a ValidationContribution, which will return a list of Validators. Validators will be invoked for all models (including the ones not actually handled by the Model Service Contribution), so they need to implement some kind of Type Guards to decide if they should actually try to validate a model or ignore it.

```ts
  @postConstruct()
  protected init(): void {
    this.initialize({
      id: COFFEE_SERVICE_KEY,
      validationContribution: new CoffeeValidationContribution(this.languageService)
    });
  }
```

A ValidationContribution simply returns a list of Validators:

```ts
class CoffeeValidationContribution implements ModelValidationContribution {

  constructor(private languageService: CoffeeLanguageModelService){
    // Empty
  }

  getValidators(): Validator[] {
    return [
      new WorkflowValidator(),
      new TaskValidator()
    ];
  }
}

class WorkflowValidator implements Validator<string> {
  async validate(modelId: string, model: object): Promise<Diagnostic> {
    // Start with a model typeguard, as all validators will be invoked
    // for all models.
    if (isWorkflow(modelId, model)){
      // Check that model is a well-formed Workflow...
    } else {
      return ok();
    }
  }
}

class TaskValidator implements Validator<string> {
  async validate(modelId: string, model: object): Promise<Diagnostic> {
    if (isWorkflow(modelId, model)){
      const tasks = model.nodes.filter(
        node => node.type === 'AutomaticTask' 
        || node.type === 'ManualTask');
      for (const task of tasks){
        // Check that each Task is well-formed...
      }
    } else {
      return ok();
    }
  }
}
```

#### Custom APIs

Model Service Contributions may (and typically should) expose a public Model Service API, that can be used to interact with the models it provides. A Model Service is identified by a Key, and defined by an Interface. The implementation is then provided by the Model Service Contribution.

Model Service definition, exposed to all clients that may require it:

```ts
/**
 *  Our custom language-specific Model Service API
 */
export interface CoffeeModelService {
  getCoffeeModel(modelUri: string): Promise<CoffeeModelRoot | undefined>;

  unload(modelUri: string): Promise<void>;

  edit(modelUri: string, patch: Operation[]): Promise<PatchResult>;

  createNode(modelUri: string, parent: string | Workflow, args: CreateNodeArgs): Promise<PatchResult>;
}

/**
 * This constant is used to register the CoffeeModelService and to retrieve
 * it from the ModelHub.
 */ 
export const COFFEE_SERVICE_KEY = 'coffeeModelService';
```

Model Service contribution, used to register our language (minimal example):

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

The API can then be retrieved and use by any model hub client:

```ts
const coffeeModelService = modelHub.getModelService<CoffeeModelService>(COFFEE_SERVICE_KEY);
const coffeeModel = await coffeeModelService.getModel('file:///coffee-editor/examples/workspace/superbrewer3000.coffee');
```