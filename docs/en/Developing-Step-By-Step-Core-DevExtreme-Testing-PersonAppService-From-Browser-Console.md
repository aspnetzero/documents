# Testing PersonAppService From Browser Console

Now, lets run and **login** to the application again, open Chrome
Developer Console (or similar tools in other browsers) and write the
following command:

<img src="images/chrome-console-get-people.png" alt="Google Chrome Console AJAX" class="img-thumbnail" width="497" height="104" />

This command performs an **AJAX** call to PersonAppService.**GetPeople**
method. We can see the request if we open **Network** tab:

<img src="images/chrome-console-get-people-network.png" alt="Google Chrome Console Get People" class="img-thumbnail" width="720" height="148" />

As we see, an AJAX request made and people are got successfully.

So, how this happen? How we could make a call to a C\# class method
(notice that it's not an MVC Controller, just a plain C\# class) from
javascript like calling a javascript method? This is provided by ASP.NET
Boilerplate. See [AspNet
Core](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#application-services-as-controllers)
documentation for more information. You can always call application
services from console to debug or see returned JSON structure.

We can also check Audit Logs to see the request. Open **AbpAuditLogs**
table in PhoneBook database to see the call information:

<img src="images/audit-log-table-getpeople.png" alt="Audit Logs table" class="img-thumbnail" width="921" height="71" />

There are some other fields not shown here. So, we see that User with
Id=2 called GetPeople method of the PersonAppService in recorded time
with the shown parameters and it's executed in 134 ms.

## Next

- [Using GetPeople Method From MVC Controller](Developing-Step-By-Step-Core-DevExtreme-Using-GetPeople-Method-From-MVC-Controller.md)

