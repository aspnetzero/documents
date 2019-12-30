# Converting Create/Edit Modal to Page

In this document we will explain how to convert AspNet Zero's tenant create & edit modals to regular mvc pages.

## Create Tenant

### Create Action Result 

Create action result named **Create**. Copy **CreateModal** partial view result into that and change necessary parts as seen below

```csharp
[AbpMvcAuthorize(AppPermissions.Pages_Tenants_Create)]
public async Task<ActionResult> Create()
{
    var editionItems = await _editionAppService.GetEditionComboboxItems();
    var defaultEditionName = _commonLookupAppService.GetDefaultEditionName().Name;
    var defaultEditionItem = editionItems.FirstOrDefault(e => e.DisplayText == defaultEditionName);
    if (defaultEditionItem != null)
    {
        defaultEditionItem.IsSelected = true;
    }

    var viewModel = new CreateTenantViewModel(editionItems)
    {
        PasswordComplexitySetting = await _passwordComplexitySettingStore.GetSettingsAsync()
    };
    return View(viewModel);
}

```

### Create View File

Then create view named **Create** in `Areas/[AppAreaName]/Views/Tenants` folder.

Copy **_CreateModel.cshtml**  into your new **Create.cshtml** .

Remove modal header, footer partialviews and  modal body tag. Add a portlet that surrounds the form tag.

The result will be like seen below:

