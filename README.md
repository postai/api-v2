### API v2.0 Introduction

Welcome to the DigitalOcean API documentation.

The DigitalOcean API allows you to manage Droplets and resources within the DigitalOcean cloud in a simple, programmatic way using conventional HTTP requests.  The endpoints are intuitive and powerful, allowing you to easily make calls to retrieve information or to execute actions.

All of the functionality that you are familiar with in the DigitalOcean control panel is also available through the API, allowing you to script the complex actions that your situation requires.

The API documentation will start with a general overview about the design and technology that has been implemented, followed by reference information about specific endpoints.

### HTML Requests

The DigitalOcean API is fully [RESTful]("http://en.wikipedia.org/wiki/Representational_state_transfer").  Users can access the resources provided by the API by using standard HTML methods.

Any tool that is fluent in HTTP can communicate with the API simply by requesting the correct URI.  Requests should be made using the HTTPS protocol so that traffic is encrypted.  The interface responds to different methods depending on the action required.

<table class="pure-table pure-table-horizontal">
  <thead>
      <tr>
          <th>Method</th>
          <th>Usage</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td>GET</td>
          <td>
              <p>For simple retrieval of information about your account, Droplets, or environment, you should use the <strong>GET</strong> method.  The information you request will be returned to you as a JSON object.</p>

              <p>The attributes defined by the JSON object can be used to form additional requests.  Any request using the GET method is read-only and will not affect any of the objects you are querying.</p>
          </td>
      </tr>
      <tr>
          <td>DELETE</td>
          <td>
              <p>To destroy a resource and remove it from your account and environment, the <strong>DELETE</strong> method should be used.  This will remove the specified object if it is found.  If it is not found, the operation will return a response indicating that the object was not found.</p>

              <p>This <a href="http://en.wikipedia.org/wiki/Idempotent#Computer_science_meaning">idempotency</a> means that you do not have to check for a resource's availability prior to issuing a delete command, the final state will be the same regardless of its existence.</p>
          </td>
      </tr>
      <tr>
          <td>PUT</td>
          <td>
              <p>To update the information about a resource in your account, the <strong>PUT</strong> method is available.</p>

              <p>Like the DELETE Method, the PUT method is idempotent.  It sets the state of the target using the provided values, regardless of their current values.  Requests using the PUT method do not need to check the current attributes of the object.</p>
          </td>
      </tr>
      <tr>
          <td>POST</td>
          <td>
              <p>To create a new object, your request should specify the <strong>POST</strong> method.</p>

              <p>The POST request includes all of the attributes necessary to create a new object.  When you wish to create a new object, send a POST request to the target endpoint.</p>
          </td>
      </tr>
      <tr>
          <td>HEAD</td>
          <td>
              <p>Finally, to retrieve metadata information, you should use the <strong>HEAD</strong> method to get the headers.  This returns only the header of what would be returned with an associated GET request.</p>

              <p>Response headers contain some useful information about your API access and the results that are available for your request.</p>

              <p>For instance, the headers contain your current rate-limit value and the amount of time available until the limit resets.  It also contains metrics about the total number of objects found, pagination information, and the total content length.</p>
          </td>
      </tr>
  </tbody>
</table>


### HTML Statuses

Along with the HTML methods that the API responds to, it will also return standard HTML statuses, including error codes.

In the event of a problem, the status will contain the error code, while the body of the response will usually contain additional information about the problem that was encountered.

In general, if the status returned is in the 200 range, it indicates that the request was fulfilled successfully and that no error was encountered.

Return codes in the 400 range typically indicate that there was an issue with the request that was sent.  Among other things, this could mean that you did not authenticate correctly, that you are requesting an action that you do not have authorization for, that the object you are requesting does not exist, or that your request is malformed.

If you receive a status in the 500 range, this generally indicates a server-side problem.  This means that we are having an issue on our end and cannot fulfill your request currently.

#### EXAMPLE ERROR RESPONSE


    HTTP/1.1 403 Forbidden

    {
      "error":       "forbidden",
      "description": "You do not have access for the attempted action."
    }

### Curl Examples

Throughout this document, some example API requests will be given using the `curl` command.  This will allow us to demonstrate the various endpoints in a simple, textual format.

The names of account-specific resources (like Droplet IDs, for instance) will be represented by variables.  For instance, a Droplet ID may be represented by a variable called `$DROPLET_ID`  You can set the associated variables in your environment if you wish to use the examples without modification.

If you are working from the command line, the previously mentioned environmental variable could be set by setting and exporting the variables, as we show in the example.

If you are following along, make sure you use a Droplet ID that you control for so that your commands will execute correctly.

#### Set and Export a Variable</h4>

    export DROPLET_ID=1111111

### OAuth Authentication

In order to interact with the DigitalOcean API, you or your application must authenticate.

The DigitalOcean API handles this through OAuth, an open standard for authorization.  OAuth allows you to delegate access to your account in full or in read-only mode.

You can generate an OAuth token by visiting the [Apps & API]("https://cloud.digitalocean.com/settings/applications") section of the DigitalOcean control panel for your account.

An OAuth token functions as a complete authentication request.  In effect, it acts as a substitute for a username and password pair.

Because of this, it is absolutely **essential** that you keep your OAuth tokens secure.  In fact, upon generation, the web interface will only display each token a single time in order to prevent the token from being compromised.

#### How to Authenticate with OAuth

There are two separate ways to authenticate using OAuth.
      
The first option is to send a bearer authorization header with your request.  This is the preferred method of authenticating because it completes the authorization request in the header portion, away from the actual request.

You can also authenticate using basic authentication.  The normal way to do this with a tool like **curl** is to use the **-u** flag which is used for passing authentication information.
        
You then send the username and password combination delimited by a colon character.  We only have an OAuth token, so use the OAuth token as the username and leave the password slot blank.

This is effectively the same as embedding the authentication information within the URI itself.

#### Authenticate with a Bearer Authorization Header

    curl -X $HTTP_METHOD -H "Authorization: Bearer $ACCESS_TOKEN" "https://api.digitalocean.com/v2/$OBJECT"

#### Authenticate with Basic Authentication

    curl -X $HTTP_METHOD -u "$ACCESS_TOKEN:" "https://api.digitalocean.com/v2/$OBJECT"

### Parameters

There are two different ways to pass parameters in a request with the API.

The best way to pass parameters is as a JSON object containing the appropriate attribute names and values as key-value pairs.  When you use this format, you should specify that you are sending a JSON object in the header.

This is done by setting the `Content-Type` header to `application/json`.  This ensures that your request is interpreted correctly.

Another way of passing parameters is using standard query attributes.

Using this format, you would pass the attributes within the URI itself.  Tools like `curl` can take parameters and value as arguments and create the appropriate URI.

With `curl` this is done using the `-F` flag and then passing the key and value as an argument.  The argument should take the form of a quoted string with the attribute being set to a value with an equal sign.

You could also use a standard query string if that would be easier in our application.  In this case, the parameters would be embedded into the URI itself by appending a `?` to the end of the URI and then setting each attribute with an equal sign.  Attributes can be separated with a `&`.

#### Pass Parameters as a JSON Object

    curl -H "Authorization: Bearer $AUTH_TOKEN" \
        -H "Content-Type: application/json" \
        -d '{"name": "example.com", "ip_address": "127.0.0.1"}' \
        -X POST "https://api.digitalocean.com/v2/domains"

#### Pass Parameters as URI Components

    curl -H "Authorization: Bearer $AUTH_TOKEN" \
        -F "name=example.com" -F "ip_address=127.0.0.1" \
        -X POST "https://api.digitalocean.com/v2/domains"

#### Pass Parameters as a Query String

    curl -H "Authorization: Bearer $AUTH_TOKEN" \
         -X POST \
         "https://api.digitalocean.com/v2/domains?name=example.com&ip_address=127.0.0.1"
# Domain Records


Domain records are the DNS records for a domain.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>A unique identifier for each domain record.</td>
      <td><code>32</code></td>
    </tr>
    <tr>
      <td><strong>type</strong></td>
      <td><em>string</em></td>
      <td>The type of the DNS record (ex: A, CNAME, TXT, ...).</td>
      <td><code>CNAME</code></td>
    </tr>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>The name to use for the DNS record.</td>
      <td><code>subdomain</code></td>
    </tr>
    <tr>
      <td><strong>data</strong></td>
      <td><em>string</em></td>
      <td>The value to use for the DNS record.</td>
      <td><code>@</code></td>
    </tr>
    <tr>
      <td><strong>priority</strong></td>
      <td><em>nullable integer</em></td>
      <td>The priority for SRV and MX records.</td>
      <td><code>100</code></td>
    </tr>
    <tr>
      <td><strong>port</strong></td>
      <td><em>nullable integer</em></td>
      <td>The port for SRV records.</td>
      <td><code>12345</code></td>
    </tr>
    <tr>
      <td><strong>weight</strong></td>
      <td><em>nullable integer</em></td>
      <td>The weight for SRV records.</td>
      <td><code>100</code></td>
    </tr>
  </tbody>
</table>

## Domain Records Collection [/v2/domains/{domain_name}/records]

### Domain Records List all Domain Records [GET]

<p>To get a listing of all records configured for a domain, send a GET request to <code>/v2/domains/$DOMAIN_NAME/records</code>.</p>

