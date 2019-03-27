# Database Migration of Phone Entity

Our entity model has changed, so we need to add a new migration. Run
this command in the .EntityFramework project's directory:

<img src="images/phonebook-migrations-core-4.png" alt="Entity Framework Migration" class="img-thumbnail" />

This will create a new code based migration file to create **PbPhones**
table:

```csharp
public partial class Added_Phone : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "PbPhones",
            columns: table => new
            {
                Id = table.Column<long>(nullable: false)
                    .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
                CreationTime = table.Column<DateTime>(nullable: false),
                CreatorUserId = table.Column<long>(nullable: true),
                Number = table.Column<string>(maxLength: 16, nullable: false),
                PersonId = table.Column<int>(nullable: false),
                Type = table.Column<byte>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_PbPhones", x => x.Id);
                table.ForeignKey(
                    name: "FK_PbPhones_PbPersons_PersonId",
                    column: x => x.PersonId,
                    principalTable: "PbPersons",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });

        migrationBuilder.CreateIndex(
            name: "IX_PbPhones_PersonId",
            table: "PbPhones",
            column: "PersonId");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "PbPhones");
    }
}
```

Before updating database, we can go to database
**InitialPeopleCreator**, rename it to **InitialPeopleAndPhoneCreator**
and add example **phone numbers** for example people (We renamed
InitialPeopleCreator.cs to InitialPeopleAndPhoneCreator.cs):

```csharp
public class InitialPeopleAndPhoneCreator
{
    //...

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
                    EmailAddress = "douglas.adams@fortytwo.com",
                    Phones = new List<Phone>
                                {
                                    new Phone {Type = PhoneType.Home, Number = "1112242"},
                                    new Phone {Type = PhoneType.Mobile, Number = "2223342"}
                                }
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
                    EmailAddress = "isaac.asimov@foundation.org",
                    Phones = new List<Phone>
                                {
                                    new Phone {Type = PhoneType.Home, Number = "8889977"}
                                }
                });
        }
    }
}
```

We added two phone numbers to Douglas, one phone number to Isaac. But if
we run our application now, phones are not inserted since this seed code
checks if person exists, and does not insert if it already exists.
Since we haven't deployed yet, we can delete database
(or remove entries from people table) and re-create it.

Now, we are running our application to re-create database and seed it.
You can check database to see **PbPhones** table and rows.

## Next

- [Changing GetPeople Method](Developing-Step-By-Step-Angular-Changing-GetPeople-Method)