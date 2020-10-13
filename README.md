# graphql-codegen-plugin-typescript-swr

A [GraphQL code generator](https://graphql-code-generator.com/) plug-in that automatically generates utility functions for [SWR](https://swr.vercel.app/).

# Example

```yaml
# codegen.yml
# Add `plugin-typescript-swr` below `typescript-graphql-request`
generates:
  path/to/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
      - typescript-graphql-request
      - plugin-typescript-swr
config:
  rawRequest: false
```

```typescript
// sdk.ts
import { GraphQLClient } from 'graphql-request'
import { getSdkWithHooks } from './graphql'

const sdk = getSdkWithHooks(
  // Disable the cache because it is managed by `swr`.
  // In the case of SSG, the server side always fetches the latest data, so it is unnecessary.
  new GraphQLClient(`${process.env.API_URL}/graphql`, { cache: 'no-cache' })
)

export default sdk
```

```tsx
// Article.tsx
import sdk from '../sdk'

type StaticPropsParams = { slug: string }
export const getStaticProps: GetStaticProps<StaticProps, StaticPropsParams> = async ({
  params,
  preview = false,
}) => {
  if (!params) {
    throw new Error('Parameter is invalid!')
  }

  const { articleBySlug: article } = await sdk.GetArticle({
    slug: params.slug,
  })

  if (!article) {
    throw new Error('Article is not found!')
  }

  return {
    props: {
      slug: params.slug,
      preview,
      initialData: {
        articleBySlug: article
      }
    }
  }
})

export const Article = ({ slug, initialData, preview }: ArticleProps): JSX.Element => {
  const { data: { article } } = sdk.useGetArticle(
    'UniqueKeyForTheRequest', { slug }, { initialData }
  )
  // ...
}
```