```html
@using Abp.Json
@using MyCompanyName.AbpZeroTemplate.MultiTenancy
@using MyCompanyName.AbpZeroTemplate.Web.Areas.AppAreaName.Models.Tenants
@model CreateTenantViewModel
@section Scripts
{
    <script src="~/view-resources/Areas/AppAreaName/Views/Tenants/Create.js"></script>
    <script>
        window.passwordComplexitySetting = @Html.Raw(Model.PasswordComplexitySetting.ToJsonString(indented: true));
    </script>
}

<div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
    <div class="kt-subheader kt-grid__item">
        <div class="@(await GetContainerClass())">
            <div class="kt-subheader__main">
                <h3 class="kt-subheader__title">
                    <span>@L("CreateNewTenant")</span>
                </h3>
            </div>
        </div>
    </div>
    <div class="@(await GetContainerClass()) kt-grid__item kt-grid__item--fluid">
        <div class="kt-portlet">
            <div class="kt-portlet__body">
                <form name="TenantInformationsForm">
                    <div class="form-group">
                        <label for="TenancyName">@L("TenancyName")</label>
                        <input id="TenancyName" class="form-control" type="text" name="TenancyName" required maxlength="@Tenant.MaxTenancyNameLength" regex="@Tenant.TenancyNameRegex">
                    </div>

                    <div class="form-group no-hint">
                        <label for="Name">@L("Name")</label>
                        <input id="Name" type="text" name="Name" class="form-control" required maxlength="@Tenant.MaxNameLength">
                    </div>

                    <div class="kt-checkbox-list">
                        <label class="kt-checkbox">
                            <input id="CreateTenant_UseHostDb" type="checkbox" name="UseHostDb" value="true" checked="checked">
                            @L("UseHostDatabase")
                            <span></span>
                        </label>
                    </div>

                    <div class="form-group no-hint" style="display: none">
                        <label for="ConnectionString">@L("DatabaseConnectionString")</label>
                        <input id="ConnectionString" type="text" name="ConnectionString" class="form-control" required maxlength="@Tenant.MaxConnectionStringLength">
                    </div>

                    <div class="form-group">
                        <label for="AdminEmailAddress">@L("AdminEmailAddress")</label>
                        <input id="AdminEmailAddress" type="email" name="AdminEmailAddress" class="form-control" required maxlength="@MyCompanyName.AbpZeroTemplate.Authorization.Users.User.MaxEmailAddressLength">
                    </div>

                    <div class="kt-checkbox-list">
                        <label class="kt-checkbox">
                            <input id="CreateTenant_SetRandomPassword" type="checkbox" name="SetRandomPassword" value="true" checked="checked" />
                            @L("SetRandomPassword")
                            <span></span>
                        </label>
                    </div>

                    <div class="form-group no-hint tenant-admin-password" style="display: none">
                        <label for="CreateTenant_AdminPassword">@L("Password")</label>
                        <input id="CreateTenant_AdminPassword" type="password" name="AdminPassword" class="form-control" maxlength="@MyCompanyName.AbpZeroTemplate.Authorization.Users.User.MaxPlainPasswordLength" autocomplete="new-password">
                    </div>

                    <div class="form-group tenant-admin-password" style="display: none">
                        <label for="AdminPasswordRepeat">@L("PasswordRepeat")</label>
                        <input id="AdminPasswordRepeat" type="password" name="AdminPasswordRepeat" class="form-control" maxlength="@MyCompanyName.AbpZeroTemplate.Authorization.Users.User.MaxPlainPasswordLength" equalto="#CreateTenant_AdminPassword" autocomplete="new-password">
                    </div>

                    <div class="form-group no-hint">
                        <label for="EditionId">@L("Edition")</label>
                        <select class="form-control" id="EditionId" name="EditionId">
                            @foreach (var edition in Model.EditionItems)
                            {
                                <option value="@edition.Value" data-isfree="@edition.IsFree">@edition.DisplayText</option>
                            }
                        </select>
                    </div>

                    <div class="kt-checkbox-list subscription-component">
                        <label for="CreateTenant_IsUnlimited" class="kt-checkbox">
                            <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited" />
                            @L("UnlimitedTimeSubscription")
                            <span></span>
                        </label>
                    </div>

                    <div class="form-group subscription-component">
                        <label for="SubscriptionEndDateUtc">@L("SubscriptionEndDateUtc")</label>
                        <input id="SubscriptionEndDateUtc" type="datetime" name="SubscriptionEndDateUtc" class="form-control date-time-picker" required>
                    </div>

                    <div class="kt-checkbox-list subscription-component">
                        <label for="CreateTenant_IsInTrialPeriod" class="kt-checkbox">
                            <input id="CreateTenant_IsInTrialPeriod" class="md-check" type="checkbox" name="IsInTrialPeriod" value="true" />
                            @L("IsInTrialPeriod")
                            <span></span>
                        </label>
                    </div>

                    <div class="kt-checkbox-list">
                        <label for="CreateTenant_ShouldChangePasswordOnNextLogin" class="kt-checkbox">
                            <input id="CreateTenant_ShouldChangePasswordOnNextLogin" type="checkbox" name="ShouldChangePasswordOnNextLogin" value="true" checked="checked">
                            @L("ShouldChangePasswordOnNextLogin")
                            <span></span>
                        </label>
                        <label for="CreateTenant_SendActivationEmail" class="kt-checkbox">
                            <input id="CreateTenant_SendActivationEmail" type="checkbox" name="SendActivationEmail" value="true" checked="checked">
                            @L("SendActivationEmail")
                            <span></span>
                        </label>
                        <label for="CreateTenant_IsActive" class="kt-checkbox">
                            <input id="CreateTenant_IsActive" type="checkbox" name="IsActive" value="true" checked="checked">
                            @L("Active")
                            <span></span>
                        </label>
                    </div>
                </form>
            </div>
            <div class="kt-portlet__foot">
                <div class="row align-items-center">
                    <div class="col-md-12 kt-align-right pr-5">
                        <a href="/AppAreaName/Tenants" class="kt-link kt-font-bold">Cancel</a>
                        <span class="kt-margin-left-10">or</span>  <button id="btnCreateTenant" class="btn btn-brand">Submit</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

```

### Create Javascript File

Create javascript file named **Create.js**.

Copy **_CreateModal.js** into **Create.js**. 

Search `modal.find` and replace all with `$` . 

Remove modal function and take their content.

The result will be like seen below:



