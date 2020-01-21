# Creating Person Entity

We define entities in **.Core** (domain) project. We can define a
**Person** entity (mapped to **PbPersons** table in database) to
represent a person in phone book as shown below:

```csharp
[Table("PbPersons")]
public class Person : FullAuditedEntity
{
    [Required]
    [MaxLength(PersonConsts.MaxNameLength)]
    public virtual string Name { get; set; }

    [Required]
    [MaxLength(PersonConsts.MaxSurnameLength)]
    public virtual string Surname { get; set; }

    [MaxLength(PersonConsts.MaxEmailAddressLength)]
    public virtual string EmailAddress { get; set; }
}
```

Person's **primary key** type is **int** (as default). It inherits
**FullAuditedEntity** that contains **creation**, **modification** and
**deletion** audit properties. It's also **soft-delete**. When we delete
a person, it's not deleted by database but marked as deleted (see
[entity](https://aspnetboilerplate.com/Pages/Documents/Entities) and
[data
filters](https://aspnetboilerplate.com/Pages/Documents/Data-Filters)
documentations for more information). We created **PersonConsts** in
**Core.Shared** project for **MaxLength** properties. This is a good
practice since we will use same values later.

```csharp
public class PersonConsts
{
    public const int MaxNameLength = 32;
    public const int MaxSurnameLength = 32;
    public const int MaxEmailAddressLength = 255;
}
```

We add a DbSet property for Person entity to **PhoneBookDemoDbContext**
class defined in **.EntityFrameworkCore** project.

```csharp
public class PhoneBookDemoDbContext : AbpZeroDbContext<Tenant, Role, User, PhoneBookDemoDbContext>
{
    public virtual DbSet<Person> Persons { get; set; }

    //...other entities

    public PhoneBookDemoDbContext()
        : base("Default")
    {

    }

    //...other codes
}
```

## Next

- [Database Migrations for Person](Developing-Step-By-Step-Core-Database-Migrations-Person.md)