<p>The response will be an array of domain record objects, each of which contains the standard domain record attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>The unique id for the individual record.</td>
        </tr>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>The DNS record type (A, MX, CNAME, etc).</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The host name, alias, or service being defined by the record.  See the [domain record] object to find out more.</td>
        </tr>
        <tr>
            <td>data</td>
            <td>string</td>
            <td>Variable data depending on record type.  See the [domain record] object for more detail on each record type.</td>
        </tr>
        <tr>
            <td>priority</td>
            <td>nullable Integer</td>
            <td>The priority of the host (for SRV and MX records. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>port</td>
            <td>nullable Integer</td>
            <td>The port that the service is accessible on (for SRV records only. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>weight</td>
            <td>nullable Integer</td>
            <td>The weight of records with the same priority (for SRV records only.  <code>null</code> otherwise).</td>
        </tr>
    </tbody>
</table>


<p>For attributes that are not used by a specific record type, the value of <code>null</code> will be returned.  For instance, all records other than SRV will have <code>null</code> for the <code>priority</code>, <code>weight</code>, and <code>port</code> attributes.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com/records" -X GET \
	-H "Authorization: Bearer 01acf89155eba2f2ed5919009516ae5c9158ce0e3f77e9be8d2fb8e9cbc285ca"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 01acf89155eba2f2ed5919009516ae5c9158ce0e3f77e9be8d2fb8e9cbc285ca
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354150
Content-Length: 861
      ```

  
    - Body

      ```json
      {
  "domain_records": [
    {
      "id": 1,
      "type": "A",
      "name": "@",
      "data": "8.8.8.8",
      "priority": null,
      "port": null,
      "weight": null
    },
    {
      "id": 2,
      "type": "NS",
      "name": null,
      "data": "NS1.DIGITALOCEAN.COM.",
      "priority": null,
      "port": null,
      "weight": null
    },
    {
      "id": 3,
      "type": "NS",
      "name": null,
      "data": "NS2.DIGITALOCEAN.COM.",
      "priority": null,
      "port": null,
      "weight": null
    },
    {
      "id": 4,
      "type": "NS",
      "name": null,
      "data": "NS3.DIGITALOCEAN.COM.",
      "priority": null,
      "port": null,
      "weight": null
    },
    {
      "id": 5,
      "type": "CNAME",
      "name": "example",
      "data": "@",
      "priority": null,
      "port": null,
      "weight": null
    }
  ]
}
      ```
  

### Domain Records Create a new Domain Record [POST]

<p>To create a new record to a domain, send a POST request to <code>/v2/domains/$DOMAIN_NAME/records</code>. </p>

<p>The request must include all of the required fields for the [domain record type] being added:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required For</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>The record type (A, MX, CNAME, etc).</td>
            <td>All records</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The host name, alias, or service being defined by the record.</td>
            <td>A, AAAA, CNAME, TXT, SRV</td>
        </tr>
        <tr>
            <td>data</td>
            <td>string</td>
            <td>Variable data depending on record type.  See the [Domain Records]() section for more detail on each record type.</td>
            <td>A, AAAA, CNAME, MX, TXT, SRV, NS</td>
        </tr>
        <tr>
            <td>priority</td>
            <td>nullable integer</td>
            <td>The priority of the host (for SRV and MX records. <code>null</code> otherwise).</td>
            <td>MX, SRV</td>
        </tr>
        <tr>
            <td>port</td>
            <td>nullable integer</td>
            <td>The port that the service is accessible on (for SRV records only. <code>null</code> otherwise).</td>
            <td>SRV</td>
        </tr>
        <tr>
            <td>weight</td>
            <td>nullable integer</td>
            <td>The weight of records with the same priority (for SRV records only.  <code>null</code> otherwise).</td>
            <td>SRV</td>
        </tr>
    </tbody>
</table>

<p>The response body will be an object of the new record.  Attributes that are not applicable for the record type will be set to <code>null</code>.  An <code>id</code> attribute is generated for each record as part of the object.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>The unique id for the individual record.</td>
        </tr>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>The DNS record type (A, MX, CNAME, etc).</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The host name, alias, or service being defined by the record.  See the [domain record] object to find out more.</td>
        </tr>
        <tr>
            <td>data</td>
            <td>string</td>
            <td>Variable data depending on record type.  See the [domain record] object for more detail on each record type.</td>
        </tr>
        <tr>
            <td>priority</td>
            <td>nullable Integer</td>
            <td>The priority of the host (for SRV and MX records. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>port</td>
            <td>nullable Integer</td>
            <td>The port that the service is accessible on (for SRV records only. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>weight</td>
            <td>nullable Integer</td>
            <td>The weight of records with the same priority (for SRV records only.  <code>null</code> otherwise).</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>name of the DNS record</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>data</strong></td>
      <td>value of the DNS record</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>type</strong></td>
      <td>type of the DNS record (ex: A, CNAME, TXT, ...)</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>priority</strong></td>
      <td>priority for SRV records</td>
      <td>false</td>
    </tr>
  
    <tr>
      <td><strong>port</strong></td>
      <td>port for SRV records</td>
      <td>false</td>
    </tr>
  
    <tr>
      <td><strong>weight</strong></td>
      <td>weight for SRV records</td>
      <td>false</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com/records" -d '{
  "name": "subdomain",
  "data": "2001:db8::ff00:42:8329",
  "type": "AAAA"
}' -X POST \
	-H "Authorization: Bearer 40c1f9ce04234a8fcc7ba3d216a69615b17c1245836c03bf27cc1182b29e225c" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 40c1f9ce04234a8fcc7ba3d216a69615b17c1245836c03bf27cc1182b29e225c
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "subdomain",
  "data": "2001:db8::ff00:42:8329",
  "type": "AAAA"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354150
Content-Length: 185
      ```

  
    - Body

      ```json
      {
  "domain_record": {
    "id": 11,
    "type": "AAAA",
    "name": "subdomain",
    "data": "2001:db8::ff00:42:8329",
    "priority": null,
    "port": null,
    "weight": null
  }
}
      ```
  

## Domain Records Member [/v2/domains/{domain_name}/records/{record_id}]

### Domain Records Update a Domain Record [PUT]

<p>To update an existing record, send a PUT request to <code>/v2/domains/$DOMAIN_NAME/records/$RECORD_ID</code>.  Set the "name" attribute to the new for the record.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>Set this to the new name you want for your record.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a domain record object which contains the standard domain record attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>The unique id for the record.</td>
        </tr>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>The DNS record type (A, MX, CNAME, etc).</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The host name, alias, or service being defined by the record.  See the [domain record] object for more info.</td>
        </tr>
        <tr>
            <td>data</td>
            <td>string</td>
            <td>Variable data depending on record type.  See the [domain records] object for more detail on each record type.</td>
        </tr>
        <tr>
            <td>priority</td>
            <td>nullable integer</td>
            <td>The priority of the host (for SRV and MX records. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>port</td>
            <td>nullable integer</td>
            <td>The port that the service is accessible on (for SRV records only. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>weight</td>
            <td>nullable integer</td>
            <td>The weight of records with the same priority (for SRV records only.  <code>null</code> otherwise).</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>New name</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com/records/16" -d '{
  "name": "new_name"
}' -X PUT \
	-H "Authorization: Bearer 819d6217c877b3e03a0de9572ecb6fa460264b23895d82ddf282c11b99e65586" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 819d6217c877b3e03a0de9572ecb6fa460264b23895d82ddf282c11b99e65586
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "new_name"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354150
Content-Length: 164
      ```

  
    - Body

      ```json
      {
  "domain_record": {
    "id": 16,
    "type": "CNAME",
    "name": "new_name",
    "data": "@",
    "priority": null,
    "port": null,
    "weight": null
  }
}
      ```
  

### Domain Records Delete a Domain Record [DELETE]

<p>To delete a record for a domain, send a DELETE request to <code>/v2/domains/$DOMAIN_NAME/records/$RECORD_ID</code>.</p>

<p>The record will be deleted and the response status will be a 204.  This indicates a successful request with no body returned.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com/records/21" -d '' -X DELETE \
	-H "Authorization: Bearer ffb4c6ef955821191f7124f5c2f4ab11ec173440515d55900b351b47558275b8" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer ffb4c6ef955821191f7124f5c2f4ab11ec173440515d55900b351b47558275b8
Content-Type: application/x-www-form-urlencoded
      ```

  

  - **Response**

    - Headers

      ```
      X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
      ```

  

### Domain Records Retrieve an existing Domain Record [GET]

<p>To retrieve a specific domain record, send a GET request to <code>/v2/domains/$DOMAIN_NAME/records/$RECORD_ID</code>.</p>

The request will be an object that contains all of the standard domain record attributes:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>The unique id for the record.</td>
        </tr>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>The DNS record type (A, MX, CNAME, etc).</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The host name, alias, or service being defined by the record.  See the [domain record] object for more info.</td>
        </tr>
        <tr>
            <td>data</td>
            <td>string</td>
            <td>Variable data depending on record type.  See the [domain records] object for more detail on each record type.</td>
        </tr>
        <tr>
            <td>priority</td>
            <td>nullable integer</td>
            <td>The priority of the host (for SRV and MX records. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>port</td>
            <td>nullable integer</td>
            <td>The port that the service is accessible on (for SRV records only. <code>null</code> otherwise).</td>
        </tr>
        <tr>
            <td>weight</td>
            <td>nullable integer</td>
            <td>The weight of records with the same priority. (for SRV records only.  <code>null</code> otherwise).</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com/records/26" -X GET \
	-H "Authorization: Bearer 2a58c526e2956dfd18df6be8be3976a216ae9fa70380d9032f685778187d45a7"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 2a58c526e2956dfd18df6be8be3976a216ae9fa70380d9032f685778187d45a7
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
Content-Length: 163
      ```

  
    - Body

      ```json
      {
  "domain_record": {
    "id": 26,
    "type": "CNAME",
    "name": "example",
    "data": "@",
    "priority": null,
    "port": null,
    "weight": null
  }
}
      ```
  
# Domains


Domains are managed domain names that DigitalOcean provides DNS for.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>The name of the domain itself.  This should follow the standard domain format of `domain.TLD`.  For instance, `example.com` is a valid domain name.</td>
      <td><code>example.com</code></td>
    </tr>
    <tr>
      <td><strong>ttl</strong></td>
      <td><em>integer</em></td>
      <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
      <td><code>1800</code></td>
    </tr>
    <tr>
      <td><strong>zone_file</strong></td>
      <td><em>string</em></td>
      <td>This attribute contains the complete contents of the zone file for the selected domain.  Individual domain record resources should be used to get more granular control over records.  However, this attribute can also be used to get information about the SOA record, which is created automatically and is not accessible as an individual record resource.</td>
      <td><code>$TTL\t600\n@\t\tIN\tSOA ...</code></td>
    </tr>
  </tbody>
</table>

## Domains Collection [/v2/domains]

### Domains List all Domains [GET]

<p>To retrieve a list of all of the domains in your account, send a GET request to <code>/v2/domains</code>.</p>

An array of Domain objects will be returned, each of which contain the standard domain attributes:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains" -X GET \
	-H "Authorization: Bearer a8512a578d778f8e95991b6d1cd55dfa7c604b0cca40e565a572e80017961312"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer a8512a578d778f8e95991b6d1cd55dfa7c604b0cca40e565a572e80017961312
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
Total: 1
Content-Length: 130
      ```

  
    - Body

      ```json
      {
  "domains": [
    {
      "name": "example.com",
      "ttl": 1800,
      "zone_file": "Example zone file text..."
    }
  ]
}
      ```
  

### Domains Create a new Domain [POST]

<p>To create a new domain, send a POST request to <code>/v2/domains</code>.  Set the "name" attribute to the domain name you are adding.  Set the "ip_address" attribute to the IP address you want to point the domain to.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The domain name to add to the DigitalOcean DNS management interface.  The name must be unique in DigitalOcean's DNS system.  The request will fail if the name has already been taken.</td>
            <td>true</td>
        </tr>
        <tr>
            <td>ip_address</td>
            <td>string</td>
            <td>This attribute contains the IP address you want the domain to point to.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will contain the standard attributes associated with a domain:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


<p>Keep in mind that, upon creation, the <code>zone_file</code> field will have a value of <code>null</code> until a zone file is generated and propagated through an automatic process on the DigitalOcean servers.</p>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>Name of the domain</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>ip_address</strong></td>
      <td>IP Address for the www entry.</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains" -d '{"name":"example.com","ip_address":"127.0.0.1"}' -X POST \
	-H "Authorization: Bearer 8de6e16d5c140efb45d32329e3383ad370797316cec4697d4133b666bde7e643" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 8de6e16d5c140efb45d32329e3383ad370797316cec4697d4133b666bde7e643
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "example.com",
  "ip_address": "127.0.0.1"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
Content-Length: 88
      ```

  
    - Body

      ```json
      {
  "domain": {
    "name": "example.com",
    "ttl": 1800,
    "zone_file": null
  }
}
      ```
  

## Domains Member [/v2/domains/{domain_name}]

### Domains Show [GET]

<p>To get details about a specific domain, send a GET request to <code>/v2/domains/$DOMAIN_NAME</code>. </p>

The response received will contain the standard attributes defined for a domain:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com" -X GET \
	-H "Authorization: Bearer 4318acab64f40d87a401584081df7c21481128d504e3d2deb10f56519b912032"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 4318acab64f40d87a401584081df7c21481128d504e3d2deb10f56519b912032
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
Content-Length: 111
      ```

  
    - Body

      ```json
      {
  "domain": {
    "name": "example.com",
    "ttl": 1800,
    "zone_file": "Example zone file text..."
  }
}
      ```
  

### Domains Delete a Domain [DELETE]

<p>To delete a domain, send a DELETE request to <code>/v2/domains/$DOMAIN_NAME</code>.</p>

<p>The domain will be removed from your account and a response status of 204 will be returned.  This indicates a successful request with no response body.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/domains/example.com" -d '' -X DELETE \
	-H "Authorization: Bearer 9666e16fb8670397f3eda4a755e2d3e81af0351475cd4dff465a676147b734ba" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 9666e16fb8670397f3eda4a755e2d3e81af0351475cd4dff465a676147b734ba
Content-Type: application/x-www-form-urlencoded
      ```

  

  - **Response**

    - Headers

      ```
      X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
      ```

  
# Droplet


Droplets are VMs in the DigitalOcean cloud.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>A unique identifier for each Droplet instance.  This is automatically generated upon Droplet creation.</td>
      <td><code>1234</code></td>
    </tr>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>The human-readable name set for the Droplet instance.</td>
      <td><code>my-droplet</code></td>
    </tr>
    <tr>
      <td><strong>region</strong></td>
      <td><em>object</em></td>
      <td>The region that the Droplet instance is deployed in.  WHen setting a region, the value should be the slug identifier for the region.  When you query a Droplet, teh entire region object will be returned.</td>
      <td><code>{"slug":"nyc1","name":"New York","sizes":["1024mb","512mb"],"available":true}</code></td>
    </tr>
    <tr>
      <td><strong>image</strong></td>
      <td><em>object</em></td>
      <td>The base image used to create the Droplet instance.  When setting an image, the value is set to the image id or slug.  When querying the Droplet, the entire image object will be returned.</td>
      <td><code>{"id":119192818,"name":"Ubuntu 13.04","distribution":"ubuntu","slug":null,"public":true,"regions":["nyc1"]}</code></td>
    </tr>
    <tr>
      <td><strong>size</strong></td>
      <td><em>object</em></td>
      <td>The size of the Droplet instance.  When setting a size, the value should be the slug identifier for a particular size.  When querying the Droplet, the entire size object will be returned.</td>
      <td><code>{"slug":"512mb","memory":512,"vcpus":1,"disk":20,"transfer":null,"price_monthly":"5.0","price_hourly":"0.00744","regions":["nyc1","sfo1","ams4"]}</code></td>
    </tr>
    <tr>
      <td><strong>networks</strong></td>
      <td><em>object</em></td>
      <td>The details of the network that are configured for the Droplet instance.  This is an object that contains keys for IPv4 and IPv6.  The value of each of these is an array that contains objects describing an individual IP resource allocated to the Droplet.  These will define attributes like the IP address, netmask, and gateway of the specific network depending on the type of network it is.</td>
      <td><code>{"v4":[{"ip_address":"127.0.0.2","netmask":"255.255.255.0","gateway":"127.0.0.1","type":"public"}],"v6":[]}</code></td>
    </tr>
    <tr>
      <td><strong>backups</strong></td>
      <td><em>array</em></td>
      <td>An array of any backups that have been taken of the Droplet instance.  Droplet backups are enabled at the time of the instance creation.  The array contains image objects representing each backup.</td>
      <td><code>[123, 456, 789]</code></td>
    </tr>
    <tr>
      <td><strong>snapshots</strong></td>
      <td><em>array</em></td>
      <td>An array of any snapshots created from the Droplet instance.  The array contains objects that represent each snapshot.  This will be in the form of an image object.</td>
      <td><code>[123, 456, 789]</code></td>
    </tr>
    <tr>
      <td><strong>locked</strong></td>
      <td><em>boolean</em></td>
      <td>A boolean value indicating whether the Droplet has been locked, preventing actions by users.</td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><strong>status</strong></td>
      <td><em>string</em></td>
      <td>A status string indicating the state of the Droplet instance.  This may be "new", "active", or "archive".</td>
      <td><code>online</code></td>
    </tr>
  </tbody>
</table>

## Droplet Collection [/v2/droplets]

### Droplet List all Droplets [GET]

<p>To list all Droplets in your account, send a GET request to <code>/v2/droplets</code>.</p>


<p>The response body will be an array containing objects representing each Droplet. These will have the standard Droplet attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet.  This is automatically generated upon Droplet creation.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>String</td>
            <td>A human-readable name set for the Droplet instance.</td>
        </tr>
        <tr>
            <td>region</td>
            <td>Object</td>
            <td>The region that the Droplet instance is deployed in.  The entire region object is returned.</td>
        </tr>
        <tr>
            <td>image</td>
            <td>Object</td>
            <td>The base image used to create the Droplet instance.  The entire image object is returned.</td>
        </tr>
        <tr>
            <td>size</td>
            <td>Object</td>
            <td>The size of the Droplet instance.  The entire size object is returned.</td>
        </tr>
        <tr>
            <td>networks</td>
            <td>Array</td>
            <td>The details of the networks configured for the Droplet.  This is an object containing arrays for each of the separate networks.  Each array defines the IP address and the network details (netmask, gateway, public or private, etc.) of the specific network.</td>
        </tr>
        <tr>
            <td>backups</td>
            <td>Array</td>
            <td>An array of any backups that have been taken of the Droplet.  The array contain backup objects.  These follow the conventions of an image object.</td>
        </tr>
        <tr>
            <td>snapshots</td>
            <td>Array</td>
            <td>An array of any snapshots created from the Droplet.  The array contains snapshot objects.  These follow the conventions of an image object.</td>
        </tr>
        <tr>
            <td>locked</td>
            <td>Boolean</td>
            <td>A boolean value indicating whether the Droplet has been locked, preventing actions by users.</td>
        </tr>
        <tr>
            <td>status</td>
            <td>String</td>
            <td>A string indicating the state of the Droplet instance.  This may be "new", "active", or "archive".</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets" -X GET \
	-H "Authorization: Bearer 2fd3240f51708df028f03e6075bb5ddcd21855e9ef2e24e928764a5e22d3e3d1"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 2fd3240f51708df028f03e6075bb5ddcd21855e9ef2e24e928764a5e22d3e3d1
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354145
Total: 1
Content-Length: 1531
      ```

  
    - Body

      ```json
      {
  "droplets": [
    {
      "id": 1,
      "name": "test.example.com",
      "region": {
        "slug": "nyc1",
        "name": "New York",
        "sizes": [
          "1024mb",
          "512mb"
        ],
        "available": true
      },
      "image": {
        "id": 119192817,
        "name": "Ubuntu 13.04",
        "distribution": "ubuntu",
        "slug": "ubuntu1304",
        "public": true,
        "regions": [
          "nyc1"
        ]
      },
      "size": {
        "slug": "512mb",
        "memory": 512,
        "vcpus": 1,
        "disk": 20,
        "transfer": null,
        "price_monthly": "5.0",
        "price_hourly": "0.00744",
        "regions": [
          "nyc1",
          "br1",
          "sfo1",
          "ams4"
        ]
      },
      "locked": false,
      "status": "active",
      "networks": {
        "v4": [
          {
            "ip_address": {
              "ip_address": "127.0.0.1",
              "netmask": "255.255.255.0",
              "gateway": "127.0.0.2",
              "type": "public"
            }
          }

        ],
        "v6": [
          {
            "ipv6_address": {
              "ip_address": "2400:6180:0000:00D0:0000:0000:0009:7001",
              "cidr": 124,
              "gateway": "2400:6180:0000:00D0:0000:0000:0009:7000",
              "type": "public"
            }
          }

        ]
      },
      "backup_ids": [
        119192818
      ],
      "snapshot_ids": [
        119192819
      ],
      "action_ids": [

      ]
    }
  ]
}
      ```
  

### Droplet Retrieve snapshots for a Droplet [GET]

Lists all of the droplet's snapshots

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/2/snapshots" -X GET \
	-H "Authorization: Bearer c8bcd3663afa7087f269b4715e62dcec55005598974cf508e312d9653f2ab683"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer c8bcd3663afa7087f269b4715e62dcec55005598974cf508e312d9653f2ab683
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354146
Total: 1
Content-Length: 207
      ```

  
    - Body

      ```json
      {
  "snapshots": [
    {
      "id": 119192820,
      "name": "Ubuntu 13.04",
      "distribution": "ubuntu",
      "slug": null,
      "public": false,
      "regions": [
        "nyc1"
      ]
    }
  ]
}
      ```
  

### Droplet Retrieve backups for a Droplet [GET]

Lists all of the droplet's backups

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/5/backups" -X GET \
	-H "Authorization: Bearer afe8045503f33f3c9a2a5a5958349006535a9d14fddb9a518688b46a194162a4"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer afe8045503f33f3c9a2a5a5958349006535a9d14fddb9a518688b46a194162a4
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354146
Total: 1
Content-Length: 205
      ```

  
    - Body

      ```json
      {
  "backups": [
    {
      "id": 119192823,
      "name": "Ubuntu 13.04",
      "distribution": "ubuntu",
      "slug": null,
      "public": false,
      "regions": [
        "nyc1"
      ]
    }
  ]
}
      ```
  

### Droplet Create a new Droplet [POST]

<p>To create a new Droplet, send a POST request to <code>/v2/droplets</code>.</p>

<p>The attribute values that must be set to successfully create a Droplet are:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>String</td>
            <td>The human-readable string you wish to use when displaying the Droplet name.  The name, if set to a domain name managed in the DigitalOcean DNS management system, will configure a PTR record for the Droplet.  The name set during creation will also determine the hostname for the Droplet in its internal configuration.</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>region</td>
            <td>String</td>
            <td>The unique slug identifier for the region that you wish to deploy in.</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>size</td>
            <td>String</td>
            <td>The unique slug identifier for the size that you wish to select for this Droplet.</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>image</td>
            <td>Integer (if using an image ID), or String (if using a public image slug)</td>
            <td>The image ID of a public or private image, or the unique slug identifier for a public image.  This image will be the base image for your Droplet.</td>
            <td>Yes</td>
        </tr> <tr>
            <td>ssh_keys</td>
            <td>Array</td>
            <td>An array containing the IDs or fingerprints of the SSH keys that you wish to embed in the Droplet's root account upon creation.</td>
            <td>No</td>
        </tr>
        <tr>
            <td>backups</td>
            <td>Boolean</td>
            <td>A boolean indicating whether automated backups should be enabled for the Droplet.  Automated backups can only be enabled when the Droplet is created.</td>
            <td>No</td>
        </tr>
        <tr>
            <td>ipv6</td>
            <td>Boolean</td>
            <td>A boolean indicating whether IPv6 is enabled on the Droplet.</td>
            <td>No</td>
        </tr>
        <tr>
            <td>private_networking</td>
            <td>Boolean</td>
            <td>A boolean indicating whether private networking is enabled for the Droplet.  Private networking is currently only available in certain regions.</td>
            <td>No</td>
        </tr>
    </tbody>
</table>

<p>A Droplet will be created using the provided information.  The response body will contain the standard attributes for your new Droplet:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet.  This is automatically generated upon Droplet creation.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>String</td>
            <td>A human-readable name set for the Droplet instance.</td>
        </tr>
        <tr>
            <td>region</td>
            <td>Object</td>
            <td>The region that the Droplet instance is deployed in.  The entire region object is returned.</td>
        </tr>
        <tr>
            <td>image</td>
            <td>Object</td>
            <td>The base image used to create the Droplet instance.  The entire image object is returned.</td>
        </tr>
        <tr>
            <td>size</td>
            <td>Object</td>
            <td>The size of the Droplet instance.  The entire size object is returned.</td>
        </tr>
        <tr>
            <td>networks</td>
            <td>Array</td>
            <td>The details of the networks configured for the Droplet.  This is an object containing arrays for each of the separate networks.  Each array defines the IP address and the network details (netmask, gateway, public or private, etc.) of the specific network.</td>
        </tr>
        <tr>
            <td>backups</td>
            <td>Array</td>
            <td>An array of any backups that have been taken of the Droplet.  The array contain backup objects.  These follow the conventions of an image object.</td>
        </tr>
        <tr>
            <td>snapshots</td>
            <td>Array</td>
            <td>An array of any snapshots created from the Droplet.  The array contains snapshot objects.  These follow the conventions of an image object.</td>
        </tr>
        <tr>
            <td>locked</td>
            <td>Boolean</td>
            <td>A boolean value indicating whether the Droplet has been locked, preventing actions by users.</td>
        </tr>
        <tr>
            <td>status</td>
            <td>String</td>
            <td>A string indicating the state of the Droplet instance.  This may be "new", "active", or "archive".</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>User assigned identifier</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>region</strong></td>
      <td>Slug of the desired Region</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>size</strong></td>
      <td>Slug of the desired Size</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>image</strong></td>
      <td>ID or Slug(public images only) of the desired Image</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>ssh_keys</strong></td>
      <td>An array of ssh key or fingerprints</td>
      <td>false</td>
    </tr>
  
    <tr>
      <td><strong>backups</strong></td>
      <td>Boolean field for enabling automatic backups</td>
      <td>false</td>
    </tr>
  
    <tr>
      <td><strong>ipv6</strong></td>
      <td>Boolean field for enabling ipv6 support</td>
      <td>false</td>
    </tr>
  
    <tr>
      <td><strong>private_networking</strong></td>
      <td>Boolean field for enabling private networking</td>
      <td>false</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets" -d '{
  "name": "My-Droplet",
  "region": "nyc1",
  "size": "512mb",
  "image": 119192824
}' -X POST \
	-H "Authorization: Bearer 7636830c83d51968bb68358476025ab6e259c4b72b0e404a91e9b230d80df89f" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 7636830c83d51968bb68358476025ab6e259c4b72b0e404a91e9b230d80df89f
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "My-Droplet",
  "region": "nyc1",
  "size": "512mb",
  "image": 119192824
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354146
Link: <http://example.org/v2/droplets/6/actions/2>; rel="monitor"
Content-Length: 844
      ```

  
    - Body

      ```json
      {
  "droplet": {
    "id": 6,
    "name": "My-Droplet",
    "region": {
      "slug": "nyc1",
      "name": "New York",
      "sizes": [
        "1024mb",
        "512mb"
      ],
      "available": true
    },
    "image": {
      "id": 119192824,
      "name": "Ubuntu 13.04",
      "distribution": "ubuntu",
      "slug": null,
      "public": true,
      "regions": [
        "nyc1"
      ]
    },
    "size": {
      "slug": "512mb",
      "memory": 512,
      "vcpus": 1,
      "disk": 20,
      "transfer": null,
      "price_monthly": "5.0",
      "price_hourly": "0.00744",
      "regions": [
        "nyc1",
        "br1",
        "sfo1",
        "ams4"
      ]
    },
    "locked": false,
    "status": "new",
    "networks": {
    },
    "backup_ids": [

    ],
    "snapshot_ids": [

    ],
    "action_ids": [
      2
    ]
  }
}
      ```
  

## Droplet Member [/v2/droplets/{droplet_id}]

### Droplet Delete a Droplet [DELETE]

<p>To delete a Droplet, send a DELETE request to <code>/v2/droplets/$DROPLET_ID</code>.</p>

<p>No response body will be sent back, but the response code will indicate success.  Specifically, the response code will be a 204, which means that the action was successful with no returned body data.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/3" -d '' -X DELETE \
	-H "Authorization: Bearer 1dcef1925963d4d12c2be260679f3db6d6f046b627995446ddd48018aee50af6" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 1dcef1925963d4d12c2be260679f3db6d6f046b627995446ddd48018aee50af6
Content-Type: application/x-www-form-urlencoded
      ```

  

  - **Response**

    - Headers

      ```
      X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354146
      ```

  

### Droplet Retrieve an existing Droplet by id [GET]

<p>To show an individual droplet, send a GET request to <code>/v2/droplets/$DROPLET_ID</code>.</p>

<p>The response will contain the Droplet's attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet.  This is automatically generated upon Droplet creation.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>String</td>
            <td>A human-readable name set for the Droplet instance.</td>
        </tr>
        <tr>
            <td>region</td>
            <td>Object</td>
            <td>The region that the Droplet instance is deployed in.  The entire region object is returned.</td>
        </tr>
        <tr>
            <td>image</td>
            <td>Object</td>
            <td>The base image used to create the Droplet instance.  The entire image object is returned.</td>
        </tr>
        <tr>
            <td>size</td>
            <td>Object</td>
            <td>The size of the Droplet instance.  The entire size object is returned.</td>
        </tr>
        <tr>
            <td>networks</td>
            <td>Array</td>
            <td>The details of the networks configured for the Droplet.  This is an object containing arrays for each of the separate networks.  Each array defines the IP address and the network details (netmask, gateway, public or private, etc.) of the specific network.</td>
        </tr>
        <tr>
            <td>backups</td>
            <td>Array</td>
            <td>An array of any backups that have been taken of the Droplet.  The array contain backup objects.  These follow the conventions of image objects. </td>
        </tr>
        <tr>
            <td>snapshots</td>
            <td>Array</td>
            <td>An array of any snapshots created from the Droplet.  The array contains snapshot objects.  These follow the conventions of image objects.</td>
        </tr>
        <tr>
            <td>locked</td>
            <td>Boolean</td>
            <td>A boolean value indicating whether the Droplet has been locked, preventing actions by users.</td>
        </tr>
        <tr>
            <td>status</td>
            <td>String</td>
            <td>A string indicating the state of the Droplet instance.  This may be "new", "active", or "archive".</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/4" -X GET \
	-H "Authorization: Bearer cb932fd2cce7090c533355ef1f99cfa56311e11b03cdf3daa95f9d17bfed9217"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer cb932fd2cce7090c533355ef1f99cfa56311e11b03cdf3daa95f9d17bfed9217
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354146
Content-Length: 1382
      ```

  
    - Body

      ```json
      {
  "droplet": {
    "id": 4,
    "name": "test.example.com",
    "region": {
      "slug": "nyc1",
      "name": "New York",
      "sizes": [
        "1024mb",
        "512mb"
      ],
      "available": true
    },
    "image": {
      "id": 119192817,
      "name": "Ubuntu 13.04",
      "distribution": "ubuntu",
      "slug": "ubuntu1304",
      "public": true,
      "regions": [
        "nyc1"
      ]
    },
    "size": {
      "slug": "512mb",
      "memory": 512,
      "vcpus": 1,
      "disk": 20,
      "transfer": null,
      "price_monthly": "5.0",
      "price_hourly": "0.00744",
      "regions": [
        "nyc1",
        "br1",
        "sfo1",
        "ams4"
      ]
    },
    "locked": false,
    "status": "active",
    "networks": {
      "v4": [
        {
          "ip_address": {
            "ip_address": "127.0.0.4",
            "netmask": "255.255.255.0",
            "gateway": "127.0.0.5",
            "type": "public"
          }
        }

      ],
      "v6": [
        {
          "ipv6_address": {
            "ip_address": "2400:6180:0000:00D0:0000:0000:0009:7004",
            "cidr": 124,
            "gateway": "2400:6180:0000:00D0:0000:0000:0009:7000",
            "type": "public"
          }
        }

      ]
    },
    "backup_ids": [
      119192821
    ],
    "snapshot_ids": [
      119192822
    ],
    "action_ids": [

    ]
  }
}
      ```
  
# Droplet Actions


Droplet actions are actions that can be executed on a droplet.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
      <td><code>1234</code></td>
    </tr>
    <tr>
      <td><strong>progress</strong></td>
      <td><em>string</em></td>
      <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
      <td><code>in-progress</code></td>
    </tr>
    <tr>
      <td><strong>type</strong></td>
      <td><em>string</em></td>
      <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
      <td><code>reboot</code></td>
    </tr>
    <tr>
      <td><strong>started_at</strong></td>
      <td><em>datetime</em></td>
      <td>The datetime containing the time that the action was initiated.</td>
      <td><code>2014-05-08T19:11:41Z</code></td>
    </tr>
  </tbody>
</table>

## Droplet Actions Collection [/v2/droplets/{droplet_id}/actions]

### Droplet Actions Restore a Droplet [POST]

<p>To restore a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>restore</code> and the "image" attribute to an image ID.</p>

<p>A Droplet restoration will rebuild an image using a backup image.  The image ID that is passed in must be a backup of the current Droplet instance.  The operation will leave any embedded SSH keys intact.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>restore</code>.</td>
            <td>true</td>
        </tr>
        <tr>
            <td>image</td>
            <td>integer</td>
            <td>The image ID of the backup image that you would like to restore.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will contain all of the standard Droplet action attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>restore</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>image</strong></td>
      <td>the desired image identifier</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/7/actions" -d '{"type":"restore","image":119192828}' -X POST \
	-H "Authorization: Bearer 4bfeaa57582a878d7f718a255302edf9afb9b0a9d46727da959ec7f34c4e0bd4" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 4bfeaa57582a878d7f718a255302edf9afb9b0a9d46727da959ec7f34c4e0bd4
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "restore",
  "image": 119192828
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354147
Content-Length: 127
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 5,
    "status": "in-progress",
    "type": "restore",
    "started_at": "2014-06-09T21:49:07Z"
  }
}
      ```
  

### Droplet Actions Resize a Droplet [POST]

<p>To resize a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>resize</code> and the "size" attribute to a sizes slug.</p>

<p>The Droplet must be powered off prior to resizing.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>resize</code>.</td>
            <td>true</td>
        </tr>
        <tr>
            <td>size</td>
            <td>string</td>
            <td>The size slug that you want to resize to.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>resize</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>size</strong></td>
      <td>the desired size slug</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/8/actions" -d '{"type":"resize","size":"1024mb"}' -X POST \
	-H "Authorization: Bearer a0a9089566e94ec20f8f5b4ff92ca0b6ce3e07c138396037ba36324f9d111dea" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer a0a9089566e94ec20f8f5b4ff92ca0b6ce3e07c138396037ba36324f9d111dea
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "resize",
  "size": "1024mb"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354147
Content-Length: 126
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 6,
    "status": "in-progress",
    "type": "resize",
    "started_at": "2014-06-09T21:49:07Z"
  }
}
      ```
  

### Droplet Actions Rename a Droplet [POST]

<p>To rename a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>rename</code> and the "name" attribute to the new name for the droplet.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>rename</code></td>
            <td>true</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The new name for the Droplet.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>rename</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>name</strong></td>
      <td>the desired name</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/9/actions" -d '{"type":"rename","name":"Droplet-Name"}' -X POST \
	-H "Authorization: Bearer e76d186a821ab77f0b9b425bea88132124051ea12d3f197a472c7fee1604ffc8" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer e76d186a821ab77f0b9b425bea88132124051ea12d3f197a472c7fee1604ffc8
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "rename",
  "name": "Droplet-Name"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 126
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 7,
    "status": "in-progress",
    "type": "rename",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Password Reset a Droplet [POST]

<p>To reset the password for a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>password_reset</code>.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>password_reset</code></td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>password_reset</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/10/actions" -d '{"type":"password_reset"}' -X POST \
	-H "Authorization: Bearer 6cd9602b825d235473a8ebb8845acb2927d971604fa48219d180296b7479b859" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 6cd9602b825d235473a8ebb8845acb2927d971604fa48219d180296b7479b859
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "password_reset"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 134
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 8,
    "status": "in-progress",
    "type": "password_reset",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Rebuild a Droplet [POST]

<p>To rebuild a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>rebuild</code> and the "image" attribute to an image ID or slug.</p>

<p>A rebuild action functions just like a new create.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>rebuild</code></td>
            <td>true</td>
        </tr>
        <tr>
            <td>image</td>
            <td>string if an image slug. integer if an image ID.</td>
            <td>An image slug or ID.  This represents the image that the Droplet will use as a base.</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>rebuild</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>image</strong></td>
      <td>the desired image identifier</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/11/actions" -d '{"type":"rebuild","image":119192833}' -X POST \
	-H "Authorization: Bearer 805bd8443c943eec0c18fd67a2ce66686f7e9b3801ff3c70bb039431f66cec78" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 805bd8443c943eec0c18fd67a2ce66686f7e9b3801ff3c70bb039431f66cec78
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "rebuild",
  "image": 119192833
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 127
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 9,
    "status": "in-progress",
    "type": "rebuild",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Reboot a Droplet [POST]

<p>To reboot a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>reboot</code>.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>reboot</code></td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>reboot</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/12/actions" -d '{"type":"reboot"}' -X POST \
	-H "Authorization: Bearer d40fad0a2fe3f5be2574da2050ff6810bb35307b5c3d78b52fed7e6f7cb5cd7d" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer d40fad0a2fe3f5be2574da2050ff6810bb35307b5c3d78b52fed7e6f7cb5cd7d
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "reboot"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 127
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 10,
    "status": "in-progress",
    "type": "reboot",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Shutdown a Droplet [POST]

<p>To shutdown a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>shutdown</code>.</p>

                <p>A shutdown action is an attempt to shutdown the Droplet in a graceful way, similar to using the <code>shutdown</code> command from the console.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>shutdown</code>.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>shutdown</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/13/actions" -d '{"type":"shutdown"}' -X POST \
	-H "Authorization: Bearer 77e9721dc854dddc5a50723f85e3893fc278f6469e6773cc5d82e5d0b6a817a7" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 77e9721dc854dddc5a50723f85e3893fc278f6469e6773cc5d82e5d0b6a817a7
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "shutdown"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 129
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 11,
    "status": "in-progress",
    "type": "shutdown",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Power On a Droplet [POST]

<p>To power on a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>power_on</code>.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>power_on</code></td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response should be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>power_on</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/14/actions" -d '{"type":"power_on"}' -X POST \
	-H "Authorization: Bearer 4d3ab9c4c1c3bae5f42226e2e81d45dadceb8c07ce8d5a491c735d3b6fbabb0a" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 4d3ab9c4c1c3bae5f42226e2e81d45dadceb8c07ce8d5a491c735d3b6fbabb0a
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "power_on"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354148
Content-Length: 129
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 12,
    "status": "in-progress",
    "type": "power_on",
    "started_at": "2014-06-09T21:49:08Z"
  }
}
      ```
  

### Droplet Actions Power Cycle a Droplet [POST]

<p>To power cycle a Droplet (power off and then back on), send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>power_cycle</code>.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>power_cycle</code></td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will be a Droplet actions object:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>power_cycle</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/16/actions" -d '{"type":"power_cycle"}' -X POST \
	-H "Authorization: Bearer c670ddadfc3328bee2e581fe3ef28d0989e42febdb1c6c179478cf40e986878e" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer c670ddadfc3328bee2e581fe3ef28d0989e42febdb1c6c179478cf40e986878e
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "power_cycle"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 132
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 14,
    "status": "in-progress",
    "type": "power_cycle",
    "started_at": "2014-06-09T21:49:09Z"
  }
}
      ```
  

### Droplet Actions Power Off a Droplet [POST]

<p>To power off a Droplet, send a POST request to <code>/v2/droplets/$DROPLET_ID/actions</code>.  Set the "type" attribute to <code>power_off</code>.</p>

<p>A <code>power_off</code> event is a hard shutdown and should only be used if the <code>shutdown</code> action is not successful.  It is similar to cutting the power on a server and could lead to complications.</p>

<p>The request should contain the following attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>powered_off</code></td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response should be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>power_off</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/17/actions" -d '{"type":"power_off"}' -X POST \
	-H "Authorization: Bearer b14b4259e5157e93abe6823ea74290b0af3f86582ce73a13ffd7da4334e718f9" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer b14b4259e5157e93abe6823ea74290b0af3f86582ce73a13ffd7da4334e718f9
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "power_off"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 130
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 15,
    "status": "in-progress",
    "type": "power_off",
    "started_at": "2014-06-09T21:49:09Z"
  }
}
      ```
  

