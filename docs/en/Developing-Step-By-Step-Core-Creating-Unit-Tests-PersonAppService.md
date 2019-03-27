# Creating Unit Tests For PersonAppService

You can skip this section if you don't interest in **automated
testing**.

By writing unit test, we can test **PersonAppService.GetPeople** method
without creating a user interface that calls it and shows people on the
screen.

We write unit test in .**Tests** project in the solution. Let's create
first test to verify getting people without any filter:

```csharp
public class PersonAppService_Tests : AppTestBase
{
    private readonly IPersonAppService _personAppService;

    public PersonAppService_Tests()
    {
        _personAppService = Resolve<IPersonAppService>();
    }

    [Fact]
    public void Should_Get_All_People_Without_Any_Filter()
    {
        //Act
        var persons = _personAppService.GetPeople(new GetPeopleInput());

        //Assert
        persons.Items.Count.ShouldBe(2);
    }
}
```

We derived test class from **AppTestBase**. AppTestBase class
initializes all system, creates an in-memory fake database, seeds
initial data (that we created before) to database and logins to
application as admin. So, this is actually an **integration test** since
it tests all server-side codes from entity framework mapping to
application services, validation and authorization.

In constructor, we get (resolve) an **IPersonAppService** from
**dependency injection** container. It creates the **PersonAppService**
class with all dependencies. Then we can use it in test methods.

Since we're using [xUnit](http://xunit.github.io/), we add **Fact**
attribute to each test method. In the test method, we called
**GetPeople** method and checked if there are **two people** in the
returned list as we know that there were 2 people in **initial**
database.

Let's run the **all unit tests** in Test Explorer and see if it works:

<img src="images/phone-book-unit-test-success-1.png" alt="xUnit unit test success" class="img-thumbnail" width="750" height="40" />

As you see, it worked **successfully**. Now, we know that
PersonAppService works properly without any filter. Let's add a new unit
test to get filtered people:

```csharp
[Fact]
public void Should_Get_People_With_Filter()
{
    //Act
    var persons = _personAppService.GetPeople(
        new GetPeopleInput
        {
            Filter = "adams"
        });

    //Assert
    persons.Items.Count.ShouldBe(1);
    persons.Items[0].Name.ShouldBe("Douglas");
    persons.Items[0].Surname.ShouldBe("Adams");
}
```

Again, since we know initial database, we can check returned results
easily. Here, initial test data is important. When we change initial
data, our test may fail even if our services are correct. So, it's
better to write unit tests independent of initial data as much as
possible. We could check incoming data to see if every people contains
"adams" in his/her name, surname or email. Thus, if we add new people to
initial data, our tests remain working.

There are many techniques on unit testing, I kept it simple here. But
ASP.NET Zero template makes very easy to write unit and integration
tests by base classes and pre-build test codes.


## Next

- [Testing PersonAppService from Browser Console](Developing-Step-By-Step-Core-Testing-PersonAppService-From-Browser-Console.md)				 
