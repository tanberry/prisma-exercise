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

## Set up Prisma
1. Install the tools we plan to use in this project:
    ```sh
    mkdir graphql-cursors
    cd graphql-cursors
    npm init --yes
    npm install apollo-server nexus graphql prisma nexus-prisma
    ```

2. Set up Prisma:
    ```sh
    npx prisma init
    ```

3. Create your sqlite database:
    ```sql
    sqlite3 sample.db;
    ```

4. Open `prisma/schema.prisma` and update the datasource to talk to your sample database:
    ```js
    datasource db {
      provider = "sqlite"
      url      = "file:./sample.db"
    }
    ```
5. Update `prisma/schema.prisma` to include a sample data model:
    ```
    ```
6. Load some sample data into the database:
    ```
    ```

### Set up Nexus

### Set up Apollo

## Querying data with pagination