## Droplet Actions Member [/v2/droplets/{droplet_id}/actions/{droplet_action_id}]

### Droplet Actions Retrieve a Droplet Action [GET]

<p>To retrieve a Droplet action, send a GET request to <code>/v2/droplets/$DROPLET_ID/actions/$ACTION_ID</code>.</p>

<p>A Droplet action object is an object containing the id, progress, type, and started_at attributes.  It can be used to track the completion of actions that were requested.</p>

<p>The response will be a standard Droplet action:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique identifier for each Droplet action event.  This is used to reference a specific action that was requested.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the action.  The value of this attribute will be "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>The type of action that the event is executing (reboot, power_off, etc.).</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>The datetime containing the time that the action was initiated.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/droplets/15/actions/13" -X GET \
	-H "Authorization: Bearer cdc51afce68d89d79f3e030a93370996e032d6c438b48c9c0b075b725bf595b0"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer cdc51afce68d89d79f3e030a93370996e032d6c438b48c9c0b075b725bf595b0
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 127
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 13,
    "status": "in-progress",
    "type": "create",
    "started_at": "2014-06-09T21:49:09Z"
  }
}
      ```
  
# Image Actions


Image actions are actions that can be executed on an image.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>A unique numeric ID that can be used to identify and reference an image action.</td>
      <td><code>1234</code></td>
    </tr>
    <tr>
      <td><strong>progress</strong></td>
      <td><em>string</em></td>
      <td>The current progress of the image action.  This will be either "in-progress", "completed", or "errored".</td>
      <td><code>in-progress</code></td>
    </tr>
    <tr>
      <td><strong>type</strong></td>
      <td><em>string</em></td>
      <td>This is the type of the image action that the JSON object represents.  For example, this could be "transfer" to represent the state of an image transfer action.</td>
      <td><code>image_transfer</code></td>
    </tr>
    <tr>
      <td><strong>started_at</strong></td>
      <td><em>datetime</em></td>
      <td>This represents the time that the image action was initiated.</td>
      <td><code>2014-05-08T19:11:41Z</code></td>
    </tr>
  </tbody>
</table>

## Image Actions Collection [/v2/images/{image_id}/actions]

### Image Actions Transfer an Image [POST]

<p>To transfer an image to another region, send a POST request to <code>/v2/images/$IMAGE_ID/actions</code>.  Set the "type" attribute to "transfer" and set "region" attribute to the slug identifier of the region you wish to transfer to.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>type</td>
            <td>string</td>
            <td>Must be <code>resize</code></td>
            <td>true</td>
        </tr>
        <tr>
            <td>region</td>
            <td>string</td>
            <td>The sizes slug that represents the resize target.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

The response will contain the image action:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>Integer</td>
            <td>A unique numeric ID that can be used to identify and reference an image action event.</td>
        </tr>
        <tr>
            <td>progress</td>
            <td>String</td>
            <td>The current progress of the image action.  This will be either "in-progress", "completed", or "errored".</td>
        </tr>
        <tr>
            <td>type</td>
            <td>String</td>
            <td>This is the type of the image action that the JSON object represents.</td>
        </tr>
        <tr>
            <td>started_at</td>
            <td>Datetime</td>
            <td>This represents the time that the image action was initiated.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>type</strong></td>
      <td>transfer</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>region</strong></td>
      <td>the desired region slug</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/119192826/actions" -d '{"type":"transfer","region":"sfo1"}' -X POST \
	-H "Authorization: Bearer 770ac4e7820e5c7153f54237466fc533b2b457f9bd11efa5c9372ddee5980a37" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 770ac4e7820e5c7153f54237466fc533b2b457f9bd11efa5c9372ddee5980a37
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "type": "transfer",
  "region": "sfo1"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354147
Content-Length: 128
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 4,
    "status": "in-progress",
    "type": "transfer",
    "started_at": "2014-06-09T21:49:07Z"
  }
}
      ```
  

