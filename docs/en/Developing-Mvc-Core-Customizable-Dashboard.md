



# Customizable Dashboard

You can create new widgets and widget filters for the customizable dashboards.

Let's create a new widget and widget filter step by step.

Pre-Note: Customizable dashboard configurations are stored in two places.

- Definitions which include permission, multitenancy side, etc. which should be controlled by the server are located in `*.Core -> DashboardCustomization -> Definitions -> DashboardConfiguration.cs` so that application layer can handle permission and other stuff.
- View side definition like CSS JS file located in the UI project. 
- UI applications get data from the server about the dashboard and use their view information to show it.

## Creating a New Widget Filter

Our widget filter name will be `FilterHelloWorld` . It will have one input and button and it will trigger an event when that input changed.

#### Step 1. Create Filter View 

* Go to `*.Web.Mvc  -> Areas -> [YouAppAreaName] -> Views -> Shared -> CustomizableDashboard -> Widgets` and create a partial view named `FilterHelloWorld` and change it as seen below.

*FilterHelloWorld.cshtml*

```html
<div class="form-group filter-hello-world-container">
    <div class="input-group">
        <input type="text" class="form-control" name="input-filter-hello" placeholder="@L("SearchWithThreeDot")">
        <div class="input-group-append">
            <button class="btn btn-primary" name="btn-filter-hello" type="button">Go!</button>
        </div>
    </div>
</div>
```

* Create folder named FilterHelloWorld into `*.Web.Mvc  -> wwwroot -> view-resources -> Areas -> [YourAppAreaName] -> Views -> CustomizableDashboard -> Widgets`  and create `FilterHelloWorld.css` and `FilterHelloWorld.js` file for into that folder.

* Open `FilterHelloWorld.js` and change it as seen below.

  ```javascript
  $(function () {
      var _$container = $(".filter-hello-world-container");
      var _$btn= _$container.find("button[name='btn-filter-hello']");
      var _$input= _$container.find("input[name='input-filter-hello']"); 
  
      _$btn.click(function () {
           abp.event.trigger('app.dashboardFilters.helloFilter.onNameChange', _$input.val());
      });
  });
  ```



#### Step 2. Define Widget Filter

##### View Definitions

Widget's/widget filter's view consts are located in `*.Core.Shared -> [YourAppName]DashboardCustomizationConsts.cs`. Open `[YourAppName]DashboardCustomizationConsts.cs` create new id for hello world filter. (This id is also used in view page so be careful when selecting this value. It should not start with a number or special characters, etc.)

```csharp
public class [YourAppName]DashboardCustomizationConsts
{
    ...
    public class Filters
    {
      ...
         public const string HelloWorldFilter = "Filters_HelloWorld";
      ...
```



Go to `*.Web.Core -> DashboardCustomization -> DashboardViewConfiguration.cs`. Add your new widget filter's view definition.

```csharp
public class DashboardViewConfiguration
{
    ...
    WidgetFilterViewDefinitions.Add(
        AbpZeroTemplateDashboardCustomizationConsts.Filters.HelloWorldFilter,
        new WidgetFilterViewDefinition(
            AbpZeroTemplateDashboardCustomizationConsts.Filters.HelloWorldFilter,
            viewFileRoot + "FilterHelloWorld.cshtml",
            jsAndCssFileRoot + "FilterHelloWorld/FilterHelloWorld.min.js",
            jsAndCssFileRoot + "FilterHelloWorld/FilterHelloWorld.min.css")
  	);
	...
}
```



##### Main Definition

Go to `*.Core -> DashboardCustomization -> Definitions -> DashboardConfiguration.cs` and add hello world filter definition.

```csharp
public class DashboardConfiguration
  {
    public DashboardConfiguration()
      {
        ...
var helloWorldFilter = new WidgetFilterDefinition(
            AbpZeroTemplateDashboardCustomizationConsts.Filters.HelloWorldFilter, "FilterHelloWorld");
        
WidgetFilterDefinitions.Add(helloWorldFilter);
        ...
```

Now your filter is available for all widgets. You can use it in any widget definition and it will be loaded to page automatically.



## Creating a New Widget

Our widget name will be `WidgetHelloWorld`

#### Step 1. Create an API

Create an API which your widgets needs. In this scenario, I will create one endpoint into `TenantDashboardAppService.cs` named `GetHelloWorldData`.

```csharp
public class GetHelloWorldInput
{
    public string Name { get; set; }
}

public class GetHelloWorldOutput
{
    public string OutPutName { get; set; }
}

public interface ITenantDashboardAppService : IApplicationService
{
  ...
  GetHelloWorldOutput GetHelloWorldData(GetHelloWorldInput input);
  ...
}
public class TenantDashboardAppService ...
{
    ...
    [AbpAuthorize(AppPermissions.HelloWorldPermission)]//check permission
    public GetHelloWorldOutput GetHelloWorldData(GetHelloWorldInput input)
    {
        return new GetHelloWorldOutput()
        {
             OutPutName = "Hello " + input.Name + " (" + Clock.Now.Millisecond + ")"
        };
    }
    ...
}
```

*Although ASP.NET Zero load widgets by filtering permission and other things. We still have to check permission here.*

#### Step 2. Create Widget View

* Go to `*.Web.Mvc  -> Areas -> [YouAppAreaName] -> Views -> Shared -> CustomizableDashboard -> Widgets` and create a partial view named `WidgetHelloWorld` and change it as seen below.

