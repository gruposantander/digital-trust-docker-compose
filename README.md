# digital-trust-docker-compose

Docker compose to run all Digital Trust Protocol components in local.
Please go to [https://gruposantander.github.io/digital-trust-docs/](https://gruposantander.github.io/digital-trust-docs/) for more info.

## Setup 

Configure `docker-compose.env` with the following parameters:

* CLIENT_BASE_URL={URL TO YOUR GRAPHQL API IF DIFFERENT TO PROVIDED STUBS}
* REGISTRATION=active (enables /reg endpoint)

Included in this bundle you have a component running a graphql stubs servers for demo purposes. If you want to customize your users data, edit the files under `mocks/people` folder

There are other internal configuration parameters available under `config` folders. Refer to the additional repos and documentation for further details.

## Up server

First you will need to login to github docker:

```bash
docker login docker.pkg.github.com -u $USERNAME -p $GITHUB_TOKEN
```

Your github token can be obtained from your [profile settings](https://github.com/settings/tokens). Create a PAC with access to packages. 

Once you are logged in, execute the following command to pull images and start all the components:

```bash
docker-compose up -d
```

## Run Demo Application

We have included a demo application to showcase E2E interaction with a fake RP (Relying Party). Once the containers are up, you would be able to use the default application, which is specified in the `docker-compose.yml` file as `CLIENT_ID` env property for the `dtp-demo` service. To run the demo, just open [http://localhost:4201](http://localhost:4201) in your browser and follow these steps:

* Click on Notifications button and click on the message
* Click on Verify with Santander and enter test user credential (hilton/123)

If you want to run again the demo, restart the container with `docker-compose restart dtp-demo`, or **[click here](#backend)** to understand how to interact with the backend using a postman collection.

## Postman Collection for further tests

There is a folder under test with a postman collection. Collection is self documented, execute the steps in order to register an application and create a sample request.
You should be able to use any of the test data users in the `mocks/people` folder files (enter username as per file name and any password, for example sara/1234).

## Interaction with the Demo App

Below you will find the endpoints available to interact with the demo app.

### GET /user-info (front-end)

Retrieve the information stored about the default user. This is used in Quickjobs, but can be directly accessed.

### POST /user-info (front-end)

Set the information stored about the user inside the Quickjobs system. This is the data that is used for the assertions that are made in the consent flow. Request body:

``` json
{
    "given_name": "string",
    "family_name": "string",
    "country_of_birth": "string",
    "title": "string",
    "address": {
        "street_address": "string",
        "locality": "string",
        "postal_code": "string",
        "country": "string",
    }
}
```

All fields are essential, and there is validation to ensure this is followed.

### POST /verified (back-end)

Attach a query param of `value=${VERIFIED_VALUE}` to this endpoint. Anything apart from `true` (including not attaching a query param) will result in `verified = false`.

### PATCH /reset (back-end)

Resets to default values for userDetails and verified returns to false. The default userDetails are as follows:

```json
{
    "title": "Mrs",
    "given_name": "Yost",
    "family_name": "Hilton",
    "country_of_birth": "GB",
    "address": {
        "street_address": "19 Kacey Forest",
        "locality": "Redding",
        "postal_code": "QZBAD9",
        "country": "United Kingdom"
    }
}
```

### GET /initiate-authorize (back-end)

Returns the url required for the user to consent in the flow. Uses the SDK to create the request body to start the flow, create the client and execute.

### POST /token (back-end)

Request body sends the code that is returned to the frontend from the digital ID flow. The endpoint uses the SDK to translate the returned JWT to a readable JSON object. Checks are done on the assertions returned to make sure that the essential fields were returned correctly.
