# Cursor-based Pagination using GraphQL and Prisma

In this tutorial, you'll learn about types of pagination, and how to implement cursor-based pagination using the Prisma framework and GraphQL Nexus. We'll be using SQLite for our database layer, and Apollo Server as our server – some familiarity with these tools is recommended in order to get the most out of the tutorial.

**Tools used in this tutorial**
* [SQLite](https://sqlite.org)
* [Prisma Framework](https://www.prisma.io)
* [GraphQL Nexus](https://nexusjs.org) (and [`nexus-prisma`](https://nexus.prisma.io))
* [Apollo Server](https://www.apollographql.com)

## What is Pagination?
When dealing with large collections of information, frequently this data needs to be broken into smaller sets, or _paginated_, in order to be processed effectively. There are two primary approaches to pagination:

**Limit/offset pagination**
* The data is handled as a list of entries.

  **Limit** refers to the maximum number of entries that should be sent as a result of a query. (This is also known as the _page size_.) 

   **Offset** refers to the number of entries the results should skip before being collected, saying where in the data collection the results should start.

* By specifying how many results should be returned, and how far into a set of results should start, you can effectively _page_ through a large set of data in more manageable chunks.

**Cursor-based pagination**
* The data is handled more like a dictionary of entries, or a linked list. A cursor generally uses a unique string associated with a specific data entry, which can be used to identify where you are in a collection.

* By specifying a starting entry and either an ending entry or a entry limit (or both), you return all entries that fit those criteria.

* While you can use cursor-based pagination without a unique identifier, doing so incurs a performance penalty, as the pagination query then needs to sort against a second field to try and identify where to stop and start the query.

Both approaches to pagination have benefits and drawbacks: notably, limit/offset pagination is linear and does not handle changes to a data set well (if entries are added or removed while pagination is happening, your results may end up with entries either omitted or added twice); meanwhile, cursor-based pagination needs something unique to sort against, and can become slow if it needs to sort by multiple properties.

## Setting up your sample project

Prisma provides a sample project that can be used to get started with this tutorial. This can be downloaded by cloning this repo:
```
git clone https://github.com/Nadreck/prisma-exercise.git
```
Once the repository has been cloned, run:
```
cd prisma-exercise
npm install
npx prisma migrate dev --name init 
npx prisma db seed 
```
This will create an sqlite database named `dev.db`, and seed it with some sample users and posts for us to query.

When ready, run:
```
npm run dev
```
Then navigate to `http://localhost:4000` to proceed.

## Querying data with pagination

The sample project is configured to support both limit/offset and cursor-based queries when using the `feed` query. In the case of cursor-based pagination, the feed is configured to use the `id` field as the basis for our cursor (taken from `src/schema.js`):
```js
t.nonNull.list.nonNull.field('feed', {
  type: 'Post',
  args: {
    searchString: stringArg(),
    skip: intArg(),
    take: intArg(),
    orderBy: arg({
      type: 'PostOrderByUpdatedAtInput',
    }),
    cursor: intArg(),
  },
  resolve: (_parent, args, context) => {
    const or = args.searchString
      ? {
          OR: [
            { title: { contains: args.searchString } },
            { content: { contains: args.searchString } },
          ],
        }
      : {}

    if (args.cursor) {
      const searchResults = context.prisma.post.findMany({
        cursor: {
          id: args.cursor || undefined,
        },
        take: args.take || undefined,
        orderBy: args.orderBy || undefined,
      })
      return searchResults
    } else {
      const searchResults = context.prisma.post.findMany({
        where: {
          published: true,
          ...or,
        },
        take: args.take || undefined,
        skip: args.skip || undefined,
        orderBy: args.orderBy || undefined,
      })
      return searchResults
    }
    
  },
})
```

To use cursor-based pagination, for example, the following query:
```js
query Feed($cursor: Int) {
  feed(cursor: 2) {
    id
    title
  }
}
```

Results in the following output:
```json
{
  "data": {
    "feed": [
      {
        "id": 2,
        "title": "Follow Prisma on Twitter"
      },
      {
        "id": 3,
        "title": "Ask a question about Prisma on GitHub"
      },
      {
        "id": 4,
        "title": "Prisma on YouTube"
      }
    ]
  }
}
```

This is because the cursor is set to start where `id` equals 2. If you wanted to then limit how many results are returned, you can include a limit, using the `take` argument:
```
query Feed($cursor: Int) {
  feed(take: 1cursor: 2) {
    id
    title
  }
}
```

This would result in returning just one entry:
```json
{
  "data": {
    "feed": [
      {
        "id": 2,
        "title": "Follow Prisma on Twitter"
      }
    ]
  }
}
```

## Conclusion

Cursor-based pagination can be a powerful tool for working with large data sets. For more examples of pagination in practice, check the [Prisma.io Client API](https://www.prisma.io/client) reference.
