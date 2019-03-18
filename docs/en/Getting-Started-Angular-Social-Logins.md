# Social Logins

Social logins can be enabled and configured from [backend](Development-Guide-Core.md). Once they are properly configured, they are automatically shown in the user interface. 

login/**login.service** implements client side logic for social logins. Note that currently only **Facebook** and **Google** authentication is implemented for Angular2 application. Microsoft and Twitter logins are on the road map.