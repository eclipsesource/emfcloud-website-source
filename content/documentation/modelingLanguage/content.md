+++
fragment = "content"
title = "Defining a Modeling Language"
weight = 160
[sidebar]
  sticky = true
+++

In Model Hub, languages can be easily defined using Eclipse Langium. 
[Langium](https://langium.org/) is an open source language engineering tool that allows you to declare the syntax of your modeling language in form of an EBNF-like grammar.
From that grammar, the Langium CLI can generate a complete TypeScript-based language server, including syntax highlighting, auto-completion, cross references, validation and many other features.
Internally, Langium uses a [Chevrotain](https://chevrotain.io/docs/) parser extended with an [ALL(*) algorithm](https://www.typefox.io/blog/allstar-lookahead) for unbounded lookahead and is re-using language server infrastructure classes from VS Code.
While Langium has more capabilities such as command line interface generation or visualization, we will focus on the aspects that relate to the model hub.

In Langium, a grammar is defined in a dedicated `.langium` file for which a VS Code Extension also provides tooling support.
Let's assume we want to have a simple grammar that follows a JSON syntax:

```ts
grammar CoffeeLanguage                    // grammar name

entry CoffeeModelRoot:                    // entry rule for the parser, i.e., document root
    Machine | WorkflowConfig;             // sequence of valid tokens → abstract syntax

fragment IdentifiableFragment:            // re-usable fragment
    '"id"' ':' id=STRING;

TYPE_MACHINE returns string: '"Machine"'; // we use a type to ease distinction for parsing
Machine:
    '{'
        IdentifiableFragment
        ',' '"name"' ':' name=STRING      // keywords as inline terminals → conrecte syntax
        ',' '"type"' ':' type=TYPE_MACHINE
        (',' '"workflows"' ':' '['
            ((workflows+=Workflow) (',' workflows+=Workflow)*)?
        ']')?                             // optional workflow children
    '}';

TYPE_WORKFLOW returns string: '"Workflow"';
Workflow:
    '{'
        IdentifiableFragment
        ',' '"name"' ':' name=STRING
        ',' '"type"' ':' type=TYPE_WORKFLOW
    '}';

TYPE_WORKFLOW_CONFIG returns string: '"WorkflowConfig"';
WorkflowConfig:
    '{'
        '"machine"' ':' machine=[Machine:STRING] // reference a machine defined somewhere else
        ',' '"workflow"' ':' workflow=[Workflow:STRING];
        ',' '"type"' ':' type=TYPE_WORKFLOW_CONFIG
    '}';

hidden terminal WS: /\s+/;                // Ignore whitespaces during parsing
terminal STRING: /"[^"]*"/;               // JSON only supports double quoted strings
```

If you try out this grammar on the [Langium Playground](https://langium.org/playground/?grammar=OYJwhgthYgBAwgewGbIKZoDJgHbAK5jBqylnkUUD0Vsok0cOkaAUK2jgC4gCeCKdGgCyiACZoANgCVEiLgC5Ky5TVice-EPkklkiOFwAWJAA4wAzmhAAaWAEsAdGkd2xiAMb4IG2CDlcrOSy8gDcKmRqVgCO%2BJweJCiwAG5gkvZisFyIANacFrCASYSwYABGFjxgHlywFrzcYAAe7CGKQWRSaD7cFgDUALwAFMJVRvY4JAA%2BsADqBjnIkogA7kg4yPbAAJQAVOHsyODA3TUAkhLc9htlugBiRydKEaRqIGgAtPgWN3oPGu2kADkACIMsDAbBAQoIRl%2BgBlAAq0lOADkAOKhdgIgCaAAUAKIAfWEAEF4AAJVH4vxoLj4EA4AoVEDjYBKEEjDxjCbg8JqZYkL4kMBZXimEjZdRgKywMT2Crjar2RA4WD6ODmEAWVmsTnctAKAGQgDegKN5HOGiu9h%2B9yIJ3NZEBNghIOYPnBkOhsHdaHhSNRaMoajyvGWBjEBWlDhw6QmWWsEHGaQKxQ8Kre1RIdQazRUztdwK4YrQnqhEOL4v6OIJxLJlJR%2BMdpEGBchwPDIAWS2WFjL3sBAG0zc8W4NO92Vn1%2BnMu4sVltYK2XbAJ-PewNZ5PlrstgB%2BZuQgC6gP3o-PwdoiFMXGVzEkq-m69gXPskjEbxwRsBAF9AZjWBrIkZgAeWkABpW5MBAmYaTpBkmR4Vl2WBLd115Vg0J7Q1yEBU1D0tS5rlKO4-m4Q82zdFh%2BwhX1-WRdEKJXEFK1LV1vVY6s8WAsDIOgmZvz-ACgMJUCIKgmDCXgECUVuU4gzeeDGVqJC8BQrCVjWDZgAwjTVhVbScKdfD82BaBXx5diIXM-V%2BkHPVxgNREGLRI9YFeNB0E-BISlgGzHNlTzHMyCxEB8ZYTDedRJCsJjCzXHsaMfOcezsvSFGcwMjzi9tWKSziRLEvjJOk2T5ME-92DGMQLgTEAk3vWY4SUKgAB0%2BiocILxeWhTmAHADBICL7C4NALHMBICjEelWVgTVtTwVhRvq5MH0y9EWuBQcAD1gSPHZgU67r3NoAApOEZNgFVJH4Cx8FMUwDC4KbEHwEiSFieQ0BC1TgAsIA&content=N4KABBYEQJYCZQFzQLYE8D6KCGBjAFjAHYCmUANOJFEdimclAOoD2RcJATgGYCuANmACyeQqQpUIUAC5oADg2giCxMpUjQA7i04BrbvxaaAzkjABtSRrDBo8M1HQZteg0YrRa9BwCESx6TBWV0NNDxl5RWYdfVCoMABfKwBdECSQUA1HUVUHVnYuPkFlMTUrKBdY90Y-AKCYtzD1alkFPIbQgGE2bhgAcyg0kCA) you'll see what content can and cannot be parsed.

Specifically, you would expect to see content like this:
```json
{
    "id": "my_machine",
    "name": "Wonderful Machine",
    "type": "Machine",
    "workflows": [
        { "id": "my_workflow", "name": "Best Workflow", "type": "Workflow" }
    ]
}
```

and

```json
{
    "machine": "Wonderful Machine",
    "workflow": "Best Workflow",
    "type": "WorkflowConfig"
}
```

When Langium encounters such content, it will first parse the document, export symbols into the global index, compute the local scope for symbols, link cross references according to the scope, index the resolved cross references and then validate the document.

However, when you input the examples above with our grammar, you will notice that there are a few things that do not work as expected out of the box:

1. References are done based on the `name` property instead of `id`.
2. The workflow cannot properly be referenced as it is only a child of the workflow.
3. If we write multiple grammars, we may need to repeat our terminal rules, i.e., `WS` or `STRING`.

Luckily, one of core principles in Langium is customization.
The core of that customization principle is a dependency injection framework with a set of default modules and their service implementations that can be overwritten in one central place.
Specifically, you define modules where implementations are bound on to a specific property and then create a set of services out of them using the injection mechanism.
Each module has a chance to provide new services, i.e., by specifying new properties, or override existing services by using an existing property name but being used later in the chain of modules.

In order to solve our first problem, we therefore would need to re-bind the default `NameProvider` from Langium and ensure that it uses our `id` property instead of the `name` attribute:


```ts
export interface ModelServicesExtension {
  references: {
    /** override */ NameProvider: NameProvider;
  }
}

export type ModelLanguageServices = LangiumServices & ModelServicesExtension;

export function createMyLangModule(context: {
  shared: ModelLanguagesSharedServices;
}): Module<ModelLanguageServices, ModelServicesExtension> {
  return {
    references: {
      NameProvider: () => new QualifiedIdProvider()
    }
  };
}

// creating the services from the modules
const shared = inject(createDefaultSharedModule(context), …);
const myLanguage = inject(createDefaultModule({ shared }), createMyLangModule({ shared }), …);

// usage: our NameProvider will be lazily created on access
myLanguage.references.NameProvider.getName()
```

In Langium, modules and the generated services can be split into two categories:
- Shared modules and services that mostly relate to the infrastructure such as the language server or the document management and build system.
- Language-specific modules and services that only relate to a single language such as parsing, auto-completion, or validation.

For more details on the dependency injection system and the individual default implementations, we refer to the [Langium documentation](https://langium.org/docs/).

As we have seen in this section, defining a grammar in Langium is very straight-forward.
However, there are certain services and capabilities that may be very common and can be shared through custom modules without having to re-implement them everytime.
Using Model Hub, we therefore offer some modules to support you in the implementation of JSON-based languages and also provide shared services and classes to ease the integration into the overall Model Hub architecture.
There is some future work on our road map to support the generation of a JSON-based grammar and overall integration based on a set of Typescript interface that represent the semantic model to make the definition of your modeling language even more efficient.
Of course, you are not limited to JSON-based grammars and there are also plans to support a YAML-like syntax.
However, if you are very keen on the textual representation of your grammar, you will always be able to simply define your own Langium grammar.

To see how we can integrate your modeling language into the Model Hub, see the next section.
