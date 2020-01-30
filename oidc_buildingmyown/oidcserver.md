## Building my own oidc server for an authservice
There was a lot of discussion I have had with friends lately about authentication. This stemmed
mostly from a belief I hold that applications should not need to know anything about
authentication. So there needs to be an authentication proxy inbetween the application and the
outside world to take of authentication for that service.

## ory/hydra and ory/fosite
This was the interesting part, lets go through their compose function to find out what we need to implement to create a server.
We need a storage and a strategy. Storage defines how to maintain state and strategy defines how to create that state
https://github.com/ory/fosite/blob/master/handler/openid/storage.go
https://github.com/ory/fosite/blob/master/handler/openid/strategy.go
There are different handlers which are used to deal with different portions of an authentication service, above are the oidc portions
storage and strategy interfaces
