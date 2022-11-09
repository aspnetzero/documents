# How to Save Audit Logs to ElasticSearch

ASP.NET Zero provides AuditLogging functionality out of the box and all audit logs are saved to database by default. 

In this article we will log all audit log data to [Elastic Search](https://www.elastic.co/). We assume that, you already have a working  [Elastic Search](https://www.elastic.co/) which you can use for this article. If not, please install it to your PC first.

[ASP.NET Boilerplate](https://aspnetboilerplate.com/) uses [IAuditingStore](https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/src/Abp/Auditing/IAuditingStore.cs) to store any audit log data. See: [AuditingHelper.cs](https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/src/Abp/Auditing/AuditingHelper.cs#L130-L146). To save audit logs to elasticsearch, we should create a service that implements `IAuditingStore`, then replace it with current implementation.

We will use [NEST](https://github.com/elastic/elasticsearch-net#nest) the offical elasticsearch .net library. Install NEST package to your project using the command below or NuGet Package Manager.

```shell
PM> Install-Package NEST
```

Then create new class named `ElasticSearchAuditingStore`

```csharp 
using System;
using System.Threading.Tasks;
using Abp.Auditing;
using Abp.Domain.Repositories;
using Nest;

public class ElasticSearchAuditingStore : AuditingStore
{
    private const string ElasticSearchIndexName = "AUDIT_LOG_INDEX_NAME";

    public ElasticSearchAuditingStore(IRepository<AuditLog, long> auditLogRepository) : base(auditLogRepository)
    {
    }

    public override void Save(AuditInfo auditInfo)
    {
        base.Save(auditInfo);
        var client = GetClient();
        client.Index(auditInfo, x => x.Index(ElasticSearchIndexName));
    }

    public override async Task SaveAsync(AuditInfo auditInfo)
    {
        await base.SaveAsync(auditInfo);
        var client = GetClient();
        await client.IndexAsync(auditInfo, x => x.Index(ElasticSearchIndexName));
    }

    public static IElasticClient GetClient()
    {
        var node = new Uri("[YOUR_ELASTIC_SEARCH_URL]");
        var settings = new ConnectionSettings(node)
            .DefaultIndex(ElasticSearchIndexName)
            .BasicAuthentication("USERNAME", "PASSWORD");
        return new ElasticClient(settings);
    }

    public static void CreateIndexIfNeededAsync()
    {
        var client = GetClient();

        var existsResponse = client.Indices.Exists(ElasticSearchIndexName);

        if (!existsResponse.Exists)
        {
            client.Indices.Create(ElasticSearchIndexName, c => c
                .Map<AuditInfo>(m =>
                {
                    return m
                        .Properties(p => p
                            .Keyword(x => x.Name(d => d.TenantId))
                            .Keyword(x => x.Name(d => d.UserId))
                            .Keyword(x => x.Name(d => d.ImpersonatorUserId))
                            .Keyword(x => x.Name(d => d.ImpersonatorTenantId))
                            .Keyword(x => x.Name(d => d.ServiceName))
                            .Keyword(x => x.Name(d => d.MethodName))
                            .Keyword(x => x.Name(d => d.Parameters))
                            .Keyword(x => x.Name(d => d.ReturnValue))
                            .Keyword(x => x.Name(d => d.ExecutionTime))
                            .Keyword(x => x.Name(d => d.ExecutionDuration))
                            .Keyword(x => x.Name(d => d.ClientIpAddress))
                            .Keyword(x => x.Name(d => d.ClientName))
                            .Keyword(x => x.Name(d => d.BrowserInfo))
                            .Text(x => x.Name(d => d.CustomData)));
                }));
        }
    }
}
```

Now, we need to replace the default AuditingStore implementation with our new ElasticSearchAuditingStore. To do that, add following code to your module's `PreInitialize` method.

```csharp
public override void PreInitialize()
{
    ElasticSearchAuditingStore.CreateIndexIfNeededAsync();
    
    Configuration.ReplaceService<IAuditingStore, ElasticSearchAuditingStore>(DependencyLifeStyle.Transient);
}
```

After all, audit logs will be saved to elastic search. Now, you can make better search operations on your audit logs using ElasticSearch.

