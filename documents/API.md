## API

**Intended Audience**: this document describes the programming interface for the ingest and reporting services of the Reporting Data Warehouse (RDW). Operations
and system adminstration will find it useful for the configuration data loading end-points and the diagnostic and task service end-points. Third party providers
of test results will find it useful for the data loading end-points.

This document describes the service end-points for the ingest and reporting services. *Note: there are many more end-points in the reporting services but most are
intended to be consumed by the reporting UI, so are not documented here.
* The import service has data loading end-points.
* The task and report processor services have task trigger end-points.
* All services have diagnostic end-points.

Quick Links:
1. [Authentication and Authorization](#authentication-and-authorization)
1. [Import Endpoints](#import-endpoints)
1. [Exam Endpoints](#exam-endpoints)
1. [Groups Endpoints](#groups-endpoints)

### Authentication and Authorization
The import service requires an OAuth2 access token for the data loading end-points. This is a password grant token requested from the OAuth2 server by a trusted 
client for a user of the system (the permissions are associated with the user).
During the first couple years, the OAuth2 server was an OpenAM server. Recently it was migrated to an Okta server. There are instructions for both.

#### Fetch Password Grant Access Token - OpenAM
Accepts x-www-form-urlencoded data including client and user credentials and returns an access token.
* Host: OpenAM
* URL: `/auth/oauth2/access_token`
* Method: `POST`
* URL Params:
  * `realm=/sbac`
* Data Params (except for `grant_type` values given are examples and should be replaced with real values):
  * `grant_type=password`
  * `username=user@example.com`
  * `password=UserPassword`
  * `client_id=client`
  * `client_secret=ClientSecret`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "scope": "cn givenName mail sbacTenancyChain sbacUUID sn",
      "expires_in": 36000,
      "token_type": "Bearer",
      "refresh_token": "639ddaa9-f993-4c52-aec5-923d5a21ee23",
      "access_token": "20b55fc2-1b84-4412-8149-88cfa622db01"
    }
    ```
* Error Response:
  * Code: 400 (Bad Request)
  * Content (specific error and description will vary):
    ```json
    {
      "error": "invalid_client",
      "error_description": "Client authentication failed"
    }
    ```
* Sample Call (curl):

`curl -s -X POST
  --data "grant_type=password&username=user@example.com&password=password&client_id=client&client_secret=secret"
  https://openam/auth/oauth2/access_token?realm=/sbac`

* Extracting the access token. A handy util is `jq` which parses json payloads. Using it the access token may be extracted from the payload, something like:

`ACCESS_TOKEN=curl -s -X POST
  --data "grant_type=password&username=user@example.com&password=password&client_id=client&client_secret=secret"
  https://openam/auth/oauth2/access_token?realm=/sbac | jq -r '.access_token'`

The resulting environment variable may be used in subsequent calls. Where the samples have `{access_token}` substitute `${ACCESS_TOKEN}`.

#### Fetch Password Grant Access Token - Okta
Accepts x-www-form-urlencoded data including client and user credentials and returns an access token.
* Host: Okta
* URL: `/oauth2/auslw2qcsmsUgzsqr0h7/v1/token` (the exact URL will depend on the installation configuration)
* Method: `POST`
* Data Params (values given are examples and should be replaced with real values):
  * `grant_type=password`
  * `scope=openid profile`
  * `username=user@example.com`
  * `password=UserPassword`
  * `client_id=client`
  * `client_secret=ClientSecret`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "scope": "openid profile",
      "expires_in": 3600,
      "token_type": "Bearer",
      "id_token": "eyJraWQi0...",
      "access_token": "eja8FA..."
    }
    ```
* Error Response:
  * Code: 400 (Bad Request)
  * Content (specific error and description will vary):
    ```json
    {
      "error": "invalid_client",
      "error_description": "Client authentication failed"
    }
    ```
* Sample Call (curl):

`curl -s -X POST
  --data "grant_type=password&scope=openid profile&username=user@example.com&password=password&client_id=client&client_secret=secret"
  https://okta/oauth2/auslw2qcsmsUgzsqr0h7/v1/token`

* Extracting the access token. A handy util is `jq` which parses json payloads. Using it the access token may be extracted from the payload, something like:

`ACCESS_TOKEN=curl -s -X POST \
  --data "grant_type=password&scope=openid profile&username=user@example.com&password=password&client_id=client&client_secret=secret" \
  https://okta/oauth2/auslw2qcsmsUgzsqr0h7/v1/token | jq -r '.access_token'`

The resulting environment variable may be used in subsequent calls. Where the samples have `{access_token}` substitute `${ACCESS_TOKEN}`.

#### Test Access Token - OpenAM
Although not needed during normal operations, this call can be used to check an access token.
* Host: OpenAM
* URL: `/auth/oauth2/tokeninfo`
* Method: `GET`
* URL Params:
  * `access_token={access_token}`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "sbacUUID": "599758d9e4b0fbecbf5fc586",
      "mail": "test@example.com",
      "sn": "Test",
      "scope": [
         "sbacUUID",
         "mail",
         "sn",
         "cn",
         "sbacTenancyChain",
         "givenName"
      ],
      "grant_type": "password",
      "cn": "Test",
      "realm": "/sbac",
      "sbacTenancyChain": "|CA|ASMTDATALOAD|STATE|1000|ART_DL|||CA|CALIFORNIA|||||||||",
      "token_type": "Bearer",
      "expires_in": 35930,
      "givenName": "Test",
      "access_token": "20b55fc2-1b84-4412-8149-88cfa622db01"
    }
    ```
* Error Response:
  * Code: 400 (Bad Request)
  * Content (specific error and description will vary):
    ```json
    {
      "error": "invalid_request",
      "error_description": "Access Token not valid"
    }
    ```
* Sample Call (curl):

`curl https://openam/auth/oauth2/tokeninfo?access_token=20b55fc2-1b84-4412-8149-88cfa622db01`

#### Test Access Token - Okta
Although not needed during normal operations, this curl call can be used to check an access token:

`curl -X POST
  --data 'client_id=<client-id>&client_secret=<client-secret>'
  'https://smarterbalanced.oktapreview.com/oauth2/auslw2qcsmsUgzsqr0h7/v1/introspect?token=<full-text-of-access-token>&token_type_hint=access_token'`

* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "active": true,
      "scope": "profile openid",
      "username": "test@example.com",
      "exp": 1566865091,
      "iat": 1566861491,
      "sub": "test@example.com",
      "aud": "api://staging",
      "iss": "https://smarterbalanced.oktapreview.com/oauth2/auslw2qcsmsUgzsqr0h7",
      "token_type": "Bearer",
      "client_id": "<client-id>",
      "uid": "00ukx23x7uJn3O4980h7",
      "sbacTenancyChain": [
        "|CA|ASMTDATALOAD|STATE|1000|ART_DL|||CA|CALIFORNIA|||||||||",
        "|CA|Administrator|STATE|1000|ART_DL|||CA|CALIFORNIA|||||||||",
        "|CA|State Coordinator|STATE|1000|ART_DL|||CA|CALIFORNIA|||||||||"
      ],
      "mail": "test@example.com",
      "sbacUUID": "59946ee4e4b031dfb7d388ca"
    }
    ```

* Error Response:
  * Code: 400 (Bad Request)
  * Content (specific error and description will vary):
    ```json
    {
      "error": "invalid_client",
      "error_description": "The client secret supplied for a confidential client is invalid."
    }
    ```

### Import Endpoints
The import service provides the end-points for submitting data to the system. All end-points require a valid token, the examples use `{access_token}` as a
placeholder. The token must provide appropriate permissions. For most content, this is the `ASMTDATALOAD` role at the client or state level but refer to
individual content end-point documentation for specific permissions. There is one diagnostic end-point provided that helps debug permissions issues:

#### Get User
This end-point may be used to get the credentials of the current user. This is provided for diagnostic purposes.

* Host: import service
* URL: `/imports/user`
* Method: `GET`
* Params: none
* Headers:
  * `Authorization: Bearer {access_token}`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "password": "N/A",
      "username": "dwtest@example.com",
      "authorities": [
        {
          "authority": "ROLE_ASMTDATALOAD"
        }
      ],
      "accountNonExpired": true,
      "accountNonLocked": true,
      "credentialsNonExpired": true,
      "enabled": true,
      "tenancyChain": {
        "grants": [
          {
            "level": "CLIENT",
            "name": "",
            "id": "SBAC",
            "role": "ASMTDATALOAD",
            "clientId": "SBAC",
            "stateId": "",
            "entityId": "SBAC"
          }
        ],
      "empty": false
      }
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide RDW authentication.
* Sample Call (curl):

`curl --header "Authorization:Bearer {access_token}" https://import-service/imports/user`

