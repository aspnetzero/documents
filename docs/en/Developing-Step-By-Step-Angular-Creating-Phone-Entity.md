# Creating Phone Entity

Let's start by creating a new Entity, **Phone** in **.Core** project:

```csharp
[Table("PbPhones")]
public class Phone : CreationAuditedEntity<long>
{
    public const int MaxNumberLength = 16;

    [ForeignKey("PersonId")]
    public virtual Person Person { get; set; }
    public virtual int PersonId { get; set; }

    [Required]
    public virtual PhoneType Type { get; set; }

    [Required]
    [MaxLength(MaxNumberLength)]
    public virtual string Number { get; set; }
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

We have a **PhoneType** enum as shown below: (in **.Core**
project)

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

- [Database Migration of Phone Entity](Developing-Step-By-Step-Angular-Migrations-Phone-Entity)