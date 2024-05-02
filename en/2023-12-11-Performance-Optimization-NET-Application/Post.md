# Performance Optimization of .NET-based application

## Introduction

In this article, we will look at how to improve performance of .NET-based application and therefore also abp framework-based application.

## Performance Optimization
Application overall performance is determined by several factors such as:

* Execution times of database queries
* Execution times of application backend code
* Amount of data that has to be sent to the application frontend and/or inside (micro)services
* Overall speed and occupancy of hardware, which provides the runtime for an application
* Network connectivity speed and responsiveness (latency) on end devices (in case of web application or application which consumes REST or other type of API)

In abp framework, all .NET code performance techniques can be used combined with abp framework-specific features.

## Database

Keep database version and potential extensions (plugins) updated as much as possible.

However, most performance gains can be achieved with the techniques described below.

**Also, keep in mind that the performance of production databases heavily depends on regular maintenance (e.g.: defragmentation of indexes, statistics update,...).**

**Additional performance gains on database-level can be achieved with multiple replicas of the same database (synchronous or asynchronous replication - decision should be made based on client's requirements). Read-only database requests can be redirected to non-primary instances. These topics are not covered in this part of the documentation, because of dependence on specific database technology and also are not for software developers (but for DB admins). DB admins can establish multiple replicas environment and all described database optimization techniques below should work without any changes (with additional application performance gains).**

### Caching
To improve performance on the database layer (regardless of database type), it's recommended to use caching wherever possible and safe and as long as possible.

Leverage:

* [Entity Cache](https://docs.abp.io/en/abp/latest/Entity-Cache)
* [Redis Cache](https://docs.abp.io/en/abp/latest/Redis-Cache)

### Denormalization techniques and JSON (RDBMS)
Using [denormalization](https://en.wikipedia.org/wiki/Denormalization) techniques can (significantly) reduce querying execution times.
Querying such pieces of data from different parts of application should be read from the location where the query is the most efficient (performance-wise).

**Writing the same piece of data multiple times inside different SQL tables or even databases consumes more disk space and requires careful domain logic to insert and update the same data across all locations inside a database (databases) as a transactional unit of work.**

Use denormalization techniques in conjunction with JSON capabilities (No-SQL capabilities) of modern RDBMS, which can extend denormalization even further.

Further reading:
* [JSON data in Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server)
* [JSON in PostgreSQL](https://www.postgresql.org/docs/current/functions-json.html)
* [JSON Mapping in Npgsql (EF Core)](https://www.npgsql.org/efcore/mapping/json.html)

### Paging Approach
If **Cursor Pagination** can be used, then it is the preferred way how to effectively sort and paginate data from database. [This article](https://khalidabuhakmeh.com/cursor-paging-with-entity-framework-core-and-aspnet-core) shows execution times and it's based on EF Core implementation.

### Database sharding
The smaller the database, the less data is for querying and this also means lower execution times.
* In multitenant applications, it is recommended to use per-tenant database or multiple databases per tenant (logging db, data db,...).
Each tenant database can be hosted inside different hardware performance tier (cost-effectiveness).
* In a microservice environment, it's recommended to use database-per-microservice or even database-per-tenant-per-microservice
It is possible to achieve described sharding in abp framework:<br>
[Connection Strings](https://docs.abp.io/en/abp/latest/Connection-Strings)<br>
[Multitenancy](https://docs.abp.io/en/abp/latest/Multi-Tenancy)<br>
[Entity Framework Core Migrations](https://docs.abp.io/en/abp/latest/Entity-Framework-Core-Migrations) (Look [Using Multiple Databases](https://docs.abp.io/en/abp/latest/Entity-Framework-Core-Migrations#using-multiple-databases) section for migrations)

### Database partitioning (RDBMS)
Most modern RDBMS offers horizontal (and also vertical) [partitioning](https://en.wikipedia.org/wiki/Partition_(database)).
When querying performance is not good enough because of large tables, it's recommended to create horizontal partitions.
* [Partitions in Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes)
* [Partitions in PostgreSQL](https://www.postgresql.org/docs/current/ddl-partitioning.html)

### Database indexing, partitioning, denormalization, and sharding techniques
**Database indexing, partitioning, denormalization, and sharding can be used in conjunction.**

### Entity Framework Core
There are some general recommendations for writing efficient queries using EF Core, which are [described here](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying).

When maximum performance is needed (complex queries, very frequent queries,...), using [Compiled queries](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ef/language-reference/compiled-queries-linq-to-entities) and/or [SQL queries](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries) can improve performance.

Use ReadOnly Repositories and NoTracking (EF Core feature) for read DB requests (abp v8.0 onwards).

For complex queries when LINQ is not suitable you can use [Dapper](https://docs.abp.io/en/abp/latest/Dapper) (but prefer EF Core SQL queries instead of Dapper, because of similar approaches - keep technology stack and libraries as small as possible).

## Application layer (backend)
* [Microsoft ASP.NET Core Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices) covers most topics regarding performance.
Best Practices is part of [Microsoft ASP.NET Core Performance](https://learn.microsoft.com/en-us/aspnet/core/performance/overview) documentation.

* Usege of [Cached Service Providers](https://docs.abp.io/en/abp/latest/Dependency-Injection#cached-service-providers) (ICachedServiceProvider or ITransientCachedServiceProvider) to optimize dependency injection.

* Using [asynchronous code](https://medium.com/@frederikbanke/improving-scalability-in-c-using-async-and-await-f97af1466922) (**async/await**) wherever possible and not mixing synchronous and asynchronous code

* For computationally intensive tasks, which can leverage parallel code use [Task Parallel Library](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl) (TPL) from Microsoft (available from .NET Framework 4.0 onwards).

* [Reduce memory allocations](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/) wherever possible.

* [Discover StringBuilder performance benefits](https://learn.microsoft.com/en-us/dotnet/api/system.text.stringbuilder)
[This article](https://www.meziantou.net/stringbuilder-performance-pitfalls.htm) shows performance benefits using StringBuilder.

* Build deployment packages in **Release** mode (and with additional optimization possibilities (AoT,...)).

* Updated to the latest stable version of .NET (and also abp framework) with latest runtime optimizations. Updating .NET is usually not a difficult task.

## Frontend
### ASP.NET Core MVC (Razor Pages)
#### Faster page loading
* Using [Bundling & Minifications](https://docs.abp.io/en/abp/latest/UI/AspNetCore/Bundling-Minification) feature of abp framework wherever possible
* Using [Static JavaScript Client Proxies](https://docs.abp.io/en/abp/latest/UI/AspNetCore/Static-JavaScript-Proxies)
* Avoid longer loading times with blank pages with minimal server-side rendering and transfer loading on javascript level (ajax) with loading spinners for specific GUI elements which require more time to get data from backend

#### Faster Javascript execution
Prefer **NOT using jQuery** (or jQuery-based libraries) in favor of pure modern Javascript code ([ECMAScript version](https://en.wikipedia.org/wiki/ECMAScript_version_history)) if possible (based on application complexity, development time,...).

Based on potential client's browser limitations (older browsers), check with [Can I Use](https://caniuse.com/) for javascript features availability.

*abp framework (Commercial) with Razor Pages frontend option integrates jQuery by default, however, custom application code may not need jQuery.*

<hr />

DevExpress also offers non-jQuery version of DevExtreme javascript suite for years now with [noticeable performance gains](https://community.devexpress.com/blogs/donw/archive/2017/11/09/devextreme-with-and-without-jquery-v17-2.aspx).

Keep in mind, that NuGet wrappers (for ASP.NET Core) for [DevExpress requires jQuery](https://docs.devexpress.com/AspNetCore/401026/devextreme-based-controls/get-started/configure-a-visual-studio-project) under the hood. Avoid using NuGet wrappers in favor of plain javascript/typescript files included in .cshtml.


## Network
This section applies to communication between frontend (MVC, Angular,...) and backend and also backend to databases and/or external API services.

* Reduce the number of calls via the network as much as possible (e.g. usage of API methods that return an array of entities,...)
* Use modern compression algorithms ([Brotli](https://en.wikipedia.org/wiki/Brotli),...) wherever possible
* Make calls in parallel if possible (usage of [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) or newer protocol for frontend-backed communication and parallel calls from application backend to external services)
* Prefer lighter data formats (e.g. JSON over XML, [Protobuf](https://en.wikipedia.org/wiki/Protocol_Buffers) over JSON) for fewer data size transfers and/or faster serialization/deserialization
* Prefer vector-based data formats for graphics elements instead of raster graphics wherever possible

**Network engineers can establish specific caching on network-level to further speed-up application (reduce the number of requests that come to the application backend - API). However, this type of optimization should be applied carefully, because application can start behave unpredictable (wrong responses) even the backend code is written correct.**

## Optimizing for Production
abp framework documentation about [Optimizing Your Application for Production Environments](https://docs.abp.io/en/abp/latest/Deployment/Optimizing-Production)
