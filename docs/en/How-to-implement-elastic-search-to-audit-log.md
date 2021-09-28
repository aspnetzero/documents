# How to Implement Elastic Search To Audit Log

In these article we will log all audit log data to Elastic Search.

Aspnetboilerplate uses [IAuditingStore](https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/src/Abp/Auditing/IAuditingStore.cs) to store any audit log data. See: [AuditingHelper.cs](https://github.com/aspnetboilerplate/aspnetboilerplate/blob/dev/src/Abp/Auditing/AuditingHelper.cs#L130-L146). To implement elastic search we should create a service that implements `IAuditingStore`, then replace it with current implementation.

We will use [NEST](https://github.com/elastic/elasticsearch-net#nest) the offical elasticsearch .net library. Install NEST package

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
                            .Text(x => x.Name(d => d.CustomData))
                }));
        }
    }
}
```

Then you should add following code to your module's `PreInitialize` method.

```csharp
public override void PreInitialize()
{
    ElasticSearchAuditingStore.CreateIndexIfNeededAsync();
    
    Configuration.ReplaceService<IAuditingStore, ElasticSearchAuditingStore>(DependencyLifeStyle.Transient);
}
```

After that audit logs also will be logged to elastic search.

