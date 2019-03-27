# Unit Testing

ASP.NET Zero startup project contains **unit** and **integration** tests. Tests are developed using following tools:

- [xUnit](http://xunit.github.io/) as as testing framework.
- [SQLite In-Memory](https://docs.microsoft.com/en-us/ef/core/miscellaneous/testing/sqlite) database for mocking entity framework and database.
- [Abp.TestBase](http://www.nuget.org/packages/Abp.TestBase) to simplify integration testing for ABP based applications.
- [Shouldly](https://github.com/shouldly/shouldly) as assertion library.

Tests cover **Domain** (core) and **Application** layers of the project. Open Test Explorer (Test\\Windows\\Test Explorer in VS main menu) to run unit tests:

Some unit tests (tenant creation, edition creation etc.) are only valid in multi tenancy concept. You can change AbpZeroTemplateConsts.MultiTenancyEnabled to false in order to make your application single tenant. Thus, multitenancy related tests will be skipped.

<img src="images/unit-tests.png" alt="Some unit tests" class="img-thumbnail" width="555" height="244" />

These unit tests will be a guide to understand the code. Also, they can be a model while writing your own unit tests for your application's functionalities.

All unit test classes (actually they are integration tests since they work integrated to ABP, EntityFramework, AutoMapper and other libraries used up to application layer) are derived from **AppTestBase**. It initializes ABP system, mocks database using Effort, creates initial test data and logins to the application for each tests. It also provides some useful common methods for all tests.

Here, a sample unit test in the application:

```
public class UserAppService_Delete_Tests : UserAppServiceTestBase
{
    [Fact]
    public async Task Should_Delete_User()
    {
        //Arrange
        CreateTestUsers();

        var user = await GetUserByUserNameOrNullAsync("artdent");
        user.ShouldNotBe(null);

        //Act
        await UserAppService.DeleteUser(new IdInput<long>(user.Id));

        //Assert
        user = await GetUserByUserNameOrNullAsync("artdent");
        user.IsDeleted.ShouldBe(true);
    }
}
```

It creates some users to test and then verifies there is a user named "artdent". Then it calls **DeleteUser** method of the **user application service** (which is being tested). Finally, checks if user is deleted. Here, User is a Soft Delete entity, so it's **IsDeleted** property must be true if it's deleted. 

You can read [this article](http://www.codeproject.com/Articles/871786/Unit-testing-in-Csharp-using-xUnit-Entity-Framewor) to understand unit testing better.