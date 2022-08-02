# Тестирование

Автоматизированное тестирование считается неотъемлемой частью любой серьезной разработки программного обеспечения. Автоматизация 
позволяет легко и быстро повторять отдельные тесты или наборы тестов в процессе разработки. Это помогает обеспечить соответствие 
релизов целям качества и производительности. Автоматизация помогает увеличить охват и обеспечивает более быстрый цикл обратной 
связи с разработчиками. Автоматизация повышает производительность труда отдельных разработчиков и обеспечивает выполнение тестов 
на критических этапах жизненного цикла разработки, таких как проверка контроля исходного кода, интеграция функций и выпуск версии.

Такие тесты часто бывают разных типов, включая модульные тесты, сквозные (e2e) тесты, интеграционные тесты и так далее. 
Несмотря на неоспоримые преимущества, их установка может быть утомительной. Nest стремится продвигать передовые методы 
разработки, включая эффективное тестирование, поэтому он включает в себя следующие функции, чтобы помочь 
разработчикам и командам создавать и автоматизировать тесты. Nest:

- автоматически создает стандартные модульные тесты для компонентов и e2e-тесты для приложений
- предоставляет инструментарий по умолчанию (например, тестовый загрузчик, который создает изолированный модуль/загрузчик приложения)
- обеспечивает интеграцию с [Jest](https://github.com/facebook/jest) и [Supertest](https://github.com/visionmedia/supertest) 
  "из коробки", оставаясь при этом независимым от инструментов тестирования
- делает систему инъекции зависимостей Nest доступной в тестовой среде для легкого мокинга компонентов.

Как уже упоминалось, вы можете использовать любой **тестовый фреймворк**, который вам нравится, поскольку Nest не навязывает 
никаких конкретных инструментов. Просто замените необходимые элементы (например, test runner), и вы по-прежнему 
будете пользоваться преимуществами готовых средств тестирования Nest.

## Установка

Чтобы начать работу, сначала установите необходимый пакет:

```bash
$ npm i --save-dev @nestjs/testing
```

## Юнит тестирование

В следующем примере мы тестируем два класса: `CatsController` и `CatsService`. Как уже упоминалось, 
[Jest](https://github.com/facebook/jest) предоставляется в качестве фреймворка тестирования по умолчанию. Он служит 
для запуска тестов, а также предоставляет функции assert и утилиты test-double, которые помогают в мокинге, шпионаже(spying) и т.д. 
В следующем базовом тесте мы вручную инстанцируем эти классы и убеждаемся, что контроллер и сервис выполняют свой API-контракт.

<div class="filename">cats.controller.spec.ts</div>

```typescript
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;
  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });
  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);
      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```

> Храните файлы тестов рядом с тестируемыми классами. Файлы тестирования должны иметь суффикс `.spec` или `.test`.

Поскольку приведенный выше пример тривиален, мы на самом деле не тестируем ничего специфичного для Nest. Более того, 
мы даже не используем инъекцию зависимостей (обратите внимание, что мы передаем экземпляр `CatsService` нашему `catsController`). 
Такая форма тестирования - когда мы вручную инстанцируем тестируемые классы - часто называется **изолированным тестированием**, 
поскольку оно не зависит от фреймворка. Давайте познакомимся с некоторыми более продвинутыми возможностями, которые помогут 
вам тестировать приложения, более широко использующие возможности Nest.

## Утилиты для тестирования

Пакет `@nestjs/testing` предоставляет набор утилит, которые обеспечивают более надежный процесс тестирования. Давайте 
перепишем предыдущий пример, используя встроенный класс `Test`:

<div class="filename">cats.controller.spec.ts</div>

```typescript
import { Test } from '@nestjs/testing';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;
  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
        controllers: [CatsController],
        providers: [CatsService],
      }).compile();
    catsService = moduleRef.get<CatsService>(CatsService);
    catsController = moduleRef.get<CatsController>(CatsController);
  });
  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);
      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```


Класс `Test` полезен для создания контекста выполнения приложения, который по сути имитирует рантайм Nest, 
но предоставляет вам хуки, облегчающие управление экземплярами класса, включая мокинг и переопределение. Класс `Test` 
имеет метод `createTestingModule()`, который принимает в качестве аргумента объект метаданных модуля (тот же объект, который 
вы передаете декоратору `@Module()`). Этот метод возвращает экземпляр `TestingModule`, который, в свою очередь, предоставляет 
несколько методов. Для модульных тестов важным является метод `compile()`. Этот метод загружает модуль с его зависимостями 
(подобно тому, как приложение загружается в обычном файле `main.ts` с помощью `NestFactory.create()`), и возвращает модуль, 
готовый к тестированию.

> Метод `compile()` является **асинхронным** и поэтому должен быть ожидаемым (await). После компиляции 
> модуля вы можете получить любой **статический** экземпляр, который в нем объявлен (контроллеры и провайдеры), используя 
> метод `get()`.

`TestingModule` наследуется от класса [module reference](/guide/fundamentals/module-ref), и поэтому способен динамически 
резолвить скопированные провайдеры (transient или request-scoped). Для этого используется метод `resolve()` (метод `get()` 
может получить только статические экземпляры).

```typescript
const moduleRef = await Test.createTestingModule({
  controllers: [CatsController],
  providers: [CatsService],
}).compile();
catsService = await moduleRef.resolve(CatsService);
```

> Метод `resolve()` возвращает уникальный экземпляр провайдера из его собственного поддерева **DI контейнера**. Каждое 
> поддерево имеет уникальный идентификатор контекста. Таким образом, если вы вызовете этот метод несколько раз и сравните 
> ссылки на экземпляры, вы увидите, что они не равны.

>Узнайте больше о возможностях ссылок на модуль [здесь](/guide/fundamentals/module-ref).

Вместо того чтобы использовать продакшн версию любого провайдера, вы можете переопределить его с помощью 
[пользовательского провайдера](/guide/fundamentals/custom-providers) для целей тестирования. Например, вы можете имитировать 
службу базы данных вместо того, чтобы подключаться к реальной базе данных. Мы рассмотрим переопределения в следующем 
разделе, но они доступны и для модульных тестов.

<demo-component></demo-component>

## Авто мокинг

Nest также позволяет вам определить фабрику моков для применения ко всем отсутствующим зависимостям. Это полезно 
в тех случаях, когда в классе имеется большое количество зависимостей, и создание всех из них займет много времени и настроек. 
Чтобы воспользоваться этой функцией, метод `createTestingModule()` нужно соединить с методом `useMocker()`, передав ему фабрику 
для моков зависимостей. Эта фабрика может принимать необязательный токен, который является токеном экземпляра, любой токен, 
который действителен для поставщика Nest, и возвращает реализацию мока. Ниже приведен пример создания общего мока с помощью 
[`jest-mock`](https://www.npmjs.com/package/jest-mock) и конкретного мока для `CatsService` с помощью `jest.fn()`.

```typescript
const moduleMocker = new ModuleMocker(global);
describe('CatsController', () => {
  let controller: CatsController;
  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      controllers: [CatsController],
    })
    .useMocker((token) => {
      if (token === CatsService) {
        return { findAll: jest.fn().mockResolveValue(results) };
      }
      if (typeof token === 'function') {
        const mockMetadata = moduleMocker.getMetadata(token) as MockFunctionMetadata<any, any>;
        const Mock = moduleMocker.generateFromMetadata(mockMetadata);
        return new Mock();
      }
    })
    .compile();
    
    controller = moduleRef.get(CatsController);
  });
})
```

> Можно также передавать непосредственно общую фабрику моков, например `createMock` из [`@golevelup/ts-jest`](https://github.com/golevelup/nestjs/tree/master/packages/testing).

Вы также можете получить эти моки из контейнера тестирования, как вы обычно делаете с пользовательскими провайдерами, 
`moduleRef.get(CatsService)`.

## Сквозное (End-to-end) тестирование

В отличие от модульного тестирования, которое фокусируется на отдельных модулях и классах, сквозное тестирование (e2e) 
охватывает взаимодействие классов и модулей на более агрегированном уровне - ближе к тому, как конечные пользователи будут 
взаимодействовать с производственной системой. По мере роста приложения становится трудно вручную тестировать сквозное 
поведение каждой конечной точки API. Автоматизированные сквозные тесты помогают нам убедиться, что общее поведение системы 
корректно и соответствует требованиям проекта. Для выполнения e2e-тестов мы используем конфигурацию, аналогичную той, 
которую мы только что рассмотрели в **юнит-тестировании**. Кроме того, Nest позволяет легко использовать 
библиотеку [Supertest](https://github.com/visionmedia/supertest) для имитации HTTP-запросов.

<div class="filename">cats.e2e-spec.ts</div>

```typescript
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';
describe('Cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };
  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [CatsModule],
    })
      .overrideProvider(CatsService)
      .useValue(catsService)
      .compile();
    app = moduleRef.createNestApplication();
    await app.init();
  });
  it(`/GET cats`, () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect({
        data: catsService.findAll(),
      });
  });
  afterAll(async () => {
    await app.close();
  });
});
```

> Если вы используете [Fastify](/guide/techniques/performance) в качестве HTTP-адаптера, он требует немного другой 
> конфигурации и имеет встроенные возможности тестирования:
> ```typescript
> let app: NestFastifyApplication;
>
> beforeAll(async () => {
>   app = moduleRef.createNestApplication<NestFastifyApplication>(
>     new FastifyAdapter(),
>   );
>
>   await app.init();
>   await app.getHttpAdapter().getInstance().ready();
> });
>
> it(`/GET cats`, () => {
>   return app
>     .inject({
>       method: 'GET',
>       url: '/cats',
>     })
>     .then((result) => {
>       expect(result.statusCode).toEqual(200);
>       expect(result.payload).toEqual(/* expectedPayload */);
>     });
> });
>  
> afterAll(async () => {
>   await app.close();
> });
> ```

В этом примере мы развиваем некоторые из описанных ранее концепций. В дополнение к методу `compile()`, который мы использовали 
ранее, мы теперь используем метод `createNestApplication()` для создания полной среды выполнения Nest. Мы сохраняем ссылку 
на запущенное приложение в нашей переменной `app`, чтобы использовать его для имитации HTTP-запросов.

Мы имитируем HTTP-тесты с помощью функции `request()` из Supertest. Мы хотим, чтобы эти HTTP-запросы направлялись к нашему 
запущенному приложению Nest, поэтому мы передаем функции `request()` ссылку на HTTP-сервер, который лежит в основе Nest 
(который, в свою очередь, может быть предоставлен платформой Express). Отсюда конструкция `request(app.getHttpServer())`. 
Вызов `request()` передает нам обернутый HTTP-сервер, теперь подключенный к приложению Nest, который предоставляет методы 
для имитации реального HTTP-запроса. Например, использование `request(...).get('/cats')` инициирует запрос к приложению Nest, 
который идентичен **актуальному** HTTP-запросу типа `get '/cats'`, поступающему по сети.

В этом примере мы также предоставляем альтернативную (test-double) реализацию `CatsService`, которая просто возвращает жестко 
закодированное значение, которое мы можем проверить. Используйте `overrideProvider()` для предоставления такой альтернативной 
реализации. Аналогично, Nest предоставляет методы для переопределения охранников, перехватчиков, фильтров и pipes с помощью 
методов `overrideGuard()`, `overrideInterceptor()`, `overrideFilter()` и `overridePipe()` соответственно.

Каждый из переопределенных методов возвращает объект с 3 различными методами, которые повторяют методы, описанные 
для [Пользовательский провайдеров](/guide/fundamentals/custom-providers):

- `useClass`: вы предоставляете класс, который будет инстанцирован, чтобы предоставить экземпляр для переопределения 
  объекта (провайдер, охранник и т.д.).
- `useValue`: вы предоставляете экземпляр, который будет переопределять объект.
- `useFactory`: вы предоставляете функцию, возвращающую экземпляр, который будет переопределять объект.

Каждый из методов переопределения, в свою очередь, возвращает экземпляр `TestingModule` и, таким образом, может быть 
связан с другими методами в [fluent style](https://en.wikipedia.org/wiki/Fluent_interface). Вы должны использовать `compile()` 
в конце такой цепочки, чтобы заставить Nest инстанцировать и инициализировать модуль.

Кроме того, иногда вы можете захотеть предоставить пользовательский логгер, например, при выполнении тестов (к примеру, на CI-сервере). 
Используйте метод `setLogger()` и передайте объект, соответствующий интерфейсу `LoggerService`, чтобы указать `TestModuleBuilder`, 
как вести журнал во время тестов (по умолчанию в консоль будут выводиться только журналы "ошибок").

Скомпилированный модуль имеет несколько полезных методов, описанных в следующей таблице:

<table>
  <tr>
    <td>
      <code>createNestApplication()</code>
    </td>
    <td>
      Создает и возвращает приложение Nest (экземпляр <code>INestApplication</code>) на основе заданного модуля. Обратите внимание, 
      что вы должны вручную инициализировать приложение с помощью метода <code>init()</code>.
    </td>
  </tr>
  <tr>
    <td>
      <code>createNestMicroservice()</code>
    </td>
    <td>
      Создает и возвращает микросервис Nest (экземпляр <code>INestMicroservice</code>) на основе заданного модуля.
    </td>
  </tr>
  <tr>
    <td>
      <code>get()</code>
    </td>
    <td>
      Получает статический экземпляр контроллера или провайдера (включая guards, filters и т.д.), доступного в контексте приложения. Наследуется от класса <a href="/guide/fundamentals/module-ref">ссылки на модуль</a>.    
    </td>
  </tr>
  <tr>
     <td>
      <code>resolve()</code>
    </td>
    <td>
      Получает динамически созданный экземпляр (request или transient) контроллера или провайдера (включая guards, filters и т.д.), доступного в контексте приложения. Наследуется от класса <a href="/guide/fundamentals/module-ref">ссылки на модуль</a>. 
    </td>
  </tr>
  <tr>
    <td>
      <code>select()</code>
    </td>
    <td>
      Перемещается по графу зависимостей модуля; может использоваться для получения конкретного экземпляра из выбранного модуля (используется вместе со строгим режимом (<code>strict: true</code>) в методе <code>get()</code>).
    </td>
  </tr>
</table>

> Храните свои тестовые файлы e2e в директории `test`. Файлы тестирования должны иметь суффикс `.e2e-spec`.

## Переопределение глобально зарегистрированных сервисов

Если у вас есть глобально зарегистрированный guard (или pipe, interceptor, или filter), вам нужно сделать еще 
несколько шагов, чтобы переопределить такой сервис. Напомним, что первоначальная регистрация выглядит следующим образом:

```typescript
providers: [
  {
    provide: APP_GUARD,
    useClass: JwtAuthGuard,
  },
],
```

Это регистрация guard как "мульти"-провайдера через токен `APP_*`. Чтобы иметь возможность заменить `JwtAuthGuard` здесь, 
регистрация должна использовать существующий провайдер в этом слоте:

```typescript
providers: [
  {
    provide: APP_GUARD,
    useExisting: JwtAuthGuard,
    // ^^^^^^^^ обратите внимание на использование 'useExisting' вместо 'useClass'
  },
  JwtAuthGuard,
],
```

> Измените `useClass` на `useExisting`, чтобы ссылаться на зарегистрированный провайдер вместо того, чтобы Nest 
> инстанцировал его под токеном.

Теперь `JwtAuthGuard` виден Nest как обычный провайдер, который можно переопределить при создании `TestingModule`:

```typescript
const moduleRef = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideProvider(JwtAuthGuard)
  .useClass(MockAuthGuard)
  .compile();
```

Теперь ваши тесты могут использовать`MockAuthGuard` при каждом запросе.

## Тестирование экземпляров, скопированных на запрос

Провайдеры [Request-scoped](/guide/fundamentals/injection-scopes) создаются уникально для каждого входящего **запроса**. 
Экземпляр удаляется сборщиком мусора после завершения обработки запроса. Это создает проблему, поскольку мы не можем получить 
доступ к поддереву инъекции зависимостей, созданному специально для текущего запроса.

Мы знаем (на основании разделов выше), что метод `resolve()` можно использовать для получения динамически инстанцированного 
класса. Также, как описано [здесь](/guide/fundamentals/module-ref#разрешение-scoped-проваидеров), мы знаем, 
что можем передавать уникальный идентификатор контекста для управления жизненным циклом поддерева контейнера DI. Как мы 
можем использовать это в контексте тестирования?

Стратегия заключается в том, чтобы заранее сгенерировать идентификатор контекста и заставить Nest использовать именно этот 
идентификатор для создания поддерева для всех входящих запросов. Таким образом, мы сможем получить экземпляры, созданные 
для проверенного запроса.

Чтобы добиться этого, используйте `jest.spyOn()` для `ContextIdFactory`:

```typescript
const contextId = ContextIdFactory.create();
jest
  .spyOn(ContextIdFactory, 'getByRequest')
  .mockImplementation(() => contextId);
```

Теперь мы можем использовать `contextId` для доступа к одному сгенерированному поддереву DI-контейнера при любом 
последующем запросе.

```typescript
catsService = await moduleRef.resolve(CatsService, contextId);
```