The import requests are processed and migrated to the reporting data mart. Import payloads are hashed and duplicate content is detected, returning any previous
import request for the given content. Thus, for most content, submitting a payload a second time will safely no-op and return the current status of the previous
import. The one exception to this rule is for Student Groups: because other factors such as permissions and organizations affect the processing of groups,
duplicate import requests are reprocessed.

All data submissions result in an import being created. Thus a POST to `/exams/imports` will create an `import` resource which can be accessed at `/imports/{id}`
(note that `exams` is not in the path). These are the end-points for querying imports.

#### Get Import Request
This end-point may be used to get the current status of an import request.

* Host: import service
* URL: `/imports/{id}`
* Method: `GET`
* Params: none
* Headers:
  * `Authorization: Bearer {access_token}`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "id": 19529,
      "content": "EXAM",
      "contentType": "application/xml",
      "digest": "5899C64887FE25BC1F015FD7C26476E4",
      "status": "BAD_DATA",
      "creator": "user@example.com",
      "created": "2017-05-08T19:45:20.476Z",
      "message": "{\"elementName\":\"Sex\",\"value\":\"M\",\"error\":\"unknown gender name [M]\"},
      {\"elementName\":\"GradeLevelWhenAssessed\",\"value\":\"SIXTHGRADE\",\"error\":\"unknown grade code [SIXTHGRADE]\"}",
      "_links": {
        "self": {
          "href": "http://import-service/imports/19529"
        },
        "payload": {
          "href": "http://import-service/imports/19529/payload"
        },
        "payload-properties": {
          "href": "http://import-service/imports/19529/payload/properties"
        }  
      }
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `ASTMDATALOAD` role.
  * Code: 404 (Not Found) if no import with the given id exists.
* Sample Call (curl):

`curl --header "Authorization:Bearer {access_token}" https://import-service/imports/19529`

#### Get Import Payload
This end-point may be used to get the payload for an import request.

* Host: import service
* URL: `/imports/{id}/payload`
* Method: `GET`
* Params: none
* Headers:
  * `Authorization: Bearer {access_token}`
* Success Response:
  * Code: 200 (OK)
  * Content: File attachment
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `ASTMDATALOAD` role.
  * Code: 404 (Not Found) if no import with the given id exists.
* Sample Call (curl):

`curl --header "Authorization:Bearer {access_token}" https://import-service/imports/19529/payload`

#### Get Import Payload Properties
This end-point may be used to get the payload properties for an import request. This can be useful because not all the properties for an import are stored in the
data warehouse, some are archived only with the payload.

* Host: import service
* URL: `/imports/{id}/payload/properties`
* Method: `GET`
* Params: none
* Headers:
  * `Authorization: Bearer {access_token}`
* Success Response:
  * Code: 200 (OK)
  * Content:
    ```json
    {
      "Content-Type": "application/xml",
      "filename": "test.xml",
      "batch": "CAF4901EA92C0A77A6DB6C37E8582852",
      "tenancy-chain": "|SBAC|ASMTDATALOAD|CLIENT|SBAC||||||||||||||",
      "username": "user@example.com"
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `ASTMDATALOAD` role.
  * Code: 404 (Not Found) if no import with the given id exists.
* Sample Call (curl):

`curl --header "Authorization:Bearer {access_token}" https://import-service/imports/19529/payload/properties`


### Exam Endpoints
End-points for submitting exams aka test results.

These require the `ASMTDATALOAD` role.

#### Create Exam Import Request
Accepts payloads in Test Result Transmission ([TRT][3]) format, creating new exam import requests. Based on the data in the TRT this may result in an exam being
created, updated or deleted. Exams are identified using the opportunity id and assessment id. If an incoming exam is not found in the system one will be added. If
a matching exam already exists, it will be updated or, if the new status is `reset`, it will be deleted.

There are two ways of posting exam content: with a raw body of type `application/xml` or form-data (file upload).

* Host: import service
* URL: `/exams/imports`
* Method: `POST`
* URL Params: any URL param will be preserved as properties for the upload, well-known params:
  * `batch=<batchtag>`
  * `filename=<originalFileName>`  (not needed for form-data)
* Headers:
  * `Authorization: Bearer {access_token}`
* Headers (raw body):
  * `Content-Type:application/xml`
* Body (raw body): TRT
* Headers (form-data):
  * `Content-Type:multipart/form-data`
* Form Data (form-data):
  * `file` - the upload file
  * `filename=<filename>`
  * any other form data will be preserved as properties for the upload
* Success Response:
  * Code: 202 (Accepted)
  * Content:
    ```json
    {
      "id": 19529,
      "content": "EXAM",
      "contentType": "application/xml",
      "digest": "5899C64887FE25BC1F015FD7C26476E4",
      "status": "ACCEPTED",
      "creator": "user@example.com",
      "created": "2017-05-08T19:45:20.476Z",
      "_links": {
        "self": {
          "href": "http://import-service/imports/19529"
        }
      }
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `ASTMDATALOAD` role.
* Sample Call (raw body):

`curl -X POST --header "Authorization:Bearer {access_token}" --header "Content-Type:application/xml"
  --data-binary "<TDSReport><Test..." https://import-service/exams/imports?batch=WinterICA&filename=winterICA.1.xml`

* Sample Call (form-data):

`curl -X POST --header "Authorization:Bearer {access_token}" -F file=@winterICA.1.xml -F batch=WinterICA https://import-service/exams/imports`


### Groups Endpoints
End-point for submitting student group data in the Smarter Balanced Student Group CSV format.

These require the `GROUP_ADMIN` permission for the schools involved.

#### Create Groups Import Request
Accepts payloads in the Smarter Balanced Student Group CSV format. This end point can be used to import new groups or update existing ones.

There are two ways of posting content: with a raw body of type `application/csv` or form-data file upload in CSV format.

* Host: import service
* URL: `/groups/imports`
* Method: `POST`
* URL Params: any URL param will be preserved as properties for the upload, well-known params:
  * `filename=<originalFileName>`  (not needed for form-data)
* Headers:
  * `Authorization: Bearer {access_token}`
* Headers (raw body):
  * `Content-Type:application/csv`
* Body (raw body): Smarter Balanced Student Group CSV
* Headers (form-data):
  * `Content-Type:multipart/form-data`
* Form Data (form-data):
  * `file` - the upload file
  * `filename=<filename>`
  * any other form data will be preserved as properties for the upload
* Success Response:
  * Code: 202 (Accepted)
  * Content:
    ```json
    {
        "id": 9,
        "content": "GROUPS",
        "contentType": "application/csv",
        "digest": "5EA9607434903493BD274611FA817304",
        "status": "PROCESSED",
        "creator": "dwtest@example.com",
        "created": "2018-02-08T22:29:47.120343Z",
        "updated": "2018-02-08T22:29:47.120343Z",
        "_links": {
            "self": {
                "href": "http://localhost:8080/imports/9"
            },
            "payload": {
                "href": "http://localhost:8080/imports/9/payload"
            },
            "payload-properties": {
                "href": "http://localhost:8080/imports/9/payload/properties"
            }
        }
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `GROUP_WRITE` permission.
* Sample Call (raw body):

`curl -X POST --header "Authorization:Bearer {access_token}" --header "Content-Type:application/csv"
  --data-binary "group_name,school_natural_id,..." https://import-service/groups/imports?filename=mygroups.csv`

* Sample Call (form-data):

`curl -X POST --header "Authorization:Bearer {access_token}" -F "file=@mygroups.csv;type=text/csv" https://import-service/groups/imports`

### Norms Endpoints
End-point for submitting norms percentile tables in the Smarter Balanced [Norms CSV format](Norms.md). This end point can be used to import new norms or update
existing ones. When norms are created, the following data elements are required and comprise the unique identifier for the norms percentile table:
* assessment_id : natural id of an assessment that must already be loaded
* start_date : inclusive start date of exam completion
* end_date : inclusive end date of exam completion

When an existing norms percentile table matches the above three data elements on import it is updated.

These require the `ASMTDATALOAD` role.

#### Create Norms Import Request
Accepts payloads in the Smarter Balanced [Norms CSV format](Norms.md).

There are two ways of posting content: with a raw body of type `application/csv` or form-data file upload in CSV format.

* Host: import service
* URL: `/norms/imports`
* Method: `POST`
* URL Params: any URL param will be preserved as properties for the upload, well-known params:
  * `filename=<originalFileName>`  (not needed for form-data)
* Headers:
  * `Authorization: Bearer {access_token}`
* Headers (raw body):
  * `Content-Type:application/csv`
* Body (raw body): Smarter Balanced [Norms CSV format](Norms.md)
* Headers (form-data):
  * `Content-Type:multipart/form-data`
* Form Data (form-data):
  * `file` - the upload file
  * `filename=<filename>`
  * any other form data will be preserved as properties for the upload
* Success Response:
  * Code: 202 (Accepted)
  * Content:
    ```json
    {
        "id": 9,
        "content": "NORMS",
        "contentType": "application/csv",
        "digest": "5EA9607434903493BD274611FA817304",
        "status": "PROCESSED",
        "creator": "dwtest@example.com",
        "created": "2018-02-08T22:29:47.120343Z",
        "updated": "2018-02-08T22:29:47.120343Z",
        "message": "Percentiles created: 0 updated: 2",
        "_links": {
            "self": {
                "href": "http://localhost:8080/imports/9"
            },
            "payload": {
                "href": "http://localhost:8080/imports/9/payload"
            },
            "payload-properties": {
                "href": "http://localhost:8080/imports/9/payload/properties"
            }
        }
    }
    ```
* Error Response:
  * Code: 401 (Unauthorized) if token is missing or invalid.
  * Code: 403 (Forbidden) if token doesn't provide the `ASTMDATALOAD` role.
* Sample Call (raw body):

`curl -X POST --header "Authorization:Bearer {access_token}" --header "Content-Type:application/csv"
  --data-binary "assessment_id,start_date,..." https://import-service/norms/imports?filename=2017-norms.csv`

* Sample Call (form-data):

`curl -X POST --header "Authorization:Bearer {access_token}" -F "file=@2017-norms.csv;type=text/csv" https://import-service/norms/imports`


[1]: https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html
[2]: http://www.smarterapp.org/documents/AccessibilityConfigurationFileSpecification.pdf
[3]: http://www.smarterapp.org/documents/TestResultsTransmissionFormat.pdf
