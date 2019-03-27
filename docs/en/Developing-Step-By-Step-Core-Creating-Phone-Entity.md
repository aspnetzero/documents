# Creating Phone Entity

Let's start by creating a new Entity, **Phone** in **.Core** project:

```csharp
[Table("PbPhones")]
public class Phone : CreationAuditedEntity<long>
{

    [ForeignKey("PersonId")]
    public virtual Person Person { get; set; }
    public virtual int PersonId { get; set; }

    [Required]
    public virtual PhoneType Type { get; set; }

    [Required]
    [MaxLength(PhoneConsts.MaxNumberLength)]
    public virtual string Number { get; set; }
}
```

And **PhoneConsts** in **Core.Shared** project:

```csharp
public class PhoneConsts
{
    public const int MaxNumberLength = 16;
}
```

Phone entities are stored in **PbPhones** table. Its primary key is
**long** and it inherits creation auditing properties. It has a reference
to **Person** entity which owns the phone number.

We added a **Phones** collection to the People:

```csharp
[Table("PbPersons")]
public class Person : FullAuditedEntity
{
    //...other properties

    public virtual ICollection<Phone> Phones { get; set; }
}
```

We have a **PhoneType** enum in **.Core.Shared** project as shown below:

```csharp
public enum PhoneType : byte
{
    Mobile,
    Home,
    Business
}
```

Lastly, we're also adding a DbSet property for Phone to our DbContext:

```csharp
public virtual DbSet<Phone> Phones { get; set; }
```

## Next

- [Database Migration of Phone Entity](Developing-Step-By-Step-Core-Database-Migration-Phone-Entity.md)