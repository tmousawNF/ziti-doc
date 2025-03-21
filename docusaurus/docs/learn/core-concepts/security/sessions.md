## API Session {#api-session}

API Sessions represent a client that is either partially or fully authenticated as a specific Ziti Identity.
API Sessions are used to scope [authentication](./authentication/auth.md) and [Posture Data](./authorization/posture-checks).
API Sessions are used to make [authorization](./authorization/auth.md) decisions. Clients interact with
API Sessions via a opaque security token value and is received during authentication via `/edge/client|management/v1/authenticate`.

## Session {#session}

Sessions represent a client's ability to dial (connect) or bind (host) a service. Clients receive them by request
to the `/edge/client/v1/sessions` endpoint. A sessions are service and identity specific. Clients interact with them
as an opaque security token and is provided to Edge Routers during dial or bind requests.