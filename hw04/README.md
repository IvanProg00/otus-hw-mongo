# Базовые запросы к MongoDB

## Цель

- Ускорить работу приложения

## Задание

- [ ] Добавить индексы в свой проект, сравнить производительность запросов

## Описание выполнения

Я экспрементировал над коллекцией фильмов состоящий из 23413 записей.

Я сделал выборку по записям, у которых год больше чем 1980

```js
db.movies.explain("executionStats").find({year: {$gt: 1980}}, {year: 1})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 19399,
    executionTimeMillis: 24,
    totalKeysExamined: 0,
    totalDocsExamined: 23413,
    executionStages: {
      stage: 'COLLSCAN',
      filter: { year: { '$gt': 1980 } },
      nReturned: 19399,
      executionTimeMillisEstimate: 2,
      works: 23415,
      advanced: 19399,
      needTime: 4015,
      needYield: 0,
      saveState: 23,
      restoreState: 23,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 23413
    }
  },
  ...
}
```

Затем я создал индекс и снова сделал выборку

```js
db.movies.createIndex({year: 1})
db.movies.explain("executionStats").find({year: {$gt: 1980}})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 19399,
    executionTimeMillis: 22,
    totalKeysExamined: 19399,
    totalDocsExamined: 19399,
    executionStages: {
      stage: 'FETCH',
      nReturned: 19399,
      executionTimeMillisEstimate: 1,
      works: 19400,
      advanced: 19399,
      needTime: 0,
      needYield: 0,
      saveState: 19,
      restoreState: 19,
      isEOF: 1,
      docsExamined: 19399,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 19399,
        executionTimeMillisEstimate: 1,
        works: 19400,
        advanced: 19399,
        needTime: 0,
        needYield: 0,
        saveState: 19,
        restoreState: 19,
        isEOF: 1,
        keyPattern: { year: 1 },
        indexName: 'year_1',
        isMultiKey: false,
        multiKeyPaths: { year: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { year: [ '(1980, inf.0]' ] },
        keysExamined: 19399,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  ...
}
```

Без индекса я потратил 24 милисикунды, а вместе с индексом 22 милисикунды. Разница не большая

Я решил создать запросы потяжелее

Сначало без индекса

```js
db.movies.explain("executionStats").find({$and: [{year: {$gt: 1980}}, {year: {$lt: 1990}}]})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 1905,
    executionTimeMillis: 24,
    totalKeysExamined: 0,
    totalDocsExamined: 23413,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [ { year: { '$lt': 1990 } }, { year: { '$gt': 1980 } } ]
      },
      nReturned: 1905,
      executionTimeMillisEstimate: 2,
      works: 23415,
      advanced: 1905,
      needTime: 21509,
      needYield: 0,
      saveState: 23,
      restoreState: 23,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 23413
    }
  },
  ...
}
```

Теперь с индексом

```js
db.movies.explain("executionStats").find({$and: [{year: {$gt: 1980}}, {year: {$lt: 1990}}]})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 1905,
    executionTimeMillis: 2,
    totalKeysExamined: 1905,
    totalDocsExamined: 1905,
    executionStages: {
      stage: 'FETCH',
      nReturned: 1905,
      executionTimeMillisEstimate: 0,
      works: 1906,
      advanced: 1905,
      needTime: 0,
      needYield: 0,
      saveState: 1,
      restoreState: 1,
      isEOF: 1,
      docsExamined: 1905,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 1905,
        executionTimeMillisEstimate: 0,
        works: 1906,
        advanced: 1905,
        needTime: 0,
        needYield: 0,
        saveState: 1,
        restoreState: 1,
        isEOF: 1,
        keyPattern: { year: 1 },
        indexName: 'year_1',
        isMultiKey: false,
        multiKeyPaths: { year: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { year: [ '(1980, 1990)' ] },
        keysExamined: 1905,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  ...
}
```

И здесь мы уже видем скорость, без индекса 24 миллисекунды, а с индексом 2 миллисекунды.

Теперь я решил попробовать использовать функцию aggregate, и добавить индекс для поля languages

Без индексов

```js
db.movies.explain("executionStats").aggregate({$match: {$or: [{year: {$gt: 1980}}, {title: /eo/}]}})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 19434,
    executionTimeMillis: 24,
    totalKeysExamined: 0,
    totalDocsExamined: 23413,
    executionStages: {
      stage: 'SUBPLAN',
      nReturned: 19434,
      executionTimeMillisEstimate: 3,
      works: 23415,
      advanced: 19434,
      needTime: 3980,
      needYield: 0,
      saveState: 23,
      restoreState: 23,
      isEOF: 1,
      inputStage: {
        stage: 'COLLSCAN',
        filter: {
          '$or': [ { year: { '$gt': 1980 } }, { title: { '$regex': 'eo' } } ]
        },
        nReturned: 19434,
        executionTimeMillisEstimate: 3,
        works: 23415,
        advanced: 19434,
        needTime: 3980,
        needYield: 0,
        saveState: 23,
        restoreState: 23,
        isEOF: 1,
        direction: 'forward',
        docsExamined: 23413
      }
    }
  },
  ...
}
```

