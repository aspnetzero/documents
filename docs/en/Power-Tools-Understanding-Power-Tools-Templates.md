# Understanding Power Tools Templates

Power Tools uses templates to generate the code for the various features. This allows you to customize the code that is generated to fit your needs. Before starting this section, please read the [How to Create & Edit Power Tools Templates](how-to-create-edit-power-tools-templates.md) section.

Basically we are using partial templates to generate code in main template. Let's take a quick look:

*MainTemplate.txt*

```csharp
{{Enum_Using_Looped_Template_Here}}{{NP_Using_Looped_Template_Here}}using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Abp.Domain.Entities.Auditing;
using Abp.Domain.Entities;{{Using_Auditing_Here}}

namespace {{Namespace_Here}}.{{Namespace_Relative_Full_Here}}
{
	[Table("{{Table_Name_Here}}")]{{Auditing_Attr_Here}}
    public{{Overridable_Entity_Abstract_Here}} class {{Entity_Name_Here}}{{Overridable_Entity_Base_Here}} : {{Base_Class_Here}}{{Primary_Key_Inside_Tag_Here}} {{May_Or_Must_Tenant_Here}}
    {{{Tenant_Id_Here}}
{{Property_Looped_Template_Here}}
{{Navigation_Property_Looped_Template_Here}}
    }
}
```

*A part of PartialTemplate.txt*

```json
{
"propertyTemplates":[
		{
			"placeholder" : "{{Property_Looped_Template_Here}}",
			"condition" : "",
			"templates" : [
					{
					"type" : "default",
					"content" : "{{Required}}{{Regex}}{{MixMaxLength}}{{Range}}
		public virtual {{Property_Type_Here}}{{Property_Nullable_Question_Mark_Here}} {{Property_Name_Here}} { get; set; }
		"
					},
					{
					"type" : "file",
					"content" : "//File
					{{Required}}{{Regex}}{{MixMaxLength}}{{Range}}
		public virtual Guid? {{Property_Name_Here}} { get; set; } //File, (BinaryObjectId)
		"
					},
				]
		}
	],
    .
    .
    .
}
```

`Property_Looped_Template_Here` is a placeholder in `MainTemplate.txt` and it is replaced with the content of `Property_Looped_Template_Here` in `PartialTemplate.txt`. 

## Partial Template Types

There are four types of partial templates:

### `propertyTemplates`

The property templates are used to generate the properties of the entity. It will loop entire content of `propertyTemplates` for each property of the entity.

*Example property template object*
```json
{
    "placeholder" : "{{Property_Looped_Template_Here}}",
    "condition" : "",
    "templates" : [
            {
            "type" : "default",
            "content" : "{{Required}}{{Regex}}{{MixMaxLength}}{{Range}}
public virtual {{Property_Type_Here}}{{Property_Nullable_Question_Mark_Here}} {{Property_Name_Here}} { get; set; }
"
            },
            {
            "type" : "file",
            "content" : "//File
            {{Required}}{{Regex}}{{MixMaxLength}}{{Range}}
public virtual Guid? {{Property_Name_Here}} { get; set; } //File, (BinaryObjectId)
"
            },
        ]
}
```

This property template will generate a content for each property of the entity. If the property is a file, it will generate a content for file property. If the property is not a file, it will generate a content for default property.

### `enumTemplates`

*Example enum template object*

```json
{
    "placeholder" : "{{Enum_Using_Looped_Template_Here}}",
    "preventDuplicate":true,
    "content" : "using {{Enum_Namespace_Here}};"
}
```

This enum template will generate a content for each enum of the entity. It appends the `using` statement for each enum.

### `conditionalTemplates`

Conditional templates are used to generate a content if the condition is true. It will not generate a content if the condition is false.

```json
{
    "placeholder": "{{May_Or_Must_Tenant_Here}}",
    "condition": "{{Is_Available_To_Host_Here}} == true && {{Is_Available_To_Tenant_Here}} == true",
    "content": ", IMayHaveTenant"
},
```

This conditional template will generate a content if the entity is available to host and tenant. It will not generate a content if the entity is not available to host or tenant.

### `navigationPropertyTemplates`

Navigation property templates are used to generate the navigation properties of the entity(foreign key / master-detail). It will loop entire content of `navigationPropertyTemplates` for each navigation property of the entity.

```json
{
    "placeholder" : "{{NP_Using_Looped_Template_Here}}",
    "templates" : [
            {
            "relation" : "single",
            "content" : "using {{NP_Namespace_Here}};"
            },
            {
            "relation" : "multi",
            "content" : "using {{NP_Namespace_Here}};"
            }
        ]
}
```

This navigation property template will generate a content for each navigation property of the entity. If the navigation property is a single relation, it will generate a content for single relation. If the navigation property is a multi relation, it will generate a content for multi relation.

