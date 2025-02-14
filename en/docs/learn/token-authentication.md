When a TPP registers an application, a consumer-key and a consumer-secret is given for that application. 
These are the credentials of the application that is being registered. The consumer-key becomes 
the unique identifier of the application, which is similar to a username that is used for authentication. 
When an **access token** is issued for the application, it is issued against the latter mentioned consumer-key.

TPPs generate access tokens and pass them in the incoming API requests. The generated access token is a simple 
string that you need to pass as an HTTP header. For example, `"Authorization: Bearer NtBQkXoKElu0H1a1fQ0DWfo6IX4a"`. 

WSO2 Open Banking supports two types of access tokens for authentication:

   - **Application Access Tokens**: Tokens to identify and authenticate an application. 
   
   - **User Access Tokens**: Tokens to identify the final user of an application. 

For more information on access tokens, see the 
[overview of access tokens in WSO2 API Manager documentation](https://apim.docs.wso2.com/en/latest/learn/consume-api/manage-application/generate-keys/obtain-access-token/overview-of-access-tokens/).

In order to cater to open banking requirements, WSO2 Open Banking contains the following access token 
features:

## Certificate-bound access tokens 

Certificate-bound access tokens are access tokens that have a certificate attached to them. When using certificate-bound 
access tokens, you need to pass the certificate, which is bound to the token to gain access to protected resources. This 
prevents attacks to the access tokens from unauthorized parties. The validation for such tokens take place as 
follows:

   - Step 1: During the token API invocation, ensure that the transport certificate is passed as a header or in the context. 
   This is performed by a **Token Filter** before the API hits the authorization server.

   - Step 2: Bind the transport certificate to the access token via the confirmation (`cnf`) claim. All **Grant Handler** 
   types (`authorization_code`, `client_credentials`, and `refresh_token`) bind the transport certificate hash to the 
   token using the `cnf` claim.

   - Step 3: Return the `cnf` claim via an access token, the `cnf` claim ensures that the certificate is bound to the access 
   token.
 
   - Step 4: Validate the certificate during the API invocation. The API Manager server needs to ensure that the 
   certificate being passed during the API call is the same certificate that is bound to the token.

The diagram below explains the flow and how the Grant Handler and Token Filters engage:
![token_flow](../assets/img/learn/access-tokens/token-flow.png)

### Token Filter

A Token Filter is engaged at the Servlet container level before the request approaches the Token API. The Token Filter 
engages the MTLS enforcement validator.

The **MTLS enforcement validator** enforces that a certificate needs to be passed during the token creation. This 
certificate is then bound to the access token. 
   
!!! note "To decode the certificate when it is passed from the load balancer to the Gateway:"
        1. Open the `<IS_HOME>/repository/conf/deployment.toml` file.
        2. Add the following configurations and set the value to `true`.
           ``` toml
            [open_banking.identity.mutualtls] 
            client_certificate_encode=true 
           ```
        3. Restart the servers.

### Grant Handler

A Grant Handler validates the grant, scopes, and access delegation. By default, the following Grant Handler types are 
engaged when issuing tokens:

   - Authorization Code Grant Handler  
   - Client Credentials Grant Handler   
   - Refresh Grant Handler  
   
Mutual TLS authentication is enabled by default in WSO2 Open Banking. When MTLS authentication is enabled, 
the **MTLS enforcement validator** is engaged. You need to ensure that the 
TLS certificate is passed with every token request. 

## Client Authentication 

According to OAuth 2.0, the authorization server and the client need to establish a client authentication method that 
meets the security requirements of the authorization server. The client has the option of choosing the authentication 
method. The OpenID specification mentions a set of client authentication methods to authenticate the clients to the 
authorization server when using the token endpoint. Regulatory applications can use either `private_key_jwt` 
or `tls_client_auth` as the authentication method.



