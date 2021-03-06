---
lang: en
title: 'Decorators'
keywords: LoopBack 4.0, LoopBack-Next
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Decorators.html
---

A decorator allows you to annotate or modify your class declarations and members
with metadata.

## Introduction

_If you're new to Decorators in TypeScript, see
[here](https://www.typescriptlang.org/docs/handbook/decorators.html) for more
info._

Decorators give LoopBack the flexibility to modify your plain TypeScript classes
and properties in a way that allows the framework to better understand how to
make use of them, without the need to inherit base classes or add functions that
tie into an API.

As a default, LoopBack comes with some pre-defined decorators:

- [Route Decorators](#route-decorators)
- [Dependency Injection](#dependency-injection)
- [Authentication Decorator](#authentication-decorator)
- [Repository Decorators](#repository-decorators)

## Route Decorators

Route decorators are used to expose controller methods as REST API operations.
If you are not familiar with the concept of Route or Controller, please see
[LoopBack Route](Routes.md) and [LoopBack Controller](Controllers.md) to learn
more about them.

By calling a route decorator, you provide OpenAPI specification to describe the
endpoint which the decorated method maps to. You can choose different decorators
accordingly or do a composition of them:

### API Decorator

Syntax:
[`@api(spec: ControllerSpec)`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#api)

`@api` is a decorator for the controller class and is appended just before it's
declared. `@api` is used when you have multiple
[Paths Objects](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#pathsObject)
that contain all path definitions of your controller. Please note the api specs
defined with `@api` will override other api specs defined inside the controller.
For example:

```ts
@api({
  basePath: '/',
  paths: {
    '/greet': {
      get: {
        'x-operation-name': 'greet',
        'x-controller-name': 'MyController',
        parameters: [{name: 'name', schema: {type: 'string'}, in: 'query'}],
        responses: {
          '200': {
            description: 'greeting text',
            content: {
              'application/json': {
                schema: {type: 'string'},
              },
            },
          },
        },
      },
    },
  },
})
class MyController {
  // The operation endpoint defined here will be overriden!
  @get('/greet')
  greet(@param.query.number('limit') name: string) {}
}
app.controller(MyController);
```

A more detailed explanation can be found in
[Specifying Controller APIs](Controllers.md#specifying-controller-apis)

### Operation Decorator

Syntax:
[`@operation(verb: string, path: string, spec?: OperationObject)`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#operation)

`@operation` is a controller method decorator. It exposes a Controller method as
a REST API operation and is represented in the OpenAPI spec as an
[Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#operation-object).
You can specify the verb, path, parameters, and response as a specification of
your endpoint, for example:

```ts
const spec = {
  parameters: [{name: 'name', schema: {type: 'string'}, in: 'query'}],
  responses: {
    '200': {
      description: 'greeting text',
      content: {
        'application/json': {
          schema: {type: 'boolean'},
        },
      },
    },
  },
};
class MyController {
  @operation('HEAD', '/checkExist', spec)
  checkExist(name: string) {}
}
```

### Commonly-used Operation Decorators

Syntax:
[`@get(path: string, spec?: OperationObject)`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#get)

Same Syntax for decorators
[`@post`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#post) ,
[`@put`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#put) ,
[`@patch`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#patch) ,
[`@del`](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#del)

You can call these sugar operation decorators as a shortcut of `@operation`. For
example:

```ts
class MyController {
  @get('/greet', spec)
  greet(name: string) {}
}
```

is equivalent to

```ts
class MyController {
  @operation('GET', '/greet', spec)
  greet(name: string) {}
}
```

### Parameter Decorator

Syntax: see
[API documentation](https://github.com/strongloop/loopback-next/tree/master/packages/openapi-v3/src/decorators/parameter.decorator.ts#L17-L29)

`@param` is applied to controller method parameters to generate an OpenAPI
parameter specification for them.

For example:

```ts
import {get, param} from '@loopback/rest';

const categorySpec = {
  name: 'category',
  in: 'path',
  required: true,
  schema: {type: 'string'},
};

const pageSizeSpec = {
  name: 'pageSize',
  in: 'query',
  required: false,
  schema: {type: 'integer', format: 'int32'},
};

class MyController {
  @get('Pets/{category}')
  list(
    @param(categorySpec) category: string,
    @param(pageSizeSpec) pageSize?: number,
  ) {}
}
```

Writing the whole parameter specification is tedious, so we've created shortcuts
to define the params with the pattern `@param.${in}.${type}(${name})`:

- in: The parameter location. It can be one of the following values: `query`,
  `header`, or `path`.
- type: A
  [common name of OpenAPI primitive data type](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#data-types).
- name: Name of the parameter. It should be a `string`.

A list of available shortcuts for `query` can be found in
[API document](http://apidocs.loopback.io/@loopback%2fdocs/openapi-v3.html#param.query),
along with the shortcuts for `path` and `header`.

An equivalent example using the shortcut decorator would be:

```ts
class MyController {
  @get('/Pets/{category}')
  list(
    @param.path.string('category') category: string,
    @param.query.number('pageSizes') pageSize?: number,
  ) {}
}
```

You can find specific use cases in
[Writing Controller methods](Controllers.md#writing-controller-methods)

_The parameter location cookie is not supported yet, see_
_(https://github.com/strongloop/loopback-next/issues/997)_

### RequestBody Decorator

Syntax: see
[API documentation](https://github.com/strongloop/loopback-next/tree/master/packages/openapi-v3/src/decorators/request-body.decorator.ts#L20-L79)

`@requestBody()` is applied to a controller method parameter to generate OpenAPI
requestBody specification for it.

_Only one parameter can be decorated by `@requestBody` per controller method._

A typical
[OpenAPI requestBody specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#requestBodyObject)
contains properties `description`, `required`, and `content`:

```ts
requestBodySpec: {
  description: 'a user',
  required: true,
  content: {
    'application/json': {...schemaSpec},
    'application/text': {...schemaSpec},
  },
}
```

In order to use `@requestBody` in a parameter type, the model in the parameter
type must be decorated with `@model` and `@property`:

```ts
import {model, property} from '@loopback/repository';
import {Address} from './address.model';

@model()
class User {
  @property()
  firstname: string;
  @property()
  lastname: string;
  @property()
  address: Address;
}
```

_To learn more about decorating models and the corresponding OpenAPI schema, see
[model decorators](#model-decorators)._

The model decorators allow type information of the model to be visible to the
spec generator so that `@requestBody` can be used on the parameter:

```ts
// in file '/src/controllers/user.controller.ts'
import {User} from '../models/user.model';
import {put} from '@loopback/rest';

class UserController {
  @put('/Users/{id}')
  async replaceUser(
    @param.path.string('id') id: string,
    @requestBody() user: User,
  ) {}
}
```

For the simplest use case, you can leave the input of `@requestBody` empty since
we automatically detect the type of `user` and generate the corresponding schema
for it. The default content type is set to be `application/json`.

You can also customize the generated `requestBody` specification in three ways:

- Add the optional fields `description` and `required`

```ts
class MyController {
  @put('/Users/{id}')
  async replaceUser(
    @param.path.string('id') id: string,
    @requestBody({
      description: 'a modified user',
      required: true,
    })
    user: User,
  ) {}
}
```

- Override the content type or define multiple content types

```ts
class MyController {
  @put('/Users/{id}')
  async replaceUser(
    @param.path.string('id') id: string,
    @requestBody({
      content: {
        // leave the schema as empty object, the decorator will generate it for both.
        'application/text': {},
        'application/xml': {},
      },
    })
    user: User,
  ) {}
}
```

- Override the schema specification

```ts
import {UserSchema, User} from '../model/user.schema';

class MyController {
  @put('/Users/{id}')
  async replaceUser(
    @param.path.string('id') id: string,
    @requestBody({
      content: {
        'application/json': UserSchema,
      },
    })
    user: User,
  ) {}
}
```

_We plan to support more `@requestBody` shortcuts in the future. You can track
the feature in story_
_(https://github.com/strongloop/loopback-next/issues/1064)_

## Dependency Injection

`@inject` is a decorator to annotate class properties or constructor arguments
for automatic injection by LoopBack's IoC container.

The injected values are applied to a constructed instance, so it can only be
used on non-static properties or constructor parameters of a Class.

The `@inject` decorator allows you to inject dependencies bound to any
implementation of the [Context](Context.md) object, such as an Application
instance or a request context instance. You can bind values, class definitions,
and provider functions to those contexts and then resolve the values (or the
results of functions that return those values!) in other areas of your code.

```ts
// src/application.ts
import {Application} from '@loopback/core';
import * as fs from 'fs-extra';
import * as path from 'path';

export class MyApp extends RestApplication {
  constructor() {
    super();
    const app = this;
    const widgetConf = JSON.parse(
      fs.readFileSync(path.resolve('widget-config.json')).toString(),
    );
    function logInfo(info: string) {
      console.log(info);
    }
    app.bind('config.widget').to(widgetConf);
    app.bind('logger.widget').to(logInfo);
  }
}
```

Now that we've bound the 'config.widget' key to our configuration object, and
the 'logger.widget' key to the function `logInfo()`, we can inject them in our
WidgetController:

```ts
// src/controllers/widget.controller.ts
import {inject} from '@loopback/context';

export class WidgetController {
  // injection for property
  @inject('logger.widget')
  private logger: Function;

  // injection for constructor parameter
  constructor(
    @inject('config.widget') protected widget: any, // This will be resolved at runtime!
  ) {}
  // etc...
}
```

A few variants of `@inject` are provided to declare special forms of
dependencies:

- `@inject.getter`: inject a getter function that returns a promise of the bound
  value of the key

Syntax: `@inject.getter(bindingKey: string)`.

```ts
import {inject, Getter} from '@loopback/context';
import {UserProfile} from '@loopback/authentication';
import {get} from '@loopback/rest';

export class HelloController {
  constructor(
    @inject.getter('authentication.currentUser')
    private userGetter: Getter<UserProfile>,
  ) {}

  @get('/hello')
  async greet() {
    const user = await this.userGetter();
    return `Hello, ${user.name}`;
  }
}
```

- `@inject.setter`: inject a setter function to set the bound value of the key

Syntax: `@inject.setter(bindingKey: string)`.

```ts
export class HelloController {
  constructor(
    @inject.setter('greeting') private greetingSetter: Setter<string>,
  ) {}

  @get('/hello')
  greet() {
    const defaultGreet = 'Greetings!';
    this.greetingSetter(defaultGreet); // key 'greeting' is now bound to 'Greetings!'
    return defaultGreet;
  }
}
```

- `@inject.tag`: inject an array of values by a pattern or regexp to match
  binding tags

Syntax: `@inject.tag(tag: string | RegExp)`.

```ts
class Store {
  constructor(@inject.tag('store:location') public locations: string[]) {}
}

const ctx = new Context();
ctx.bind('store').toClass(Store);
ctx
  .bind('store.locations.sf')
  .to('San Francisco')
  .tag('store:location');
ctx
  .bind('store.locations.sj')
  .to('San Jose')
  .tag('store:location');
const store = ctx.getSync<Store>('store');
console.log(store.locations); // ['San Francisco', 'San Jose']
```

- `@inject.context`: inject the current context

Syntax: `@inject.context()`.

```ts
class MyComponent {
  constructor(@inject.context() public ctx: Context) {}
}

const ctx = new Context();
ctx.bind('my-component').toClass(MyComponent);
const component = ctx.getSync<MyComponent>('my-component');
// `component.ctx` should be the same as `ctx`
```

**NOTE**: It's recommended to use `@inject` with specific keys for dependency
injection if possible. Use `@inject.context` only when the code needs to access
the current context object for advanced use cases.

For more information, see the [Dependency Injection](Dependency-Injection.md)
section under [LoopBack Core Concepts](Concepts.md)

## Authentication Decorator

Syntax: `@authenticate(strategyName: string, options?: Object)`

Marks a controller method as needing an authenticated user. This decorator
requires a strategy name as a parameter.

Here's an example using 'BasicStrategy': to authenticate user in function
`whoAmI`:

```ts
// src/controllers/who-am-i.controller.ts
import {inject} from '@loopback/context';
import {
  AuthenticationBindings,
  UserProfile,
  authenticate,
} from '@loopback/authentication';
import {get} from '@loopback/rest';

export class WhoAmIController {
  constructor(
    @inject(AuthenticationBindings.CURRENT_USER) private user: UserProfile,
  ) {}

  @authenticate('BasicStrategy')
  @get('/whoami')
  whoAmI(): string {
    return this.user.id;
  }
}
```

For more information on authentication with LoopBack, visit
[here](https://github.com/strongloop/loopback-next/blob/master/packages/authentication/README.md).

## Repository Decorators

As a [domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design)
concept, the repository is a layer between your domain object and data mapping
layers that uses a collection-like interface for accessing domain objects.

In LoopBack, a domain object is usually a TypeScript/JavaScript Class instance.
A typical example of a data mapping layer module could be a database's node.js
driver.

LoopBack repository encapsulates your TypeScript/JavaScript Class instance and
the methods that communicate with your database. It is an interface to implement
data persistence.

Repository decorators are used for defining models (domain objects) for use with
your chosen datasources and for the navigation strategies among models.

If you are not familiar with repository related concepts like `Model`, `Entity`
and `Datasource`, see LoopBack concept [Repositories](Repositories.md) to learn
more.

### Model Decorators

Model is a class that LoopBack builds for you to organize the data that shares
the same configurations and properties. You can use model decorators to define a
model and its properties.

#### Model Decorator

Syntax: `@model(definition?: ModelDefinitionSyntax)`

Model decorator is a class decorator. In LoopBack 4, we inherit the model
definition format from LoopBack 3, which is described in the
[Model definition JSON file](https://loopback.io/doc/en/lb3/Model-definition-JSON-file).
For usage examples, see [Define Models](Repositories.md#define-models)

_Please note we will elaborate more about model and model definition in the
[Model](Model.md) page, and replace the link above with a LoopBack 4 link_

By using a model decorator, you can define a model as your repository's
metadata, which then allows you to choose between two ways of creating the
repository instance:

1. Inject your repository and resolve it with the datasource juggler bridge  
   that's complete with CRUD operations for accessing the model's data. A use
   case can be found in this section:
   [Repository decorator](#repository-decorator)

2. Define your own repository without using the datasource juggler bridge, and
   use an ORM/ODM of your choice.

```ts
// Missing example here
// Will be provided in Model.md
// refer to [example code](https://github.com/strongloop/loopback-next-example/blob/master/services/account-without-juggler/repositories/account/models/Account.ts)
```

#### Property Decorator

Syntax: `@property(definition?: PropertyDefinition)`

The property decorator defines metadata for a property on a Model definition.
The format of property definitions can be found in
[Property definitions](https://loopback.io/doc/en/lb2/Model-definition-JSON-file.html#properties)

For usage examples, see [Define Models](Repositories.md#define-models)

### Repository Decorator

Syntax:

[`@repository(modelOrRepo: string | Class<Repository<Model>> | typeof Entity, dataSource?: string | juggler.DataSource)`](http://apidocs.loopback.io/@loopback%2fdocs/repository.html#repository)

This decorator either injects an existing repository or creates a repository
from a model and a datasource.

The injection example can be found in
[Repository#controller-configuration](Repositories.md#controller-configuration)

To create a repository in a controller, you can define your model and datasource
first, then import them in your controller file:

```ts
// src/controllers/todo.controller.ts
import {Todo} from '../models';
import {db} from '../datasources/db.datasource';
import {repository, EntityCrudRepository} from '@loopback/repository';

export class TodoController {
  @repository(Todo, db)
  todoRepo: EntityCrudRepository<Todo, number>;
  // ...
}
```

If the model or datasource is already bound to the app, you can create the
repository by providing their names instead of the classes. For example:

```ts
// with `datasource` and `Todo` already defined.
app.bind('datasources.db').to(db);
app.bind('models.Todo').to(Todo);

export class TodoController {
  @repository('Todo', 'db')
  repository: EntityCrudRepository<Todo, number>;
  // etc
}
```

### Relation Decorators

_This feature has not yet been released in alpha form. Documentation will be
added here as this feature progresses._

The relation decorator defines the nature of a relationship between two models.

#### Relation Decorator

Syntax: `@relation`

Register a general relation.

_This feature has not yet been released in alpha form. Documentation will be
added here as this feature progresses._

#### Specific Relation Decorator

Syntax:

- `@belongsTo`
- `@hasOne`
- `@hasMany`
- `@embedsOne`
- `@embedsMany`
- `@referencesOne`
- `@referencesMany`

Register a specific relation.

_This feature has not yet been released in alpha form. Documentation will be
added here as this feature progresses._
