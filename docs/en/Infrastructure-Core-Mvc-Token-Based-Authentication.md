# Token Based Authentication

Any application can authenticate and use any functionality in the application as remote API. For instance, you can create a mobile application consumes the same API. In this section, we'll demonstrate the usage of the remote API using [Postman](https://www.getpostman.com/docs/introduction) (a Google Chrome extension).

## Authentication

We suggest you to disable two factor authentication for the user which will be used for remote authentication. Otherwise, two factor authentication flow should be implemented by the client. We assume that you have disabled two factor authentication for the **admin** user of **default** tenant since we will use it in this sample.

Following headers should be configured for all requests (`Abp.TenantId` is Id of the default tenant. This is not required for single tenant applications or if you want to work with host users):

<img src="images/postman-ng2-auth-headers.png" alt="Postman auth headers" class="img-thumbnail" width="523" height="112" />

Then we can send username and password as a **POST** request to https://localhost:44302/api/TokenAuth/Authenticate

<img src="images/postman-authenticate-core-2.png" alt="Postman get user list" class="img-thumbnail" width="919" height="1023" />

In the returning response, **accessToken** will be used to authorize for the API.

# Using API

After authenticate and get the access token, we can use it to call any **authorized** actions. All **services** are available to be used remotely. For example, we can use the **User service** to get a **list of users**:

<img src="images/postman-getusers-core-2.png" alt="Postman authentication" class="img-thumbnail" width="919" height="1023" />

We sent a GET request to https://localhost:44302/api/services/app/User/GetUsers and added
Authorization to the header as "**Bearer &lt;accessToken&gt;**". Returning JSON contains the list of users.