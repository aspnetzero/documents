# Publishing to Docker Containers

ASP.NET Zero solution has a **build folder** which contains a PowerShell script to **build & publish** your solution to the **output** folder. 

In order to build Docker images for your project, run ```build-mvc.ps1```. This script will generate 2 images, one is for the MVC web project and other one is for the Public web project. After creating the images , you can go to ```build/outputs/Mvc``` or  ```build/outputs/Public```  folders and run related images by running the command below;

```
docker compose -f docker-compose.yml -f docker-compose.override.yml up
```

By default, these images will try to connect to a database on server 192.168.1.37,1433. Change this IP address to IP address of the database you want to connect before building images. 

These images will use the same ports when you run your apps in Visual Studio, so you can also change those ports for production. 


