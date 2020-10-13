# Database Migrations for Person

We use **Entity Framework Code-First migrations** to migrate database
schema. Since we added **Person entity**, our DbContext model is
changed. So, we should create a **new migration** to create the new
table in the database.

Open **Package Manager Console**, run the **Add-Migration
"Added\_Persons\_Table"** command as shown below:

<img src="images/phonebook-migrations-core-3.png" alt="Entity Framework Code First Migration" class="img-thumbnail" />

This command will add a **migration class** named
"**Added\_Persons\_Table**" as shown below:

```csharp
public partial class Added_Persons_Table : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "PbPersons",
            columns: table => new
            {
                Id = table.Column(nullable: false)
                    .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
                CreationTime = table.Column(nullable: false),
                CreatorUserId = table.Column(nullable: true),
                DeleterUserId = table.Column(nullable: true),
                DeletionTime = table.Column(nullable: true),
                EmailAddress = table.Column(maxLength: 255, nullable: true),
                IsDeleted = table.Column(nullable: false),
                LastModificationTime = table.Column(nullable: true),
                LastModifierUserId = table.Column(nullable: true),
                Name = table.Column(maxLength: 32, nullable: false),
                Surname = table.Column(maxLength: 32, nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_PbPersons", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "PbPersons");
    }
}
```

We don't have to know so much about format and rules of this file. But,
it's suggested to have a basic understanding of migrations. In the same
Package Manager Console, we write **Update-Database** command in order
to apply the new migration to database. After updating, we can see that
**PbPersons table** is added to database.

<img src="images/phonebook-tables-mpa.png" alt="Phonebook tables" class="img-thumbnail" width="192" height="370" />

But this new table is empty. In ASP.NET Zero, there are some classes to
fill initial data for users and settings:

<img src="images/aspnet-core-ef-seed-1.png" alt="Seed folders" class="img-thumbnail" />

So, we can add a separated class to fill some people to database as
shown below:

```csharp
namespace Acme.PhoneBookDemo.Migrations.Seed.Host
{
    public class InitialPeopleCreator
    {
        private readonly PhoneBookDemoDbContext _context;

        public InitialPeopleCreator(PhoneBookDemoDbContext context)
        {
            _context = context;
        }

        public void Create()
        {
            var douglas = _context.Persons.FirstOrDefault(p => p.EmailAddress == "douglas.adams@fortytwo.com");
            if (douglas == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Douglas",
                        Surname = "Adams",
                        EmailAddress = "douglas.adams@fortytwo.com"
                    });
            }

            var asimov = _context.Persons.FirstOrDefault(p => p.EmailAddress == "isaac.asimov@foundation.org");
            if (asimov == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Isaac",
                        Surname = "Asimov",
                        EmailAddress = "isaac.asimov@foundation.org"
                    });
            }
        }
    }
}
```

These type of default data is good since we can also use these data in
**unit tests**. Surely, we should be careful about seed data since this
code will always be executed in each **PostInitialize** of your
PhoneBookEntityFrameworkCoreModule. This class (InitialPeopleCreator) is
created and called in **InitialHostDbBuilder** class. This is not so
important, just for a good code organization (see source codes).

```csharp
public class InitialHostDbBuilder
{
    //existing codes...

    public void Create()
    {
        //existing codes...
        new InitialPeopleCreator(_context).Create();

        _context.SaveChanges();
    }
}
```

We run our project again, it runs seed and adds two people to PbPersons
table:

<img src="images/phonebook-persons-table-initial-data.png" alt="Persons initial data" class="img-thumbnail" width="720" height="50" />

## Next

- [Creating Person Application Service](Developing-Step-By-Step-Core-DevExtreme-Creating-Person-Application-Service.md)				