## Template Info

Template info is used to define the path and condition for the template. It is used to generate the code for the feature.

*Example TemplateInfo.txt*
```json
{
	"path" : "{{Namespace_Here}}.Core\\{{Namespace_Relative_Full_Reverse_Slash_Here}}\\{{Entity_Name_Here}}.cs",
	"condition": "{{Is_Master_Detail_Page_Child_Here}} == false",
}
```

## Placeholders

Placeholders are used to replace the content of the template. As you are familiar with from many programming tools and languages, `{{placeholder}}` is used to define a placeholder.

### Predefined Placeholders

| Placeholder | Description | 
| --- | --- |
| {{Namespace_Here}} | Namespace of the project |
| {{App_Area_Name_Here}} | Area name of the project |
| {{Entity_Name_Here}} | Entity name |
| {{Table_Name_Here}} | Table name of the entity |
| {{Entity_History_Here}} | Is entity history enabled |
| {{Menu_Position_Here}} | `Admin` or `menu` |
| {{Namespace_Relative_Here}} | Entity namespace relative to the project |
| {{Namespace_Relative_Full_Here}} | Entity namespace relative to the project with full path |
| {{Namespace_Relative_Full_Reverse_Slash_Here}} | Entity namespace relative to the project with full path and reverse slash |
| {{Entity_Name_Plural_Here}} | Entity name in plural form |
| {{Project_Name_Here}} | Project name |
| {{Base_Class_Here}} | Base class of the entity |
| {{Primary_Key_Here}} | Primary key of the entity |
| {{Permission_Name_Here}} | Permission name of the entity |
| {{Permission_Value_Here}} | Permission value of the entity |
| {{Page_Names_Sub_Class_Name_Here}} | Page names sub class name of the entity |
| {{Is_Available_To_Host_Here}} | Is entity available to host |
| {{Is_Available_To_Tenant_Here}} | Is entity available to tenant |
| {{Create_View_Only_Here}} | Is entity view only |
| {{Create_Excel_Export_Here}} | Is entity excel export enabled |
| {{Project_Version_Here}} | Project version |
| {{Property_Name_Here}} | Property name |
| {{Property_Type_Here}} | Property type |
| {{Property_MaxLength_Here}} | Property max length |
| {{Property_MinLength_Here}} | Property min length |
| {{Property_RangeMin_Here}} | Property range min |
| {{Property_RangeMax_Here}} | Property range max |
| {{Property_Nullable_Question_Mark_Here}} | Property nullable question mark |
| {{Property_Is_Nullable_Here}} | Is property nullable |
| {{Property_Is_Range_Set_Here}} | Is property range set |
| {{Property_Regex_Here}} | Property regex |
| {{Property_Required_Here}} | Is property required |
| {{Property_Listed_Here}} | Is property listed |
| {{Property_CreateOrEdit_Here}} | Is property create or edit |
| {{Is_That_Property_Enum}} | Is that property enum |
| {{Property_Advanced_Filter_Here}} | Is advanced filter enabled |
| {{Property_Is_Decimal_Here}} | Is property decimal |
| {{Property_Has_Decimal_Precision_Here}} | Is property has decimal precision |
| {{Property_Decimal_Precision_Here}} | Property decimal precision |
| {{Enum_Name_Here}} | Enum name |
| {{Enum_Count_Here}} | Enum count |
| {{Enum_Namespace_Here}} | Enum namespace |
| {{Enum_Property_Name_Here}} | Enum property name |
| {{Enum_Property_Value_Here}} | Enum property value |
| {{Enum_Used_For_Property_Name_Here}} | Enum used for property name |
| {{Is_Non_Modal_CRUD_Page}} | Is non modal crud page |
| {{Add_Modal_If_Modal_Based_CRUD_Page}} | Add modal if modal based crud page |
| {{Is_Master_Detail_Page}} | Is master detail page |
| {{Is_Master_Detail_Page_Child_Here}} | Is master detail page child |
| {{Master_Detail_Child_Prefix_Here}} | Master detail child prefix |
| {{Master_Detail_Child_Base_Entity_Name_Here}} | Master detail child base entity name |
| {{Master_Detail_Child_Base_Entity_Type_Here}} | Master detail child base entity type |
| {{Master_Detail_Child_Foreign_Property_Name_Here}} | Master detail child foreign property name |
| {{Generate_Overridable_Entity_Here}} | Generate overridable entity enabled |
| {{Navigation_Property_One_To_Many_Td_Col_Span_Here}} | Navigation property one to many td col span |
| {{Has_File_Prop}} | Has file property |
| {{Project_Type_Here}} | `Angular` or `Mvc` |