## Image Actions Member [/v2/images/{image_id}/actions/{image_action_id}]

### Image Actions Retrieve an existing Image Action [GET]



**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/119192825/actions/3" -X GET \
	-H "Authorization: Bearer 57be924ee06e41684c09f2d7338b43eaaffc99b1ce6d1708a70426d174a85225"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 57be924ee06e41684c09f2d7338b43eaaffc99b1ce6d1708a70426d174a85225
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354147
Content-Length: 128
      ```

  
    - Body

      ```json
      {
  "event": {
    "id": 3,
    "status": "in-progress",
    "type": "transfer",
    "started_at": "2014-06-09T21:49:07Z"
  }
}
      ```
  
# Images


Images are either snapshots or backups you've made, or public images of applications or base systems.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>A unique integer that can be used to identify and reference a specific image.</td>
      <td><code>1234</code></td>
    </tr>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>The display name that has been given to an image.  This is what is shown in the control panel and is generally a descriptive title for the image in question.</td>
      <td><code>my image</code></td>
    </tr>
    <tr>
      <td><strong>distribution</strong></td>
      <td><em>string</em></td>
      <td>This attribute describes the base distribution used for this image.  This will not contain release information, but only a simple string like "Ubuntu".</td>
      <td><code>ubuntu</code></td>
    </tr>
    <tr>
      <td><strong>slug</strong></td>
      <td><em>nullable string</em></td>
      <td>A uniquely identifying string that is associated with each of the DigitalOcean-provided public images.  These can be used to reference a public image as an alternative to the numeric id.</td>
      <td><code>ubuntu12.04</code></td>
    </tr>
    <tr>
      <td><strong>public</strong></td>
      <td><em>boolean</em></td>
      <td>This is a boolean value that indicates whether the image in question is public or not.  An image that is public is available to all accounts.  A non-public image is only accessible from your account.</td>
      <td><code>false</code></td>
    </tr>
    <tr>
      <td><strong>regions</strong></td>
      <td><em>array</em></td>
      <td>This attribute is an array of the regions that the image is available in.  The regions are represented by their identifying slug values.</td>
      <td><code>["nyc1", "sfo1"]</code></td>
    </tr>
  </tbody>
</table>

## Images Collection [/v2/images]

### Images List a
    ll Images [GET]

<p>To list all of the images available on your account, send a GET request to <code>/v2/images</code>.</p>

<p>The response will be an array of image objects.  The objects will each contain the standard image attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>A unique integer used to identify and reference a specific image.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The display name of the image.  This is shown in the web UI and is generally a descriptive title for the image in question.</td>
        </tr>
        <tr>
            <td>distribution</td>
            <td>string</td>
            <td>The base distribution used for this image.  This will not contain release information, but only a simple string like "Ubuntu".</td>
        </tr>
        <tr>
            <td>slug</td>
            <td>nullable string</td>
            <td>A uniquely identifying string that is associated with each of the DigitalOcean-provided public images.  These can be used to reference a public image as an alternative to the numeric id.</td>
        </tr>
        <tr>
            <td>public</td>
            <td>boolean</td>
            <td>A boolean value that indicates whether the image in question is public.  An image that is public is available to all accounts.  A non-public image is only accessible from your account.</td>
        </tr>
        <tr>
            <td>regions</td>
            <td>array</td>
            <td>An array of the regions that the image is available in.  The regions are represented by their identifying slug values.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/" -X GET \
	-H "Authorization: Bearer 8a991e3a01341c590727485466d8bb643082446a42176f7dcb34d93eb5bc2892"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 8a991e3a01341c590727485466d8bb643082446a42176f7dcb34d93eb5bc2892
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Total: 2
Content-Length: 390
      ```

  
    - Body

      ```json
      {
  "images": [
    {
      "id": 119192817,
      "name": "Ubuntu 13.04",
      "distribution": "ubuntu",
      "slug": "ubuntu1304",
      "public": true,
      "regions": [
        "nyc1"
      ]
    },
    {
      "id": 119192840,
      "name": "Ubuntu 13.04",
      "distribution": null,
      "slug": null,
      "public": false,
      "regions": [
        "nyc1"
      ]
    }
  ]
}
      ```
  

