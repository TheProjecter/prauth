Prauth is a very simple protocol and implementation for using internal authentication mechanisms to access both internal and external web services.

The gateway for proxy auth is minimal and simple and can be used by any organization that has its internal authentication mechanisms in place. Thus it will work with Open ID, Shibboleth, LDAP, even custom authentication systems.

The use of this proxy auth, allows using intranet authentication system for trusted external web services and websites as well.

There are 3 main parts to the prauth protocol:
  1. **Prauth Gateway**
  1. Prauth Client
  1. User

### Prauth Gateway ###
The prauth gateway is just a URL that uses the native authentication system and returns an information packet as defined by the prauth protocol. The prauth gateway url accepts a _return_ query string parameter which is the return url where the user information will be delivered. The information packet is delivered as the value of a query string variable <strong>i</strong>_to the return url._

The information packet is created as follows:
```
URL_Encode (
  Base64_Encode (
    JSON_Encode (
      {
        uid: <unique id>,
        ...
        ... 
        _token: MD5(<user data values list>, _secret, _return),
        _formula: "<user data keys list>, _secret, _return"
      }
    )
  )
)
```
The field names are defined as follows:

  * `_`secret = Shared secret
  * `_`return = The return URL of the client using the prauth gateway
  * `_`formula = List of user data keys and other variables used to generate the token
  * `_`token = The token string generated according to the _`_`formula_ that should be used to validate the user by the client.
  * uid = A unique string for the user for this gateway
  * name = Full name of the user
  * email = Email id of the user
  * first = First name of the user
  * last = Last name of the user
  * url = Any url related to the user
  * pic = Any image related to the user
  * ...
  * ...
  * You may define custom fields beyond this.

The general thumb rule is that field names with preceding underscore (`_`) are not part of user data fields returned as a result of authentication.

A Prauth gateway can be further categorized in two type:
  1. **Open Prauth Gatway**: This gateway does not validate the return url as trusted and does not use a shared secret as part of the MD5 token.
  1. **Closed Prauth Gateway**: This gateway validates the return url as a trusted URL and uses a relative shared secret as part of the MD5 token.

The implementation of both is up to the gateway and not part of this protocol.

### Prauth Client ###
The prauth client is not defined by the prauth protocol. Here are a general list of to-do's that if implemented by the client would ensure a safer user entry:
  * Generate a public token that maps to the resource requested and local timestamp and include it as part of the return URL
  * Validate the prauth request against the local time difference using the token that is part of the URL and make the public token use once (destroy the token)
--
> Note: The main purpose of Prauth is to simplify use of org wide authentication system for validating users for CMS websites, intranet applications etc. that require non critical user information such as name and email id. This protocol should not be used to transfer critical user information such as credit card numbers, date of birth etc. and the gateway can ensure this security.

## Example ##

**User:**
Name: John Galt
Email: john@example.org

**Prauth Gateway:**
URL: http://example.org/prauth/

**Prauth Client:**
Domain: http://blogger.com/
Return URL: http://blogger.com/prauth/

**Shared Secret:**
Secret1234

### Using Prauth ###

User John Galt tries to access http://blogger.com/johngalt/admin/ .

Blogger.com stores this request in a local data structure as:
```
  token1234: {
    url: 'http://blogger.com/johngalt/admin/',
    timestamp: '1305757281'
  }
```
It then redirects the user to:
http://example.org/prauth/?return=http://blogger.com/prauth/token1234

The Prauth gateway authenticates the user and generates the following information packet:
```
i = URL_Encode(
      Base64_Encode(
        JSON_Encode( 
          {
            name: "John Galt",
            email: "john@example.org",
            _formula: "name,email,_secret,_return",
            _token: "John Galtjohn@example.orgSecret1234http://blogger.com/prauth/token1234"
          }
        )
      )
    )
```

Let's call the resulting string to be <information packet>.

After authentication is done, the prauth gateway redirects the user to: http://blogger.com/prauth/token1234?i=<information packet>.

The prauth client at blogger.com now can validate the user by regenerating the token using the user data and formula in the information packet against the token in the same packet. It can also validate against the timestamp it stored locally for timeouts and can delete the token1234 data so that the same request cannot be repeated.

Once authenticated via prauth, blogger.com can create an authenticated session and even store a persistent cookie to save the session.