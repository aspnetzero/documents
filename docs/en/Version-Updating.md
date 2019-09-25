# Version Updating

ASP.NET ZERO is designed to be a startup template. Updatability of templates is not automatic and straightforward as you know. We do not officially support updating the ASP.NET ZERO version.

But we suggest a solution:

- In the beginning, create your project, for example, named 'Acme.PhoneBook' from aspnetzero.com

- Add it to source control (like Git). But do not add it publicly to Github since our source code is private :)

- For example, push it to the 'master' branch.

- Create a 'dev' branch. Make all your development in this branch.

  ![version-update-new-brach](C:\Users\Musa\Desktop\documents\docs\en\images\version-update-new-brach.png)

- When a new version of ASP.NET ZERO is released, re-create your project from scratch with the same name, 'Acme.PhoneBook'. (Make sure you create the new project with the same name as the old project. Otherwise, you can not merge them.)

- Override all files to your master branch and commit.

- Switch to dev branch and merge branch master into dev. There will be conflicts, resolve them manually.

![version-update-update-from-master](C:\Users\Musa\Desktop\documents\docs\en\images\version-update-update-from-master.png)

That's all. Your dev and master branches will have a new ASP.NET ZERO version.

Notes:

 Deploy always from the dev branch.

 Do not use the branch where you keep the main code for other things. (in this example it was master branch)