## Images Member [/v2/images/{public_image_slug}]

### Images Retrieve an existing Image by slug [GET]

<p>To retrieve information about a public image, one option is to send a GET request to <code>/v2/images/$IMAGE_SLUG</code>.</p>

<p>The response will be an image object containing the standard image attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>A unique integer used to identify and reference a specific image.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The display name of the image.  This is shown in the web UI and is generally a descriptive title for the image in question.</td>
        </tr>
        <tr>
            <td>distribution</td>
            <td>string</td>
            <td>The base distribution used for this image.  This will not contain release information, but only a simple string like "Ubuntu".</td>
        </tr>
        <tr>
            <td>slug</td>
            <td>nullable string</td>
            <td>A uniquely identifying string that is associated with each of the DigitalOcean-provided public images.  These can be used to reference a public image as an alternative to the numeric id.</td>
        </tr>
        <tr>
            <td>public</td>
            <td>boolean</td>
            <td>A boolean value that indicates whether the image in question is public.  An image that is public is available to all accounts.  A non-public image is only accessible from your account.</td>
        </tr>
        <tr>
            <td>regions</td>
            <td>array</td>
            <td>An array of the regions that the image is available in.  The regions are represented by their identifying slug values.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/ubuntu1304" -X GET \
	-H "Authorization: Bearer 3862e1cfa4777df5d33bfe22aa815f2ea9d5fe38ecf77bcdd3a7170df0d7821b"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 3862e1cfa4777df5d33bfe22aa815f2ea9d5fe38ecf77bcdd3a7170df0d7821b
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 182
      ```

  
    - Body

      ```json
      {
  "image": {
    "id": 119192817,
    "name": "Ubuntu 13.04",
    "distribution": "ubuntu",
    "slug": "ubuntu1304",
    "public": true,
    "regions": [
      "nyc1"
    ]
  }
}
      ```
  

### Images Update an Image [PUT]

<p>To update an image, send a PUT request to <code>/v2/images/$IMAGE_ID</code>.  Set the "name" attribute to the new value you would like to use.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The new name that you would like to use for the image.</td>
            <td>true</td>
    </tbody>
</table>

<p>The response will be an image object containing the standard image attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>A unique integer used to identify and reference a specific image.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The display name of the image.  This is shown in the web UI and is generally a descriptive title for the image in question.</td>
        </tr>
        <tr>
            <td>distribution</td>
            <td>string</td>
            <td>The base distribution used for this image.  This will not contain release information, but only a simple string like "Ubuntu".</td>
        </tr>
        <tr>
            <td>slug</td>
            <td>nullable string</td>
            <td>A uniquely identifying string that is associated with each of the DigitalOcean-provided public images.  These can be used to reference a public image as an alternative to the numeric id.</td>
        </tr>
        <tr>
            <td>public</td>
            <td>boolean</td>
            <td>A boolean value that indicates whether the image in question is public.  An image that is public is available to all accounts.  A non-public image is only accessible from your account.</td>
        </tr>
        <tr>
            <td>regions</td>
            <td>array</td>
            <td>An array of the regions that the image is available in.  The regions are represented by their identifying slug values.</td>
        </tr>
    </tbody>
</table>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>Image Name</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/119192842" -d '{"name":"New Image Name"}' -X PUT \
	-H "Authorization: Bearer 6327df9c58df8bd4a141cc0d230f1495fe770b1da22008e31898e09e8ef25432" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 6327df9c58df8bd4a141cc0d230f1495fe770b1da22008e31898e09e8ef25432
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "New Image Name"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 173
      ```

  
    - Body

      ```json
      {
  "image": {
    "id": 119192842,
    "name": "New Image Name",
    "distribution": null,
    "slug": null,
    "public": false,
    "regions": [
      "nyc1"
    ]
  }
}
      ```
  

