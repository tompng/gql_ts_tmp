# gql-sparrow
![logo](logo.svg)

A GraphQL Query Builder for typescript.

Write query once. Types are automatically calculated.

```ts
const query = { articles: { id: true, title: true, author: ['id', 'name'] } } as const
const gqlQuery = buildQuery(query)
type Result = DataTypeFromQuery<typeof query>
// { articles: { id: number; title: string; author: { id: number; name: string } }[] }
```

You don't need to write types by yourself anymore.

You don't need to re-generate type files each time you edit a query.

```sh
% npm install [package-name-here] # not published to npm yet
```

## 1. Generate type from schema.graphql

```sh
% gql-sparrow-gen foobar/schema.graphql foo/bar/generated_types.ts
```

If you have custom scalar types, define it to `foo/bar/customScalarTypes.ts`
```ts
// if `scalar Foo` is `number` in typescript
export TypeFoo = number
```

## 2. Write glue code
```ts
import { buildQuery } from 'gql-sparrow'
import { DataTypeFromQuery, TypeRootQuery } from 'foo/bar/generated_types'
async function myExecuteQuery<Q extends TypeRootQuery>(query: Q) {
  const graphqlQuery = buildQuery(query)
  const result = await executeGraphQLByYourFavoriteLibrary(graphqlQuery)
  return result as DataTypeFromQuery<Q>
}
```

## 3. Write query in plain javascript object
```ts
const query = {
  articles: {
    params: { foo: 'bar' },
    query: {
      id: true,
      author: 'name',
      Title: { field: 'title' },
      Comments: {
        field: 'comments',
        params: { userId: 123 },
        query: ['id', 'text']
      }
    }
  }
} as const
```

## 4. Then you'll get a nice type support.
```ts
const result = await myExecuteQuery(query)
result.articles[0].author // => { name: string }
const { article } = await myExecuteQuery({ article: { params: { id: 1 }, query: ['id', 'title'] } })
article.id // => number
article.title // => string
article.author // => compile error
```

## Examples
```ts
// Query & Types
type QueryResult = DataTypeFromQuery<typeof query>
// {
//   id: number
//   author: { name: string }
//   Title: string
//   Comments: { id: number; text: string }[]
// }
const graphqlQuery = buildQuery(query)
// {
//   articles(foo: "bar") {
//     id
//     author {
//       name
//     }
//     Title: title
//     Comments: comments(userId: 123) {
//       id
//       text
//     }
//   }
// }
```

```ts
// Mutations
import { buildMutationQuery } from 'gql-sparrow'
import { DataTypeFromMutation, TypeMutationQuery } from 'foo/bar/generated_types'
const mutationQuery = { field: 'createArticle', params: { title: 'aaa' }, query: ['id'] }
type QueryResult = DataTypeFromMutation<typeof mutationQuery>
// { createArticle: { id: number } }
const graphqlMutationQuery = buildMutationQuery(mutationQuery)
// mutation {
//   createArticle(title: "aaa") {
//     id
//   }
// }
```

```ts
// Warnings: if you specify an undefined field.
const { article } = await myExecuteQuery({
  article: {
    params: { id: 1 },
    query: {
      id: true,
      titllllle: true,
      comments: {
        user: ['id', 'name'],
        text: true,
        creeeeeeatedAt: true
      }
    }
  }
})
article.id // compile error
// typeof article is { error: { extraFields: 'titllllle' | 'creeeeeeatedAt' } }
```
