keycloak_test.go

Located in `pkg/agent/authentication/authenticator`
Part of `package authenticator`

Tests two functions:
NewKeycloakAuthenticator
TestGetToken

NewKeycloakAuthenticaor:

Error Tests:

```
_, err := NewKeycloakAuthenticator(true, "", "")
if err == nil {
    t.Fatal("ERROR: successfully initialized keyfunc for empty issuer url")
}
```
```
_, err = NewKeycloakAuthenticator(false, "", "")
if err == nil {
    t.Fatal("ERROR: successfully initialized keyfunc for empty jwks json")
}
```
Tests to see if function properly fails with empty parameters.

```
_, err = NewKeycloakAuthenticator(true, "invalid url", "")
if err == nil {
    t.Fatal("ERROR: successfully initialized keyfunc for invalid url")
}
```
Tests to see if function properly fails is an invalid url is provided.

```
_, err = NewKeycloakAuthenticator(false, sample_json, "")
if err != nil {
    t.Fatalf("ERROR: could not create keyfunc from json: %v", err)
}
```
Tests to see if function works if a valid JWKS JSON is provided.

Currently errors with 
```
--- FAIL: TestNewKeycloakAuthenticator (0.00s)
    keycloak_test.go:43: ERROR: could not create keyfunc from json: Could not set up OIDC Discovery client with issuer = '{"keys":[{"kty":"RSA","e":"AQAB","use":"sig","kid":"MjhhMDk2N2M2NGEwMzgzYjk2OTI3YzdmMGVhOGYxNjI2OTc5Y2Y2MQ","alg":"RS256","n":"zZU9xSgK77PbtkjJgD2Vmmv6_QNe8B54eyOV0k5K2UwuSnhv9RyRA3aL7gDN-qkANemHw3H_4Tc5SKIMltVIYdWlOMW_2m3gDBOODjc1bE-WXEWX6nQkLAOkoFrGW3bgW8TFxfuwgZVTlb6cYkSyiwc5ueFV2xNqo96Qf7nm5E7KZ2QDTkSlNMdW-jIVHMKjuEsy_gtYMaEYrwk5N7VoiYwePaF3I0_g4G2tIrKTLb8DvHApsN1h-s7jMCQFBrY4vCf3RBlYULr4Nz7u8G2NL_L9vURSCU2V2A8rYRkoZoZwk3a3AyJiqeC4T_1rmb8XdrgeFHB5bzXZ7EI0TObhlw"}]}': error fetching {"keys":[{"kty":"RSA","e":"AQAB","use":"sig","kid":"MjhhMDk2N2M2NGEwMzgzYjk2OTI3YzdmMGVhOGYxNjI2OTc5Y2Y2MQ","alg":"RS256","n":"zZU9xSgK77PbtkjJgD2Vmmv6_QNe8B54eyOV0k5K2UwuSnhv9RyRA3aL7gDN-qkANemHw3H_4Tc5SKIMltVIYdWlOMW_2m3gDBOODjc1bE-WXEWX6nQkLAOkoFrGW3bgW8TFxfuwgZVTlb6cYkSyiwc5ueFV2xNqo96Qf7nm5E7KZ2QDTkSlNMdW-jIVHMKjuEsy_gtYMaEYrwk5N7VoiYwePaF3I0_g4G2tIrKTLb8DvHApsN1h-s7jMCQFBrY4vCf3RBlYULr4Nz7u8G2NL_L9vURSCU2V2A8rYRkoZoZwk3a3AyJiqeC4T_1rmb8XdrgeFHB5bzXZ7EI0TObhlw"}]}/.well-known/openid-configuration: parse "{\"keys\":[{\"kty\":\"RSA\",\"e\":\"AQAB\",\"use\":\"sig\",\"kid\":\"MjhhMDk2N2M2NGEwMzgzYjk2OTI3YzdmMGVhOGYxNjI2OTc5Y2Y2MQ\",\"alg\":\"RS256\",\"n\":\"zZU9xSgK77PbtkjJgD2Vmmv6_QNe8B54eyOV0k5K2UwuSnhv9RyRA3aL7gDN-qkANemHw3H_4Tc5SKIMltVIYdWlOMW_2m3gDBOODjc1bE-WXEWX6nQkLAOkoFrGW3bgW8TFxfuwgZVTlb6cYkSyiwc5ueFV2xNqo96Qf7nm5E7KZ2QDTkSlNMdW-jIVHMKjuEsy_gtYMaEYrwk5N7VoiYwePaF3I0_g4G2tIrKTLb8DvHApsN1h-s7jMCQFBrY4vCf3RBlYULr4Nz7u8G2NL_L9vURSCU2V2A8rYRkoZoZwk3a3AyJiqeC4T_1rmb8XdrgeFHB5bzXZ7EI0TObhlw\"}]}/.well-known/openid-configuration": first path segment in URL cannot contain colon
FAIL
exit status 1
FAIL    github.com/spiffe/tornjak/pkg/agent/authentication/authenticator        0.003s
```
Removing this line makes all tests pass.

```
if issuerURL != "" {
    _, err = NewKeycloakAuthenticator(true, issuerURL, "")
    if err != nil {
        t.Fatalf("ERROR: could not create keyfunc from HTTP: %v", err)
    }
} else {
    fmt.Printf("WARNING: not testing http jwks")
}
```
Checks to see if passes with issuerURL.



TestGetToken

All Tests Pass

```
// sample request with token
request_body, err := json.Marshal(map[string]string{
    "name": "nobody",
})
if err != nil {
    t.Fatalf("ERROR: could not create request body")
}

// test with no Authorization header
request, err := http.NewRequest("GET", "some/url", bytes.NewBuffer(request_body))
if err != nil {
    t.Fatalf("ERROR: could not create request")
}
token, err := getToken(request, "redirecturl")
if err == nil {
    t.Fatalf("ERROR: successfully obtained access token from request with no auth header: %s", token)
}

// test with Authorization header but no Bearer token
request.Header.Set("Authorization", "something else")
token, err = getToken(request, "redirecturl")
if err == nil {
    t.Fatalf("ERROR: successfully obtained access token from request with no bearer token: %s", token)
}

// test with Authorization header but empty Bearer token
request.Header.Set("Authorization", "Bearer ")
token, err = getToken(request, "redirecturl")
if err == nil {
    t.Fatalf("ERROR: successfully obtained access token from request with empty bearer token: %s", token)
}
```
All these test for invalid requests. No headers, or no tokens

```
// test with good Authorization header and bearer token
request.Header.Set("Authorization", "Bearer <access_token>")
token, err = getToken(request, "redirecturl")
if err != nil {
    t.Fatalf("ERROR: could not obtain access token from request with bearer token: %s", token)
}
```

Tests to see if it can return a proper header with the right input.




NewKeycloakAuthenticator:
Function to get JWKS URL from issuer using OpenIS connect discovery. Stores it
as a KeycloackAuthenticator struct.

GetToken:
Extracts the Bearer token string from Auhorization headers.