### Images Delete an Image [DELETE]

<p>To delete an image, send a DELETE request to <code>/v2/images/$IMAGE_ID</code>.</p>

<p>A status of 204 will be given.  This indicates that the request was processed successfully, but that no response body is needed.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/119192843" -d '' -X DELETE \
	-H "Authorization: Bearer 41a4075969f0814ae05653643db56a0ccd8e2300e5fd2bd740bb7f505e9c0585" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 41a4075969f0814ae05653643db56a0ccd8e2300e5fd2bd740bb7f505e9c0585
Content-Type: application/x-www-form-urlencoded
      ```

  

  - **Response**

    - Headers

      ```
      X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354150
      ```

  

### Images Retrieve an existing Image by id [GET]

<p>To retrieve information about an image (public or private), send a GET request to <code>/v2/images/$IMAGE_ID</code>.</p>

<p>The response will be an image object containing the standard image attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>id</td>
            <td>integer</td>
            <td>A unique integer used to identify and reference a specific image.</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The display name of the image.  This is shown in the web UI and is generally a descriptive title for the image in question.</td>
        </tr>
        <tr>
            <td>distribution</td>
            <td>string</td>
            <td>The base distribution used for this image.  This will not contain release information, but only a simple string like "Ubuntu".</td>
        </tr>
        <tr>
            <td>slug</td>
            <td>nullable string</td>
            <td>A uniquely identifying string that is associated with each of the DigitalOcean-provided public images.  These can be used to reference a public image as an alternative to the numeric id.</td>
        </tr>
        <tr>
            <td>public</td>
            <td>boolean</td>
            <td>A boolean value that indicates whether the image in question is public.  An image that is public is available to all accounts.  A non-public image is only accessible from your account.</td>
        </tr>
        <tr>
            <td>regions</td>
            <td>array</td>
            <td>An array of the regions that the image is available in.  The regions are represented by their identifying slug values.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/images/119192844" -X GET \
	-H "Authorization: Bearer 62c245b5638e7358cc1984bcf708f936b8b9ebe02801b33ae4bde6a93ac6018d"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 62c245b5638e7358cc1984bcf708f936b8b9ebe02801b33ae4bde6a93ac6018d
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354150
Content-Length: 171
      ```

  
    - Body

      ```json
      {
  "image": {
    "id": 119192844,
    "name": "Ubuntu 13.04",
    "distribution": null,
    "slug": null,
    "public": false,
    "regions": [
      "nyc1"
    ]
  }
}
      ```
  