*WidgetHelloWorld.cshtml*

```html
<div class="kt-portlet kt-portlet--height-fluid HelloWorldContainer">
    <div class="kt-portlet__head">
        <div class="kt-portlet__head-label">
            <h3 class="kt-portlet__head-title">
                Hello World
            </h3>
        </div>
    </div>
    <div class="kt-portlet__body">
        Hello World Works! <br/>
        Response: <span class="hello-response">NULL</span>
    </div>
</div>
```

* Create a folder named HelloWorld into `*.Web.Mvc  -> wwwroot -> view-resources -> Areas -> [YourAppAreaName] -> Views -> CustomizableDashboard -> Widgets`  and create `HelloWorld.css` and `HelloWorld.js` file for into that folder.

![customizable-dashboard-mvc-css-js](images/customizable-dashboard-mvc-css-js.png)

* Open `HelloWorld.js` and change it as seen below.

```javascript
$(function () {
    var _tenantDashboardService = abp.services.app.tenantDashboard;
	var _widgetBase = app.widgetBase.create();
    var _$Container = $('.HelloWorldContainer');

    var getHelloWorld = function (name) {
        abp.ui.setBusy(_$Container);

        _tenantDashboardService
            .getHelloWorldData({name:name})
            .done(function (result) {
                 _$Container.find(".hello-response").text(result.outPutName);
            }).always(function () {
                abp.ui.clearBusy(_$Container);
            });
    };
    
     _widgetBase.runDelayed(function(){
          getHelloWorld("First Attempt");
     });
    
	 //event which your filter send
    abp.event.on('app.dashboardFilters.helloFilter.onNameChange', function (name) {
        _widgetBase.runDelayed(function(){
          getHelloWorld(name);
     	});
    });
});
```



#### Step 3. Define Widget 

##### View Definitions

Widget's/widget filter's view consts are located in `*.Core.Shared -> [YourAppName]DashboardCustomizationConsts.cs ` . Open `*DashboardCustomizationConsts.cs` create new id for hello world widget. (This id is also used in view page so be careful when selecting this value. It should not start with a number or special characters, etc.)

```csharp
public class [YourAppName]DashboardCustomizationConsts
{
    
    public class Widgets
    {
        public class Tenant
        {
            public const string HelloWorld = "Widgets_Tenant_HelloWorld";

      ...
```

Go to `*.Web.Core -> DashboardCustomization -> DashboardViewConfiguration.cs`. Add your hello world widget's view definition.

```csharp
public class DashboardViewConfiguration
{
    ...
		WidgetViewDefinitions.Add(
            AbpZeroTemplateDashboardCustomizationConsts.Widgets.Tenant.HelloWorld,
            new WidgetViewDefinition(
                AbpZeroTemplateDashboardCustomizationConsts.Widgets.Tenant.HelloWorld,
                viewFileRoot + "WidgetHelloWorld.cshtml",
                jsAndCssFileRoot + "HelloWorld/HelloWorld.min.js",
                jsAndCssFileRoot + "HelloWorld/HelloWorld.min.css",
                defaultWidth:6,
                defaultHeight:4)
        );
	...
}
```

##### Main Definition

Go to `*.Core -> DashboardCustomization -> Definitions -> DashboardConfiguration.cs` and add hello world widget's definition.

```csharp
public class DashboardConfiguration
  {
    public DashboardConfiguration()
      {
        ...
var helloWorld = new WidgetDefinition(
    id:AbpZeroTemplateDashboardCustomizationConsts.Widgets.Tenant.HelloWorld,
    name:"WidgetRecentTenants",//localized string key
    side: MultiTenancySides.Tenant,
    usedWidgetFilters: new List<string>() { helloWorldFilter.Id },// you can use any filter you need
    permissions: tenantWidgetsDefaultPermission);
        
helloWorld.Permissions.Add(AppPermissions.HelloWorldPermission);
        ...
        
        ...
var defaultTenantDashboard = new DashboardDefinition(
    AbpZeroTemplateDashboardCustomizationConsts.DashboardNames.DefaultTenantDashboard,
    new List<string>()
    {
        generalStats.Id, dailySales.Id, profitShare.Id, memberActivity.Id, regionalStats.Id, topStats.Id, salesSummary.Id, helloWorld.Id //add your widget to dashboard
    });
        ...
```



After all of these, open any terminal and run `npm run create-bundles` command in `*.Web.Mvc` project folder. This will create all minified CSS and JS files we defined.

After that you will be able to use your new widget. 

## Usage

Since we create tenant side widget, open tenant dashboard. 

Select the page you want to use hello widget.

Add hello widget to the page as described in that article: [Customizable Dashboard Usage](Features-Angular-Customizable-Dashboard.md) 

After that, you will see that your widget is located on the page and works as expected.

![customizable-dashboard-widget-hello-world](images/customizable-dashboard-widget-hello-world.png)



Since hello world widget needs hello world filter *(we defined it in DashboardConfiguration)* hello world filter will be loaded to the page. To use it. Click the filter button next to the **Edit Mode** button. It will open the filter modal. 

As you can below, you will be able to see filters that your widgets need. Change input and click **Go**. Hello world widget will be changed by your filter.

![customizable-dashboard-filter-hello-world](images/customizable-dashboard-filter-hello-world-2.png)