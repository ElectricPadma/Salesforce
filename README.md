#Salesforce
This library wraps the [Force.com REST API](https://www.salesforce.com/us/developer/docs/api_rest/). Force.com is a suite of point-and-click tools that make creating custom employee-facing apps lightning fast. The Electric Imp Salesforce library enables you to interact with your Force.com objects, allowing you to easily create products that can interact with a powerful CRM backend.

**To add this library to your project, add `#require "Salesforce.class.nut:1.1.0"` to the top of your agent code.**

You can view the library’s source code on [GitHub](https://github.com/electricimp/Salesforce/tree/v1.1.0).

## Callbacks
All methods that make requests to the Force.com API can be called asynchronously (by providing the optional callback function) or synchronously (by not provinding the callback). If a callback is supplied, it must take two parameters: *err* and *data*. If no errors were encountered, *err* will be `null` and *data* will contain the result of the request. If an error occured during the request, *err* will contain the error information and *data* will be `null`.

If no callback is supplied (ie. a synchronous request was made), the method will return a table containing two fields, *err* and *data*, which follow the same conventions as outlined above.

**Note** If you receive an error with an *errorCode* of `"INVALID_SESSION_ID"` it means that your login is no longer valid (or that you never logged in).

## Class Usage

### Constructor: Salesforce(*consumerKey, consumerSecret, [loginService], [version]*)

To create a new Salesforce object you will need the Consumer Key and Consumer Secret of a Connected App. Information about creating a Connected App can be found [here](https://help.salesforce.com/apex/HTViewHelpDoc?id=connected_app_create.htm). The constructor also allows you to pass in two additional parameters to override defaults: *loginService* (default value is "login.salesforce.com") and *version* (default value is "v33.0"). If you are working in a Salesforce sandbox environment, you should pass `"test.salesforce.com"` as *loginService*.

```squirrel
#require "Salesforce.class.nut:1.1.0"

force <- Salesforce("<-- CONSUMER_KEY -->", "<-- CONSUMER_SECRET -->");
```

### Class Methods

### login(*username, password, [securityToken], [callback]*)

Once you’ve created a Salesforce object, you need to provide the login credentials via the *login()* method. This requires *username* and *password* parameter values. You may also provide an optional *securityToken* value (information about acquiring your security token can be found [here](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm)). The method may also be called with an optional callback function (see note [above](#callbacks)) that will be executed upon the completion of the login request.

The data from the *login()* method consists of a table with a single field, *result*, which contains a boolean value indicating whether or not the login was successful.

```squirrel
force.login(USERNAME, PASSWORD, SECURITY_TOKEN, function(err, data) {
    if (err != null) {
        // if there was an error, log it
        server.error (err);
        return;
    }

    // If the login failed
    if(data.result == false) {
        server.error("Could not login");
        return;
    }

    // Do things after logging in:
    // ...
 });
```

### isLoggedIn()

The **isLoggedIn()** method immediately returns a boolean value indicating whether or not the Salesforce object has completed a login request and stored the authentication token.

### setVersion(*versionString*)

The **setVersion()** method can be used to set or change the version of the Force.com REST API you are working with:

```squirrel
force.setVersion("v27.0");  // set to v27.0 instead of v33.0 (default)
```

### setLoginService(*loginService*)

The **setLoginService()** method can be used to set or change the endpoint the Salesforce object uses in its login requests:

```squirrel
//const ENVIRONMENT = "LIVE";
const ENVIRONMENT = "TEST";

// If we're not using our live Salesforce instance:
if (ENVIRONMENT == "TEST") force.setLoginService("https://test.salesforce.com");

// Login
// ...
```

### setToken(*token*)

The **setToken()** method can be used to manually set the token the Salesforce object uses in its requests. This method is intended to be used with the OAuth2 flow.

### setInstanceUrl(url)

The **setInstanceUrl()** method can be used to manually set or change the instanceUrl the Salesforce object uses in its requests.

### getRefreshToken()

The **getRefreshToken()** method will return the authentication’s [refresh token](https://help.salesforce.com/HTViewHelpDoc?id=remoteaccess_oauth_refresh_token_flow.htm&language=en_US) if it exists. If the login flow did not result in a refresh token, or a succesfull call to *login()* has not been made yet, this method will return `null`. Since the only authentication mechanism currently supported is username/password, which does not result in a refresh token, `getRefreshToken()` always returns `null`.

### getUser(*[callback]*)

The **getUser()** method makes a request to Salesforce to retreive information about the user currently logged in. If a successful call to the *login()* method has not yet occured, an error will be thrown.

```squirrel
force.login(USERNAME, PASSWORD, SECURITY_TOKEN, function(err, data) {
    if (err != null || (data != null && data.result == false)) {
        server.error("Could not login to Salesforce");
        return;
    }

    force.getUser(function(err, data) {
        if (err) {
            server.error("ERROR: " + http.jsonencode(err));
            return;
        }

        // If it worked, log all the information we have about the user

        foreach(idx, val in data) {
            server.log(idx + ": " + val);
        }
    })
})
```

### request(*verb, service, [body], [callback]*)

The **request()** method is the most basic way of interacting with objects in your Salesforce database. It creates an HTTP request with properly formated authentication headers, etc. If a successful call to the *login()* method has not yet occured, an error will be thrown.

The data from the *request()* method will be the parsed body of the request’s response. In the example below, we are fetching data on custom objects (campsites) from our Salesforce database:

```squirrel
force.login(USERNAME, PASSWORD, SECURITY_TOKEN, function(err, data) {
    if (err != null || (data != null && data.result == false)) {
        server.error("Could not login to Salesforce");
        return;
    }

    force.request("get", "sobjects/campsites__c", null, function(err, data) {
        if (err) {
            server.error("ERROR: " + http.jsonencode(err));
            return;
        }

        // Log the names of all the recent campsites
        if ("recentItems" in data) {
            server.log("Recent Campsites: ");
            foreach(campsite in data.recentItems) {
                server.log(campsite.Name);
            }
        }
    });
});
```

## License

The Salesforce library is licensed under the [MIT License](https://github.com/electricimp/salesforce/blob/master/LICENSE).
