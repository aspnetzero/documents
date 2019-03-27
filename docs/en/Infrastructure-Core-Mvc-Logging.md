# Logging

ASP.NET Zero uses **Log4Net** for logging as default. Configuration is defined in **log4net.config** file in the .Web project. It writes all logs to **App\_Data/Logs/Logs.txt** folder of web site as default. When you publish your project, remember to configure **write permission** to Logs folder.

Check [logging documentation](https://aspnetboilerplate.com/Pages/Documents/Logging) to see how to inject `ILogger` and write logs.