# Keys


Keys are your public SSH keys that you can use to access Droplets.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>id</strong></td>
      <td><em>integer</em></td>
      <td>This is a unique identification number for the key.  This can be used to reference a specific SSH key when you wish to embed a key into a Droplet.</td>
      <td><code>1234</code></td>
    </tr>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>This is the human-readable display name for the given SSH key.  This is used to easily identify the SSH keys when they are displayed.</td>
      <td><code>my key</code></td>
    </tr>
    <tr>
      <td><strong>fingerprint</strong></td>
      <td><em>string</em></td>
      <td>This attribute contains the fingerprint value that is generated from the public key.  This is a unique identifier that will differentiate it from other keys using a format that SSH recognizes.</td>
      <td><code>f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2</code></td>
    </tr>
    <tr>
      <td><strong>public_key</strong></td>
      <td><em>string</em></td>
      <td>This attribute contains the entire public key string that was uploaded.  This is what is embedded into the root user's `authorized_keys` file if you choose to include this SSH key during Droplet creation.</td>
      <td><code>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com</code></td>
    </tr>
  </tbody>
</table>

## Keys Collection [/v2/account/keys]

### Keys List all Keys [GET]

<p>To retrieve a list of all of the domains in your account, send a GET request to <code>/v2/domains</code>.</p>

An array of Domain objects will be returned, each of which contain the standard domain attributes:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/account/keys" -X GET \
	-H "Authorization: Bearer 9605dc4b941da6eb133892feb613f78766b5e5efa915a94fee7a955d334afd99"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 9605dc4b941da6eb133892feb613f78766b5e5efa915a94fee7a955d334afd99
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354151
Content-Length: 572
      ```

  
    - Body

      ```json
      {
  "ssh_keys": [
    {
      "id": 1,
      "fingerprint": "f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2",
      "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com",
      "name": "Example Key"
    }
  ]
}
      ```
  

### Keys Create a new Key [POST]

<p>To create a new domain, send a POST request to <code>/v2/domains</code>.  Set the "name" attribute to the domain name you are adding.  Set the "ip_address" attribute to the IP address you want to point the domain to.</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
            <th>Required?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The domain name to add to the DigitalOcean DNS management interface.  The name must be unique in DigitalOcean's DNS system.  The request will fail if the name has already been taken.</td>
            <td>true</td>
        </tr>
        <tr>
            <td>ip_address</td>
            <td>string</td>
            <td>This attribute contains the IP address you want the domain to point to.</td>
            <td>true</td>
        </tr>
    </tbody>
</table>

<p>The response will contain the standard attributes associated with a domain:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


<p>Keep in mind that, upon creation, the <code>zone_file</code> field will have a value of <code>null</code> until a zone file is generated and propagated through an automatic process on the DigitalOcean servers.</p>

**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>User assigned identifier</td>
      <td>true</td>
    </tr>
  
    <tr>
      <td><strong>public_key</strong></td>
      <td>Public SSH Key</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/account/keys" -d '{
  "name": "Example Key",
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com"
}' -X POST \
	-H "Authorization: Bearer 0a5b07dd9b6d52a42c39d747ef93d77417bd743aabf2d72b711c21cc020998d6" \
	-H "Content-Type: application/json"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 0a5b07dd9b6d52a42c39d747ef93d77417bd743aabf2d72b711c21cc020998d6
Content-Type: application/json
      ```

  
    - Body

      ```json
      {
  "name": "Example Key",
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354152
Content-Length: 551
      ```

  
    - Body

      ```json
      {
  "ssh_key": {
    "id": 6,
    "fingerprint": "f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2",
    "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com",
    "name": "Example Key"
  }
}
      ```
  

## Keys Member [/v2/account/keys/{id_or_fingerprint}]

### Keys Show [GET]

<p>To get details about a specific domain, send a GET request to <code>/v2/domains/$DOMAIN_NAME</code>. </p>

The response received will contain the standard attributes defined for a domain:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The name of the domain name itself.  The string should be in the form of <code>domain.TLD</code>.  For instance, <code>example.com</code> is a valid domain value.</td>
        </tr>
        <tr>
            <td>ttl</td>
            <td>integer</td>
            <td>This value is the time to live for the records on this domain, in seconds.  This defines the time frame that clients can cache queried information before a refresh should be requested.</td>
        </tr>
        <tr>
            <td>zone_file</td>
            <td>string</td>
            <td>This attribute contains the complete contents of the zone file for the selected domain.  Most individual domain records can be accessed through the <code>/v2/domains/$DOMAIN_NAME/records</code> endpoint.  However, the SOA record for the domain is only available through the <code>zone_file</code>.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/account/keys/f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2" -X GET \
	-H "Authorization: Bearer ada774bded6e3a3ea1eb817532cc6e5666b4d36000b2593c256dbcf0b3171b8b"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer ada774bded6e3a3ea1eb817532cc6e5666b4d36000b2593c256dbcf0b3171b8b
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354152
Content-Length: 551
      ```

  
    - Body

      ```json
      {
  "ssh_key": {
    "id": 2,
    "fingerprint": "f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2",
    "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com",
    "name": "Example Key"
  }
}
      ```
  

### Keys Destroy [DELETE]

<p>To delete a domain, send a DELETE request to <code>/v2/domains/$DOMAIN_NAME</code>.</p>

<p>The domain will be removed from your account and a response status of 204 will be returned.  This indicates a successful request with no response body.</p>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/account/keys/f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2" -d '' -X DELETE \
	-H "Authorization: Bearer c1ba6b2c4d367e511d292cc07d36eebd764b95f34944ff682debcabeabe5290c" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer c1ba6b2c4d367e511d292cc07d36eebd764b95f34944ff682debcabeabe5290c
Content-Type: application/x-www-form-urlencoded
      ```

  

  - **Response**

    - Headers

      ```
      X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354152
      ```

  