```javascript
(function () {
    $(function () {
        //Custom validation type for tenancy name
        $.validator.addMethod("tenancyNameRegex", function (value, element, regexpr) {
            return regexpr.test(value);
        }, app.localize('TenantName_Regex_Description'));

        var _tenantService = abp.services.app.tenant;
        var _$tenantInformationForm = null;
        var _passwordComplexityHelper = new app.PasswordComplexityHelper();

        _$tenantInformationForm = $('form[name=TenantInformationsForm]');
        _$tenantInformationForm.validate({
            rules: {
                TenancyName: {
                    tenancyNameRegex: new RegExp(_$tenantInformationForm.find('input[name=TenancyName]').attr('regex'))
                }
            }
        });

        //Show/Hide password inputs when "random password" checkbox is changed.
        var passwordInputs = $('input[name=AdminPassword],input[name=AdminPasswordRepeat]');
        var passwordInputGroups = passwordInputs.closest('.form-group');

        _passwordComplexityHelper.setPasswordComplexityRules(passwordInputs, window.passwordComplexitySetting);

        $('#CreateTenant_SetRandomPassword').change(function () {
            if ($(this).is(':checked')) {
                passwordInputGroups.slideUp('fast');
                passwordInputs.removeAttr('required');
            } else {
                passwordInputGroups.slideDown('fast');
                passwordInputs.attr('required', 'required');
            }
        });

        //Show/Hide connection string input when "use host db" checkbox is changed.

        var connStringInput = $('input[name=ConnectionString]');
        var connStringInputGroup = connStringInput.closest('.form-group');

        $('#CreateTenant_UseHostDb').change(function () {
            if ($(this).is(':checked')) {
                connStringInputGroup.slideUp('fast');
                connStringInput.removeAttr('required');
            } else {
                connStringInputGroup.slideDown('fast');
                connStringInput.attr('required', 'required');
            }
        });

        $('.date-time-picker').datetimepicker({
            locale: abp.localization.currentLanguage.name,
            format: 'L'
        });

        var $subscriptionEndDateDiv = $('input[name=SubscriptionEndDateUtc]').parent('div');
        var $isUnlimitedInput = $('#CreateTenant_IsUnlimited');
        var subscriptionEndDateUtcInput = $('input[name=SubscriptionEndDateUtc]');
        function toggleSubscriptionEndDateDiv() {
            if ($isUnlimitedInput.is(':checked')) {
                $subscriptionEndDateDiv.slideUp('fast');
                subscriptionEndDateUtcInput.removeAttr('required');
            } else {
                $subscriptionEndDateDiv.slideDown('fast');
                subscriptionEndDateUtcInput.attr('required', 'required');
            }
        }

        $isUnlimitedInput.change(function () {
            toggleSubscriptionEndDateDiv();
        });

        var $editionCombobox = $('#EditionId');
        $editionCombobox.change(function () {
            var isFree = $('option:selected', this).attr('data-isfree') === "True";
            var selectedValue = $('option:selected', this).val();

            if (selectedValue === "" || isFree) {
                $('.subscription-component').slideUp('fast');
                if (isFree) {
                    $isUnlimitedInput.prop("checked", true);
                } else {
                    $isUnlimitedInput.prop("checked", false);
                }
            } else {
                $('.subscription-component').slideDown('fast');
            }
        });

        toggleSubscriptionEndDateDiv();
        $editionCombobox.trigger('change');

        function save() {
            if (!_$tenantInformationForm.valid()) {
                return;
            }

            var tenant = _$tenantInformationForm.serializeFormToObject();

            //take selected date as UTC
            if ($('#CreateTenant_IsUnlimited').is(':visible') && !$('#CreateTenant_IsUnlimited').is(':checked')) {
                tenant.SubscriptionEndDateUtc = $('.date-time-picker').data("DateTimePicker").date().format("YYYY-MM-DDTHH:mm:ss") + 'Z';
            } else {
                tenant.SubscriptionEndDateUtc = null;
            }

            if (tenant.SetRandomPassword) {
                tenant.Password = null;
                tenant.AdminPassword = null;
            }

            if (tenant.UseHostDb) {
                tenant.ConnectionString = null;
            }

            abp.ui.setBusy();
            _tenantService.createTenant(
                tenant
            ).done(function () {
                abp.notify.info(app.localize('SavedSuccessfully'));
                window.location.href = "/AppAreaName/Tenants";
            }).always(function () {
                abp.ui.clearBusy();
            });
        };

        $("#btnCreateTenant").click(save);//Now there is no modal button. Thats why you need to call save function with another button
    });
})();
```

And finally go **Index.cshtml** and change **Create new tenant** button as seen below

