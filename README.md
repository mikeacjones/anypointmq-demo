# Anypoint MQ Demo

This demo asset includes two sample projects:
* A simple queue syncing data from a MongoDB to Salesforce (Account object) as new documents are added to the Mongo collection
* An Async API that utilizes an Exchange to push new notifications to three queues to delivery three separate notifications, finally updating the notification from pending to complete.



## Installation

Download the projects and import into your workspace. Each project contains a config file, `config.secure.sandbox.yaml`

You will need to update the configuration for each project with your own credentials, encrypted using the Blowfish algorithm and the CBC state. I recommend that you use [this tool](https://secure-properties-api.us-e1.cloudhub.io/) to facilitate. When deploying, the encryption key should be included as a property called `securePropertiesKey`.

#### Local Secure Property Encrtyption
For local development, I recommend adding a `local-props.xml` to the project (.gitignore will prevent accidentally commiting) and adding the global property `securePropertiesKey`. You can also add it as a vm argument in studio, `-M-DsecurePropertiesKey=`.

#### MongoDB
The project is setup to use a connection string. I recommend using [mongodb.com](https://mongodb.com) as a free tier host as the free tier never expires.
In `global.xml` in each project that utilizes a MongoDB, you will need to update the URL to match your setup; the database utilized is controlled via the properties file.

#### Email
[You will need an app token from Gmail](https://support.google.com/mail/answer/185833?hl=en). You can not create this token using your work email, and so will need to use either a personal Gmail account or [create a new Gmail account](https://accounts.google.com/signup).

#### Notification GroupID

The groupID for notifications is currently hardcoded in the `notification-process` project. In `notification-process.xml` you will need to update the variable `recipients`. This variable should be a JSON map with group IDs linked to an array of email address.

Example:

```dataweave
%dw 2.0
output application/json
---
var groups = {
  "0": [], //groupID 0 sends email to no-one
  "1": [
    "example@test.com",
    "example2@test.com"
  ]
}
```

## Usage

#### MongoDB to Salesforce Queue

Simle insert a new element into the document collection; I recommend using (Compass)[https://www.mongodb.com/products/compass] to insert the documents. Structure should look like this:

```JSON
{
    "_id": {
        "$oid": "5f7486a192ea33fe3e2f08b9"
    },
    "name": "Bob Smith - Demo Account Template",
    "dateAdded": {
        "$date": "2020-09-30T04:00:00.000Z"
    },
    "lastModified": {
        "$date": "2020-09-30T13:19:51.137Z"
    },
    "billing": {
        "state": "GA",
        "city": "Atlanta",
        "zip": 55555
    },
    "contact": {
        "phone": "555-555-5555",
        "email": "example@test.com"
    }
}
```

#### Async Notification API

The RAML spec can be found here: https://anypoint.mulesoft.com/exchange/portals/mulesoft-6908/968cf9cf-415e-4340-817c-e0e32f24a22e/notification-sapi/

When posting a new notification, you will receive a 201 created response as well as an ID that can be used to query the status of the notification.

Example return:

```JSON
{
"id": "5f7486a192ea33fe3e2f08b9",
"status": "pending"
}
```

You can then use this ID to query `/notification/{id}` which will return only the status of the notification. For full details on the notification, GET `/notification/{id}?details=true`. When doing a GET on `/notification` you can also include `/notification?status=complete`.

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