С индексом

```js
db.movies.explain("executionStats").aggregate({$match: {$or: [{year: {$gt: 1980}}, {title: /eo/}]}})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 19434,
    executionTimeMillis: 45,
    totalKeysExamined: 42812,
    totalDocsExamined: 19434,
    executionStages: {
      stage: 'SUBPLAN',
      nReturned: 19434,
      executionTimeMillisEstimate: 5,
      works: 42814,
      advanced: 19434,
      needTime: 23379,
      needYield: 0,
      saveState: 42,
      restoreState: 42,
      isEOF: 1,
      inputStage: {
        stage: 'FETCH',
        nReturned: 19434,
        executionTimeMillisEstimate: 4,
        works: 42814,
        advanced: 19434,
        needTime: 23379,
        needYield: 0,
        saveState: 42,
        restoreState: 42,
        isEOF: 1,
        docsExamined: 19434,
        alreadyHasObj: 0,
        inputStage: {
          stage: 'OR',
          nReturned: 19434,
          executionTimeMillisEstimate: 2,
          works: 42814,
          advanced: 19434,
          needTime: 23379,
          needYield: 0,
          saveState: 42,
          restoreState: 42,
          isEOF: 1,
          dupsTested: 19599,
          dupsDropped: 165,
          inputStages: [
            {
              stage: 'IXSCAN',
              nReturned: 19399,
              executionTimeMillisEstimate: 0,
              works: 19400,
              advanced: 19399,
              needTime: 0,
              needYield: 0,
              saveState: 42,
              restoreState: 42,
              isEOF: 1,
              keyPattern: { year: 1 },
              indexName: 'year_1',
              isMultiKey: false,
              multiKeyPaths: { year: [] },
              isUnique: false,
              isSparse: false,
              isPartial: false,
              indexVersion: 2,
              direction: 'forward',
              indexBounds: { year: [ '(1980, inf.0]' ] },
              keysExamined: 19399,
              seeks: 1,
              dupsTested: 0,
              dupsDropped: 0
            },
            {
              stage: 'IXSCAN',
              filter: {
                '$or': [ { title: { '$regex': 'eo' } } ]
              },
              nReturned: 200,
              executionTimeMillisEstimate: 2,
              works: 23414,
              advanced: 200,
              needTime: 23213,
              needYield: 0,
              saveState: 42,
              restoreState: 42,
              isEOF: 1,
              keyPattern: { title: 1 },
              indexName: 'title_1',
              isMultiKey: false,
              multiKeyPaths: { title: [] },
              isUnique: false,
              isSparse: false,
              isPartial: false,
              indexVersion: 2,
              direction: 'forward',
              indexBounds: { title: [ '["", {})', '[/eo/, /eo/]' ] },
              keysExamined: 23413,
              seeks: 1,
              dupsTested: 0,
              dupsDropped: 0
            }
          ]
        }
      }
    }
  },
  ...
}
```

В данном случае индекс медленне чем без индекса

В прошлом случае я использовал или (\$or), а теперь попробую тоже самое с и ($and)

Без индексов

```js
db.movies.explain("executionStats").aggregate({$match: {$and: [{year: {$gt: 1980}}, {title: /eo/}]}})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 165,
    executionTimeMillis: 33,
    totalKeysExamined: 0,
    totalDocsExamined: 23413,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [ { year: { '$gt': 1980 } }, { title: { '$regex': 'eo' } } ]
      },
      nReturned: 165,
      executionTimeMillisEstimate: 2,
      works: 23415,
      advanced: 165,
      needTime: 23249,
      needYield: 0,
      saveState: 23,
      restoreState: 23,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 23413
    }
  },
  ...
}
```

С индексом

```js
db.movies.explain("executionStats").aggregate({$match: {$and: [{year: {$gt: 1980}}, {title: /eo/}]}})
```

```json
{
  ...
  executionStats: {
    executionSuccess: true,
    nReturned: 165,
    executionTimeMillis: 53,
    totalKeysExamined: 19399,
    totalDocsExamined: 19399,
    executionStages: {
      stage: 'FETCH',
      filter: { title: { '$regex': 'eo' } },
      nReturned: 165,
      executionTimeMillisEstimate: 8,
      works: 19400,
      advanced: 165,
      needTime: 19234,
      needYield: 0,
      saveState: 29,
      restoreState: 29,
      isEOF: 1,
      docsExamined: 19399,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 19399,
        executionTimeMillisEstimate: 2,
        works: 19400,
        advanced: 19399,
        needTime: 0,
        needYield: 0,
        saveState: 29,
        restoreState: 29,
        isEOF: 1,
        keyPattern: { year: 1 },
        indexName: 'year_1',
        isMultiKey: false,
        multiKeyPaths: { year: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { year: [ '(1980, inf.0]' ] },
        keysExamined: 19399,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  ...
}
```

В данном примере индекс опять проиграл без индексам. Но в данном случае мы видим что индекс был использован только по полю year, а по title был использован FETCH (проверка поля не по индексу)