```html
<a href="/[AppAreaName]/Tenants/Create" class="btn btn-primary">
    <i class="fa fa-plus"></i> @L("CreateNewTenant")
</a>
```

### Result

![converting-modal-to-page-tenant-create](images/converting-modal-to-page-tenant-create.png)





## Edit Tenant

Converting edit modal to edit page has same step with converting create modal to create page. The results will be as seen below.

### Create Action Result

*TenantsController.cs*

```csharp
[AbpMvcAuthorize(AppPermissions.Pages_Tenants_Edit)]
public async Task<ActionResult> Edit(int id)
{
    var tenantEditDto = await _tenantAppService.GetTenantForEdit(new EntityDto(id));
    var editionItems = await _editionAppService.GetEditionComboboxItems(tenantEditDto.EditionId);
    var viewModel = new EditTenantViewModel(tenantEditDto, editionItems);

    return View(viewModel);
}
```



### Create View File

*Edit.cshtml*

```html
@using Abp.Extensions
@using MyCompanyName.AbpZeroTemplate.MultiTenancy
@using MyCompanyName.AbpZeroTemplate.Web.Areas.AppAreaName.Models.Common.Modals
@using MyCompanyName.AbpZeroTemplate.Web.Areas.AppAreaName.Models.Tenants
@model EditTenantViewModel

@section Scripts
{
    <script src="~/view-resources/Areas/AppAreaName/Views/Tenants/Edit.js"></script>
}

<div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">

    <div class="kt-subheader kt-grid__item">
        <div class="@(await GetContainerClass())">
            <div class="kt-subheader__main">
                <h3 class="kt-subheader__title">
                    <span>@L("EditTenant")</span>
                </h3>
                <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                <span class="kt-subheader__desc">
                    @Model.Tenant.TenancyName
                </span>
            </div>
        </div>
    </div>
    <div class="@(await GetContainerClass()) kt-grid__item kt-grid__item--fluid">
        <div class="kt-portlet">
            <div class="kt-portlet__body">
                <form name="TenantInformationsForm">

                    <input type="hidden" name="Id" value="@Model.Tenant.Id" />
                    <input type="hidden" name="TenancyName" value="@Model.Tenant.TenancyName" />

                    <div class="form-group">
                        <label for="Name">@L("Name")</label>
                        <input id="Name" type="text" name="Name" value="@Model.Tenant.Name" class="form-control edited" required maxlength="@Tenant.MaxNameLength">
                    </div>

                    @if (!Model.Tenant.ConnectionString.IsNullOrEmpty())
                    {
                        <div class="form-group">
                            <label for="ConnectionString">@L("DatabaseConnectionString")</label>
                            <input id="ConnectionString" type="text" name="ConnectionString" class="form-control edited" value="@Model.Tenant.ConnectionString" required maxlength="@Tenant.MaxConnectionStringLength">
                        </div>

                        <div>
                            <span class="help-block text-warning">@L("TenantDatabaseConnectionStringChangeWarningMessage")</span>
                        </div>
                    }

                    <div class="form-group">
                        <label for="EditionId">@L("Edition")</label>
                        <select class="form-control" id="EditionId" name="EditionId">
                            @foreach (var edition in Model.EditionItems)
                            {
                                <option value="@edition.Value" data-isfree="@edition.IsFree" selected="@edition.IsSelected">@edition.DisplayText</option>
                            }
                        </select>
                    </div>

                    <div class="kt-checkbox-list subscription-component">
                        <label class="kt-checkbox">
                            <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited" @(!Model.Tenant.SubscriptionEndDateUtc.HasValue ? "checked=\"checked\"" : "") />
                            @L("UnlimitedTimeSubscription")
                            <span></span>
                        </label>
                    </div>

                    <div class="form-group position-relative">
                        <label for="SubscriptionEndDateUtc">@L("SubscriptionEndDateUtc")</label>
                        <input id="SubscriptionEndDateUtc" type="datetime" name="SubscriptionEndDateUtc" value="@(Model.Tenant.SubscriptionEndDateUtc)" class="form-control edited date-time-picker" @(!Model.Tenant.SubscriptionEndDateUtc.HasValue ? "required" : "")>
                    </div>

                    <div class="kt-checkbox-list subscription-component">
                        <label class="kt-checkbox">
                            <input id="EditTenant_IsInTrialPeriod" type="checkbox" name="IsInTrialPeriod" value="true" @(Model.Tenant.IsInTrialPeriod ? "checked=\"checked\"" : "") />
                            @L("IsInTrialPeriod")
                            <span></span>
                        </label>
                    </div>

                    <div class="kt-checkbox-list">
                        <label class="kt-checkbox">
                            <input id="EditTenant_IsActive" type="checkbox" name="IsActive" value="true" @Html.Raw(Model.Tenant.IsActive ? "checked=\"checked\"" : "")>
                            @L("Active")
                            <span></span>
                        </label>
                    </div>

                </form>
            </div>
            <div class="kt-portlet__foot">
                <div class="row align-items-center">
                    <div class="col-md-12 kt-align-right pr-5">
                        <a href="/AppAreaName/Tenants" class="kt-link kt-font-bold">Cancel</a>
                        <span class="kt-margin-left-10">or</span>  <button id="btnEditTenant" class="btn btn-brand">Submit</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

```

