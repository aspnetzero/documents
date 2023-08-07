# How to Create & Edit Power Tools Templates

Power Tools uses text templates for code generation, and these templates are located inside `/AspNetZeroRadTool/FileTemplates` directory in your project's root directory.

**MainTemplate.txt:** Power Tools uses this template for main code generation.
**PartialTemplates.txt:** Power Tools renders some placeholders in `MainTemplate.txt` conditionally. These conditional templates are stored in `PartialTemplates.txt`.
**TemplateInfo.txt:** Stores information about the template like path and condition.

*Folder structure*

![Folder structure](images/power-tools-folder-structure.png)

## Edit Pre-defined Templates

If you want to edit any file, copy it in the same directory and change it's an extension to `.custom.txt` from `.txt`. For example, you can create `MainTemplate.custom.txt` to override `MainTemplate.txt` in the same directory. Please don't make any changes to the original templates.

![Edit Template](images/power-tools-edit-template.png)

## Create New Templates

To create new templates, simply add a new file to the `FileTemplates` directory. You can use the existing templates as a reference. For guidance on how templates function and how to use placeholders, please refer to the documentation provided at [Understanding Power Tools Templates](power-tools-understanding-power-tools-templates.md).

Power Tools discovers templates in the `FileTemplates` directory every time it is run. So, restarting Power Tools will find your newly created templates.

## Change Destination Path Of New Files

To change the destination path of a template, find the template folder of it in `AspNetZeroRadTool/FileTemplates` directory and edit the content of `TemplateInfo.txt` file.