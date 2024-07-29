This chapter covers setup and use DatX data store within Angular project and how to use it in tests. If you are not familiar with it yet, resources below can help.

## Resources

You can read up on DatX at the official [docs site](https://datx.dev/docs/getting-started/installation). Infinum also published the [working with JSON API](https://infinum.com/the-capsized-eight/working-with-JSON-API) article with examples. We also have an [example repository](https://github.com/infinum/js-angular-datx-example) that you can take a look at when in doubt about how to set up DatX in an Angular project or visit Angular specific [setup section](https://www.npmjs.com/package/@datx/jsonapi-angular#setup).

## Why use DatX

Apart from providing possible single source of truth for any shared data with a basic querying and referencing support, it also provides options to extend any entity with mixins such as [withActions](https://datx.dev/docs/mixins/with-actions), [withMeta](https://datx.dev/docs/mixins/with-meta), [withPatches](https://datx.dev/docs/mixins/with-patches), which enhance entities with usefull utility functions, or network related ones like [withNetwork](https://datx.dev/docs/mixins/with-network), [jsonapi](https://datx.dev/docs/mixins/jsonapi-mixin) or, useful in Angular's case, [jsonapiAngular](https://datx.dev/docs/mixins/jsonapi-angular-mixin). Especially with [jsonapi](https://datx.dev/docs/mixins/jsonapi-mixin) and/or [jsonapiAngular](https://datx.dev/docs/mixins/jsonapi-angular-mixin), interacting with [JSONAPI standard](https://jsonapi.org/) compliat backend can be much easier then building requests manually and more managable than building API related code with some sort of generator.

## Setup

Let's start by installing all DatX related dependencies we will need:

```
pnpm i -E @datx/core @datx/jsonapi @datx/jsonapi-angular
```

Since we didn't include mobx (because it doesn't play nice with RxJS), we need to add a little boilerplate to work around that. First we need to instruct DatX not to use mobx, by adding `'@datx/core/disable-mobx'` before App bootstrap:

```ts
// src/main.js AND src/test.js
import '@datx/core/disable-mobx';
```

Next, we need to overwrite mobx path so that it can be resolved by datx, albeit to an **empty** file:

```ts
// /tsconfig.json
{
	...
	"compilerOptions": {
		...
		"paths": {
			...
			"mobx": ["./mobx.js"]
		}
	}
}
```

Lastly, we bypass `@datx/core/disable-mobx` related import warning by adding its non ECMAScript module dependency to a whitelist:

```ts
// angular.json
{
	...
	"architect": {
		...
		"build": {
			...
			"options": {
				...
				"allowedCommonJsDependencies": ["@datx/utils"],
			}
		}
	}
}
```

## Defining models and their relationships

This topic is heavily described in the resources above, please refer to them. By convention, you can put all models in `src/app/models` directory and collections to `src/app/collections` of your Angular project. Once you are finished describing all *Resource Objects* you can move to the next section, just remember that you will most likely want to use *jsonapiAngular* so that you can work with observables when calling async methods on models.

## Store setup and injection

To create and provide a single instance of Collection across entire Angular app, you must provide `APP_COLLECTION` injection token, in root module, like so:

```ts
// app.module.ts
import { APP_COLLECTION } from '@datx/jsonapi-angular';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { AppCollection } from '<path-to-collection-definition>';

@NgModule({
	declarations: [AppComponent],
	imports: [
		...
	],
	providers: [
    ...
    {
      provide: APP_COLLECTION,
      useValue: new AppCollection();
    }
    ...
  ],
  bootstrap: [AppComponent],
}
export class AppModule {}
```

Whenever you want to work with the collection from within Angular's DI container, you can simply use the same token above to inject the collection instance.

```ts
// example.service.ts
import { APP_COLLECTION } from '<path-to-token-definition>';
import { Inject, Injectable } from '@angular/core';
import { AppCollection } from '<path-to-collection-definition>';

@Injectable({
  providedIn: 'root',
})
export class ExampleService {
  protected readonly collection: AppCollection = inject(APP_COLLECTION) // app.module.ts scoped instance of AppCollection
}
```

When testing consumers of APP\_COLLECTION, just inject the collection under the same token again, as with any other dependency:

```ts
// example.service.spec.ts
import { AppCollection } from '<path-to-collection-definition>';
import { APP_COLLECTION } from '<path-to-token-definition>';
import { TestBed } from '@angular/core/testing';
import { ExampleService } from './example.service';

describe('ExampleService', () => {
  let service: ExampleService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        {
          provide: APP_COLLECTION,
          useValue: new AppCollection(), // empty mock collection
        },
      ],
    });
    service = TestBed.inject(ExampleService);
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });
});
```

## Abstracting entity related logic into separate service

[@datx/jsonapi-angular](https://www.npmjs.com/package/@datx/jsonapi-angular) package exposes a `CollectionService` in order to abstract away a bit of the tediousness of working with DatX and/or bind some special logic to a given entity. Since many of the Collection's methods require an Entity type as a parameter you can create separate service that will hide this small implementation detail for you. One way to do that would be to extend `CollectionService` like so:

```ts
// entity.service.ts
import { Inject, Injectable } from '@angular/core';
import { AppCollection } from '<path-to-collection-definition>';
import { APP_COLLECTION } from '<path-to-token-definition>';
import { Entity } from '<path-to-entity-definition>';
import { CollectionService } from '@datx/jsonapi-angular';

@Injectable({
  providedIn: 'root',
})
export class EntityService extends CollectionService<Project> {
  protected readonly ctor = Entity;
}
```

Such service can be injected from within DI container like any other:

```ts
// entity-list.component.ts
@Component({
	...
})
export class EntityListComponent {
  private readonly entity = inject(EntityService);

	public readonly entities$ = this.entity.getAllModels();
}
```

When testing these abstractions, it advisable to create test doubles that replace any methods that would make API calls with an implementation which works with in memory collection. For this use-case the library exposes `CollectionTestingService`

Where `ExtractPublic` is covered in [Testing chapter](/books/frontend/angular/angular-guidelines-and-best-practices/testing#typing-test-doubles). The `asyncData` and `asyncError` helpers make sure, that values provided are observed on next macrotask at soonest. This makes methods that would normally perform an HTTP call more alike their non-double counterparts.

```ts
// helpers.ts
import { asyncScheduler, Observable, of, throwError } from 'rxjs';
import { observeOn } from 'rxjs/operators';

export function asyncData<TData>(data: TData): Observable<TData> {
  return of(data).pipe(observeOn(asyncScheduler));
}

export function asyncError(err: Error): Observable<never> {
  return throwError(err).pipe(observeOn(asyncScheduler));
}
```

Subsequently, you can then create doubles for entity specific services by extending the **CollectionTestingService**:

```ts
// entity.testing.service.ts
import { Inject, Injectable } from '@angular/core';
import { AppCollection } from '<path-to-collection>';
import { APP_COLLECTION } from '<path-to-token-definition>';
import { Entity } from '<path-to-entity-model-definition>';
import { EntityService } from '<path-to-entity-service>';
import { ExtractPublic } from '<path-to-extract-public-type>';
import { CollectionTestingService } from '<path-to-collection-testing-service>';

@Injectable()
export class EntityTestingService
  extends CollectionTestingService<Entity>
  implements ExtractPublic<EntityService>
{
  protected ctor = Entity;
}
```

These can be used in tests for consumers of original entity specific service:

```ts
// entity-list.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { AppCollection } from '<path-to-collection>';
import { APP_COLLECTION } from '<path-to-token-definition>';
import { Entity } from '<path-to-entity>';
import { EntityService } from '<path-to-entity-service>';
import { EntityTestingService } from '<path-to-entity-service-double>';
import { EntityListComponent } from '<path-to-component>';

describe('EntityListComponent', () => {
  let component: EntityListComponent;
  let fixture: ComponentFixture<EntityListComponent>;
  let collection: AppCollection;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [EntityListComponent],
      providers: [
        {
          provide: APP_COLLECTION,
          useValue: new AppCollection(), // empty mock collection
        },
        {
          provide: EntityService,
          useClass: EntityTestingService, // provide the double instead
        },
      ],
    });
  });

  beforeEach(() => {
    collection = TestBed.inject(APP_COLLECTION);
    collection.add({ name: 'EntityName' }, Entity); // entities available from entity-list.component.ts #getAllModels call
    fixture = TestBed.createComponent(EntityListComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should be created', () => {
    expect(component).toBeTruthy();
  });
});
```
