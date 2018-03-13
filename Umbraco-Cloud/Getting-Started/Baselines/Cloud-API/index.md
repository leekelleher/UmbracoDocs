# Public Umbraco Cloud REST API

Umbraco Cloud has a REST API that you can use to automatically create new projects. This can come in really handy if you are going to create a lot of projects within a short period of time.

In this article you'll learn how to setup Umbraco Cloud projects using the REST API, and also how to invite and add users to these projects.

## Authentication

In order to start using the API you will need to have a **valid access token**. The token will be used in the **HTTP Authorization header** that you need to send along with every request you make to the API. Please contact our support about how to acquire a token, either through the chat on [Umbraco.io](https://www.s1.umbraco.io) or through contact@umbraco.com.

The scheme we use is called `token` and it's constructed by combining the email you are using with Umbraco.io and the provided token with a colon:

    user-email@example.com:6a5b0c687b7973db4b2568a59f2c446ee1ee9f9f06322f1bd6983718a48b5a6c

This then needs to be `base64` encoded (encoding should be UTF8):

    dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj

The resulting header should look like this:

    Authorization: token dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj

## Project endpoints

### Create project

The following endpoint should be used for creating new projects on Umbraco Cloud:

    POST https://www.s1.umbraco.io/api/public/project/create

**BEWARE** that this endpoint is subject to throttling; you can only create x number of projects every x number of minutes (the actual numbers can vary but will be written in the HTTP message). If you get an HTTP status code of `409 (Conflict)` it means that you are being throttled.

To use this endpoint, make a request like this:

**Request**

    POST https://www.s1.umbraco.io/api/public/project/create
    Authorization: token dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj
    Content-Type: application/json

    {
        "projectName": "Name of the project", // Required
        "plan": "Single", // Required. Options are: "Single or Studio"
        "ownerId": "c5821a98-ce88-4796-be90-e29f0a05fa39", // Optional. Can be a guid of an organization or an email. If nothing is provided the owner becomes the user the token is associated with
        "baselineAlias": "an-alias-of-a-baseline" // Optional. If the project needs to be a child, then you can provide the alias of the baseline
    }

If everything went as expected, the endpoint will return a `HTTP 200` status code and a JSON object that looks like this:

**Response**

    {
        "url": "https://www.s1.umbraco.io/project/alias-of-project,
        "alias": "alias-of-project",
        "projectId: "2af1dc0e-a454-4956-ba26-59036ac4bb99",
        "projectIsReady": false,
        "creationStatus": "Creating",
        "creationStatusEndpoint": "https://www.s1.umbraco.io/api/public/project/creationstatus"
    }

Because the creation of a project happens asynchronously we are providing you with another endpoint for checking the status of the creation (the creationStatusEndpoint listed in the above object) if you need to know when it is done. Read more about this endpoint below.

### Project creating status

The following endpoint is used to check the status of a project creation:

    POST https://www.s1.umbraco.io/api/public/project/creationstatus

The project id needs to passed along as a HTTP header called `X-Project-Id` - e.g. `X-Project-Id: 2af1dc0e-a454-4956-ba26-59036ac4bb99`

**Request**

    POST https://www.s1.umbraco.io/api/public/project/creationstatus
    Authorization: token dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj
    X-Project-Id: 2af1dc0e-a454-4956-ba26-59036ac4bb99
    Content-Type: application/json

The request above will return a JSON object with a `creationStatus` which will tell you whether a project is created or still in the process of being created.

In the example below the project creation is still under-way:

    {
        "projectIsReady": false,
        "creationStatus": "Creating",
        "creationStatusEndpoint": "https://www.s1.umbraco.io/api/public/project/creationstatus",
        "backofficeUrl": ""
    }

You should keep polling this until the `creationStatus` changes to "Created" (or `projectIsReady=true`). The `backofficeUrl` will also be filled with the correct url to the backoffice once the project has been created.

### Invite to project

The following endpoint is used for inviting users to a project. 

    POST https://www.s1.umbraco.io/api/public/project/invite

If the user is a new user we will create the user and send an activation email to the provided email and then add the project to the users overview. If the user is an existing user he/she will get an email telling them that they have been invited to the project.

**Request**

    POST https://www.s1.umbraco.io/api/public/project/invite
    Authorization: token dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj
    X-Project-Id: 2af1dc0e-a454-4956-ba26-59036ac4bb99
    Content-Type: application/json
                            
    {
        "email": "user-email@example.com", // Required
        "name": "The users name", // Required
        "isAdmin": false, //Optional
        "title": "A title that goes into the title of the mail sent to the user", // Optional
        "message": "A message that goes into the message of the mail sent to the user" // Optional
    }

This will return an appropriate HTTP status code and a JSON object like this:

**Response**

    {
        "message": "An invitation to collaborate on 'Project Name' has been sent to 'user-email@example.com'"
    }

## User endpoints

### Create a user

The following endpoint is used for creating a user on Umbraco.io, and not necessarily add them to a project right away.

    POST https://www.s1.umbraco.io/api/public/user/create

**Request**

    POST https://www.s1.umbraco.io/api/public/user/create
    Authorization: token dXNlci1lbWFpbEBleGFtcGxlLmNvbTo2YTViMGM2ODdiNzk3M2RiNGIyNTY4YTU5ZjJjNDQ2ZWUxZWU5ZjlmMDYzMjJmMWJkNjk4MzcxOGE0OGI1YTZj
    Content-Type: application/json
                            
    {
        "email": "user-email@example.com", // Required
        "name": "The users name", // Required
        "password": "dontReuseOrRecycleYourPassword" //Required
    }

This will return an appropriate HTTP status code and a JSON object like this:

**Response**

    {
        "message": "User created"
    }