### Create Javascript File

*Edit.js*

```javascript
(function () {
    $(function () {
        var _tenantService = abp.services.app.tenant;
        var _$tenantInformationForm = null;


        _$tenantInformationForm = $('form[name=TenantInformationsForm]');
        _$tenantInformationForm.validate();

        $('.date-time-picker').datetimepicker({
            locale: abp.localization.currentLanguage.name,
            format: 'L'
        });

        var $subscriptionEndDateDiv = $('input[name=SubscriptionEndDateUtc]').parent('div');
        var isUnlimitedInput = $('#CreateTenant_IsUnlimited');
        function toggleSubscriptionEndDateDiv() {
            if (isUnlimitedInput.is(':checked')) {
                $subscriptionEndDateDiv.slideUp('fast');
            } else {
                $subscriptionEndDateDiv.slideDown('fast');
            }
        }

        isUnlimitedInput.change(function () {
            toggleSubscriptionEndDateDiv();
        });

        var $editionCombobox = $('#EditionId');
        var $isInTrialCheckbox = $('#EditTenant_IsInTrialPeriod');
        $editionCombobox.change(function () {
            var isFree = $('option:selected', this).attr('data-isfree') === "True";
            var selectedValue = $('option:selected', this).val();
            if (isFree) {
                $isInTrialCheckbox.closest('div').slideUp('fast');
            } else {
                $isInTrialCheckbox.closest('div').slideDown('fast');
            }

            if (selectedValue <= 0) {
                $('.subscription-component').slideUp('fast');
                if (!isUnlimitedInput.is(':checked')) {
                    $subscriptionEndDateDiv.slideDown('fast');
                }
            } else {
                $('.subscription-component').slideDown('fast');
            }
        });

        $editionCombobox.trigger('change');
        toggleSubscriptionEndDateDiv();


        function edit() {
            if (!_$tenantInformationForm.valid()) {
                return;
            }

            var tenant = _$tenantInformationForm.serializeFormToObject();

            //take selected date as UTC
            if ($('#CreateTenant_IsUnlimited').is(':visible') && !$('#CreateTenant_IsUnlimited').is(':checked')) {
                tenant.SubscriptionEndDateUtc = $('.date-time-picker').data('DateTimePicker').date().format("YYYY-MM-DDTHH:mm:ss") + 'Z';
            } else {
                tenant.SubscriptionEndDateUtc = null;
            }

            abp.ui.setBusy();
            _tenantService.updateTenant(
                tenant
            ).done(function () {
                abp.notify.info(app.localize('SavedSuccessfully'));
                location.href="/AppAreaName/Tenants";
            }).always(function () {
               abp.ui.clearBusy();
            });
        };

        $("#btnEditTenant").click(edit);
    });
})();
```

Then go index.js and change edit button on data table.

```javascript
{
    text: app.localize('Edit'),
    visible: function () {
        return _permissions.edit;
    },
    action: function (data) {
        location.href = "/AppAreaName/Tenants/Edit?id=" + data.record.id;//change that
    }
},
```

### Result

![converting-modal-to-page-tenant-edit](images/converting-modal-to-page-tenant-edit.png)