### stories
- authentication verifier could be a session token

GIVEN A user is authenticated with an authentication verifier
AND the users authentication verifier is compromised
WHEN the user revokes the authentication verifier
THEN that authentication verifier no longer works

GIVEN A user has had their auth with service provider expired
AND the user is still authed with the identity provider
WHEN the user goes to access the service provider WITH both post and get
THEN they should be reauthed with the service provider
AND their request should finish without needing to repeat any actions