### Keys Update [PUT]

Update a Key
**Parameters:**
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
  
    <tr>
      <td><strong>name</strong></td>
      <td>User assigned identifier</td>
      <td>true</td>
    </tr>
  
  </tbody>
</table>

**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/account/keys/f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2" -d '{
  "name": "New Name"
}' -X PUT \
	-H "Authorization: Bearer 5d4b8ba40320e5ace9a4dd4c3b7cecb6537c4358389a9086ce17f0183a1194de" \
	-H "Content-Type: application/x-www-form-urlencoded"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 5d4b8ba40320e5ace9a4dd4c3b7cecb6537c4358389a9086ce17f0183a1194de
Content-Type: application/x-www-form-urlencoded
      ```

  
    - Body

      ```json
      {
  "name": "New Name"
}
      ```
  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354152
Content-Length: 551
      ```

  
    - Body

      ```json
      {
  "ssh_key": {
    "id": 4,
    "fingerprint": "f5:de:eb:64:2d:6a:b6:d5:bb:06:47:7f:04:4b:f8:e2",
    "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPrtBjQaNBwDSV3ePC86zaEWu06g4+KEiivyqWAiOTvIp33Nia3b91NjfQydMkJlVfKuFs+hf2buQvCvslF4NNmWqxkPB69d+fS0ZL8Y4FMqut2I8hJuDg5MHO66QX6BkMqjqt3vsaJqbn7/dy0rKsqnaHgH0xqg0sPccK98nhL3nuoDGrzlsK0zMdfktX/yRSdjlpj4KdufA8/9uX14YGXNyduKMr8Sl7fLiAgtM0J3HHPAEOXce1iSmfIbxn16c8ikOddgM5MGK8DveX4EEscqwG0MxNkXJxgrU3e+k6dkb6RKuvGCtdSthrJ5X6O99lZCP0L6i3CD69d13YFobB name@example.com",
    "name": "Example Key"
  }
}
      ```
  
# Regions


Sizes represent possible Droplet resources.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>slug</strong></td>
      <td><em>string</em></td>
      <td>A human-readable string that is used as a unique identifier for each region.</td>
      <td><code>nyc1</code></td>
    </tr>
    <tr>
      <td><strong>name</strong></td>
      <td><em>string</em></td>
      <td>The display name of the region.  This will be a full name that is used in the control panel and other interfaces.</td>
      <td><code>New York 1</code></td>
    </tr>
    <tr>
      <td><strong>sizes</strong></td>
      <td><em>array</em></td>
      <td>This attribute is set to an array which contains the identifying slugs for the sizes available in this region.</td>
      <td><code>["512mb", "1024mb"]</code></td>
    </tr>
    <tr>
      <td><strong>available</strong></td>
      <td><em>boolean</em></td>
      <td>This is a boolean value that represents whether new Droplets can be created in this region.</td>
      <td><code>true</code></td>
    </tr>
  </tbody>
</table>

## Regions Collection [/v2/regions]

### Regions List all Regions [GET]

<p>To list all of the regions that are available, send a GET request to <code>/v2/regions</code>.</p>

The response will be an array of region objects, each of which will contain the standard region attributes:

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>slug</td>
            <td>string</td>
            <td>A human-readable string that is used as a unique identifier for each region.  An example is "nyc2".</td>
        </tr>
        <tr>
            <td>name</td>
            <td>string</td>
            <td>The display name of the region.  This is the full name that is used in the web UI and other interfaces.</td>
        </tr>
        <tr>
            <td>sizes</td>
            <td>array</td>
            <td>An array which contains the sizes slugs that represent the sizes available for deployment in this region.</td>
        </tr>
        <tr>
            <td>available</td>
            <td>boolean</td>
            <td>A boolean that represents whether new Droplets can be created in this region.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/regions" -X GET \
	-H "Authorization: Bearer 810e8c2c675a0b471c888f0c4ef5a2548b0765b865d3c41cccfc377e4412be66"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 810e8c2c675a0b471c888f0c4ef5a2548b0765b865d3c41cccfc377e4412be66
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354147
Content-Length: 431
      ```

  
    - Body

      ```json
      {
  "regions": [
    {
      "slug": "nyc1",
      "name": "New York",
      "sizes": [

      ],
      "available": false
    },
    {
      "slug": "sfo1",
      "name": "San Francisco",
      "sizes": [
        "1024mb",
        "512mb"
      ],
      "available": true
    },
    {
      "slug": "ams4",
      "name": "Amsterdam",
      "sizes": [
        "1024mb",
        "512mb"
      ],
      "available": true
    }
  ]
}
      ```
  

# Sizes


Sizes represent possible Droplet resources.

<table>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Type</th>
      <th>Description</th>
      <th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>slug</strong></td>
      <td><em>string</em></td>
      <td>A human-readable string that is used to uniquely identify each size.</td>
      <td><code>512mb</code></td>
    </tr>
    <tr>
      <td><strong>memory</strong></td>
      <td><em>integer</em></td>
      <td>The amount of RAM available to Droplets created with this size.  This value is given in megabytes.</td>
      <td><code>512</code></td>
    </tr>
    <tr>
      <td><strong>vcpus</strong></td>
      <td><em>integer</em></td>
      <td>The number of virtual CPUs that are allocated to Droplets with this size.</td>
      <td><code>1</code></td>
    </tr>
    <tr>
      <td><strong>disk</strong></td>
      <td><em>integer</em></td>
      <td>This is the amount of disk space set aside for Droplets created with this size.  The value is given in gigabytes.</td>
      <td><code>20</code></td>
    </tr>
    <tr>
      <td><strong>transfer</strong></td>
      <td><em>integer</em></td>
      <td>The amount of transfer bandwidth that is available for Droplets created in this size.  This only counts traffic on the public interface.  The value is given in terabytes.</td>
      <td><code>5</code></td>
    </tr>
    <tr>
      <td><strong>price_monthly</strong></td>
      <td><em>string</em></td>
      <td>This attribute describes the monthly cost of this Droplet size if the Droplet is kept for an entire month.  The value is measured in US dollars.</td>
      <td><code>5</code></td>
    </tr>
    <tr>
      <td><strong>price_hourly</strong></td>
      <td><em>string</em></td>
      <td>This describes the price of the Droplet size as measured hourly.  The value is measured in US dollars.</td>
      <td><code>.00744</code></td>
    </tr>
    <tr>
      <td><strong>regions</strong></td>
      <td><em>array</em></td>
      <td>An array that contains the region slugs where this size is available for Droplet creates.</td>
      <td><code>["nyc1", "sfo1"]</code></td>
    </tr>
  </tbody>
</table>

## Sizes Collection [/v2/sizes]

### Sizes List all Sizes [GET]

<p>To list all of the sizes, send a GET request to <code>/v2/sizes</code>.</p>

<p>The response will be an array of size objects each of which contain the standard sizes attributes:</p>

<table class="pure-table pure-table-horizontal">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>slug</td>
            <td>string</td>
            <td>A human-readable string that is used to uniquely identify each Droplet/image size.</td>
        </tr>
        <tr>
            <td>memory</td>
            <td>integer</td>
            <td>The amount of RAM allocated to Droplets created of this size.  The value is represented in megabytes.</td>
        </tr>
        <tr>
            <td>vcpus</td>
            <td>integer</td>
            <td>The number of virtual CPUs allocated to Droplets of this size.</td>
        </tr>
        <tr>
            <td>disk</td>
            <td>integer</td>
            <td>The amount of disk space set aside for Droplets of this size.  The value is represented in gigabytes.</td>
        </tr>
        <tr>
            <td>transfer</td>
            <td>integer</td>
            <td>The amount of transfer bandwidth available for Droplets of this size.  The value is represented in terabytes.</td>
        </tr>
        <tr>
            <td>price_monthly</td>
            <td>string</td>
            <td>The monthly cost of this Droplet size.  This is represented in US dollars.</td>
        </tr>
        <tr>
            <td>price_hourly</td>
            <td>string</td>
            <td>The price of this Droplet size as measured hourly.  The value is represented in US dollars.</td>
        </tr>
        <tr>
            <td>regions</td>
            <td>array</td>
            <td>An array containing the region slugs where this size is available for Droplet creates.</td>
        </tr>
    </tbody>
</table>


**Example:**

  - **cURL**

    ```bash
    curl "https://api.digitalocean.com/v2/sizes" -X GET \
	-H "Authorization: Bearer 156891ad382380b48f518a7d00247f07c7be8375f6c6f535fc9630a19194b2b8"
    ```

  - **Request**

    - Headers

      ```
      Authorization: Bearer 156891ad382380b48f518a7d00247f07c7be8375f6c6f535fc9630a19194b2b8
      ```

  

  - **Response**

    - Headers

      ```
      Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1200
X-RateLimit-Remaining: 1199
X-RateLimit-Reset: 1402354149
Content-Length: 561
      ```

  
    - Body

      ```json
      {
  "sizes": [
    {
      "slug": "512mb",
      "memory": 512,
      "vcpus": 1,
      "disk": 20,
      "transfer": null,
      "price_monthly": "5.0",
      "price_hourly": "0.00744",
      "regions": [
        "nyc1",
        "br1",
        "sfo1",
        "ams4"
      ]
    },
    {
      "slug": "1024mb",
      "memory": 1024,
      "vcpus": 2,
      "disk": 30,
      "transfer": null,
      "price_monthly": "10.0",
      "price_hourly": "0.01488",
      "regions": [
        "nyc1",
        "br1",
        "sfo1",
        "ams4"
      ]
    }
  ]
}
      ```
  

