# Logging

## Log levels and log modes

Authorino outputs 3 levels of log messages: (from lowest to highest level)
1. `debug`
2. `info` (default)
3. `error`

`info` logging is restricted to high-level information of the gRPC and HTTP authorization services, limiting messages to incomming request and respective outgoing response logs, with reduced details about the corresponding objects (request payload and authorization result), and without any further detailed logs of the steps in between, except for errors.

Only `debug` logging will include processing details of each [Auth Pipeline](architecture.md#the-auth-pipeline), such as intermediary requests to validate identities with external auth servers, requests to external sources of auth metadata or authorization policies.

To configure the desired log level, set the environment variable `LOG_LEVEL` to one of the supported values listed above. Default log level is `info`.

Apart from log level, Authorino can output messages to the logs in 2 different formats:
- `production` (default): each line is a parseable JSON object with properties `{"level":string, "ts":int, "msg":string, "logger":string, extra values...}`
- `development`: more human-readable outputs, extra stack traces and logging info, plus extra values output as JSON, in the format: `<timestamp-iso-8601>\t<log-level>\t<logger>\t<message>\t{extra-values-as-json}`

To configure the desired log mode, set the environment variable `LOG_MODE` to one of the supported values listed above. Default log level is `production`.

## Sensitive data output to the logs

Authorino will never output HTTP headers and query string parameters to `info` log messages, as such values usually include sensitive data (e.g. access tokens, API keys and Authorino [Festival Wristbands](architecture.md#festival-wristband-authentication)). However, `debug` log messages may include such sensitive information and those are not redacted.

Therefore, **DO NOT USE `debug` LOG LEVEL IN PRODUCTION**! Instead use either `info` or `error`.

## Tracing ID

Most log messages associated with an auth request include a `request id` extra value. The value represents the ID of the external authorization request received and processed by Authorino. This value is particularly useful to link incomming request and outgoing response log messages, as well as the more fine-grained log details available only in `debug` level.

## Typical log messages

Some typical log messages output by the Authorino service are listed in the table below:

| logger | level | message | extra values |
| -------|-------|---------|--------|
| `authorino` | `info` | "setting instance base logger" | `min level=info\|debug`, `mode=production\|development` |
| `authorino` | `info` | "attempting to acquire leader lease authorino/cb88a58a.authorino.3scale.net...\n" | |
| `authorino` | `info` | "successfully acquired lease authorino/cb88a58a.authorino.3scale.net\n" | |
| `authorino` | `info` | "starting grpc service" | `port`, `tls` |
| `authorino` | `error` | "failed to obtain port for grpc auth service" | |
| `authorino` | `error` | "failed to load tls cert" | |
| `authorino` | `error` | "failed to start grpc service" | |
| `authorino` | `info` | "starting oidc service" | `port`, `tls` |
| `authorino` | `error` | "failed to obtain port for http oidc service" | |
| `authorino` | `error` | "failed to start oidc service" | |
| `authorino` | `info` | "starting manager" | |
| `authorino` | `error` | "unable to start manager" | |
| `authorino` | `error` | "unable to create controller" | `controller=authconfig\|secret\|authconfigstatusupdate` |
| `authorino` | `error` | "problem running manager" | |
| `authorino` | `info` | "starting status update manager" | |
| `authorino` | `error` | "unable to start status update manager" | |
| `authorino` | `error` | "problem running status update manager" | |
| `authorino.controller-runtime.metrics` | `info` | "metrics server is starting to listen" | `addr` |
| `authorino.controller-runtime.manager` | `info` | "starting metrics server" | `path`
| `authorino.controller-runtime.manager.events` | `debug` | "Normal" | `object={kind=ConfigMap, apiVersion=v1}`, `reauthorino.ason=LeaderElection`, `message="authorino-controller-manager-* became leader"`
| `authorino.controller-runtime.manager.events` | `debug` | "Normal" | `object={kind=Lease, apiVersion=coordination.k8s.io/v1}`, `reauthorino.ason=LeaderElection`, `message="authorino-controller-manager-* became leader"`
| `authorino.controller-runtime.manager.controller.authconfig` | `info` | "resource reconciled" | |
| `authorino.controller-runtime.manager.controller.authconfig` | `info` | "object has been deleted, deleted related configs" | |
| `authorino.controller-runtime.manager.controller.authconfig` | `info` | "host already taken in another namespace" | |
| `authorino.controller-runtime.manager.controller.authconfig.statusupdater` | `info` | "resource status updated" | |
| `authorino.controller-runtime.manager.controller.secret` | `info` | "resource reconciled" | |
| `authorino.controller-runtime.manager.controller.secret` | `info` | "could not reconcile authconfigs using api key autauthorino.hentication" | |
| `authorino.service.oidc` | `info` | "request received" | `request id`, `url`, `realm`, `config`, `path` |
| `authorino.service.oidc` | `info` | "response sent" | `request id` |
| `authorino.service.oidc` | `error` | "failed to serve oidc request" | |
| `authorino.service.auth` | `info` | "incoming authorization request" | `request id`, `object` |
| `authorino.service.auth` | `debug` | "incoming authorization request" | `request id`, `object` |
| `authorino.service.auth` | `info` | "outgoing authorization response" | `request id`, `authorized`, `response`, `object` |
| `authorino.service.auth` | `debug` | "outgoing authorization response" | `request id`, `authorized`, `response`, `object` |
| `authorino.service.auth` | `error` | "failed to create dynamic metadata" | `request id`, `object` |
| `authorino.service.auth.authpipeline` | `debug` | "skipping config" | `request id`, `config`, `reason` |
| `authorino.service.auth.authpipeline.identity` | `debug` | "identity validated" | `request id`, `config`, `object` |
| `authorino.service.auth.authpipeline.identity` | `debug` | "cannot validate identity" | `request id`, `config`, `reason` |
| `authorino.service.auth.authpipeline.identity` | `error` | "failed to extend identity object" | `request id`, `config`, `object` |
| `authorino.service.auth.authpipeline.identity.oidc` | `error` | "failed to discovery openid connect configuration" | `endpoint` |
| `authorino.service.auth.authpipeline.identity.oauth2` | `debug` | "sending token introspection request" | `request id`, `url`, `data` |
| `authorino.service.auth.authpipeline.identity.kubernetesauth` | `debug` | "calling kubernetes token review api" | `request id`, `tokenreview` |
| `authorino.service.auth.authpipeline.identity.apikey` | `error` | "Something went wrong fetching the authorized credentials" | |
| `authorino.service.auth.authpipeline.metadata` | `debug` | "fetched auth metadata" | `request id`, `config`, `object` |
| `authorino.service.auth.authpipeline.metadata` | `debug` | "cannot fetch metadata" | `request id`, `config`, `reason` |
| `authorino.service.auth.authpipeline.metadata.http` | `debug` | "sending request" | `request id`, `method`, `url`, `headers` |
| `authorino.service.auth.authpipeline.metadata.userinfo` | `debug` | "fetching user info" | `request id`, `endpoint` |
| `authorino.service.auth.authpipeline.metadata.uma` | `debug` | "requesting pat" | `request id`, `url`, `data`, `headers` |
| `authorino.service.auth.authpipeline.metadata.uma` | `debug` | "querying resources by uri" | `request id`, `url` |
| `authorino.service.auth.authpipeline.metadata.uma` | `debug` | "getting resource data" | `request id`, `url` |
| `authorino.service.auth.authpipeline.authorization` | `debug` | "evaluating for input" | `request id`, `input` |
| `authorino.service.auth.authpipeline.authorization` | `debug` | "access granted" | `request id`, `config`, `object` |
| `authorino.service.auth.authpipeline.authorization` | `debug` | "access denied" | `request id`, `config`, `reason` |
| `authorino.service.auth.authpipeline.authorization.opa` | `error` | "Invalid response from OPA policy evaluation" | `secret` |
| `authorino.service.auth.authpipeline.authorization.opa` | `error` | "Failed to precompile OPA policy" | `secret` |
| `authorino.service.auth.authpipeline.authorization.kubernetesauthz` | `debug` | "calling kubernetes subject access review api" | `request id`, `subjectaccessreview` |
| `authorino.service.auth.authpipeline.response` | `debug` | "dynamic response built" | `request id`, `config`, `object` |
| `authorino.service.auth.authpipeline.response` | `debug` | "cannot build dynamic response" | `request id`, `config`, `reason` |
| `authorino.authcredential` | `error` | "the credential was not found in the request header" | |
| `authorino.authcredential` | `error` | "the Authorization header is not set" | |
| `authorino.authcredential` | `error` | "the Cookie header is not set" | |

### Examples

The examples below are all with `LOG_LEVEL=debug` and `LOG_MODE=production`.

#### Booting up the service:

```jsonc
{"level":"info","ts":1634674939.7563884,"logger":"authorino","msg":"setting instance base logger","min level":"debug","mode":"production"}
{"level":"info","ts":1634674941.0670755,"logger":"authorino.controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"127.0.0.1:8080"}
{"level":"info","ts":1634674941.0946925,"logger":"authorino","msg":"starting grpc service","port":"50051","tls":true}
{"level":"info","ts":1634674941.103486,"logger":"authorino","msg":"starting oidc service","port":"8083","tls":true}
{"level":"info","ts":1634674941.1678321,"logger":"authorino","msg":"starting manager"}
{"level":"info","ts":1634674941.1765432,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting EventSource","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig","source":"kind source: /, Kind="}
{"level":"info","ts":1634674941.182127,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting Controller","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
{"level":"info","ts":1634674941.1928735,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"Starting EventSource","reconciler group":"","reconciler kind":"Secret","source":"kind source: /, Kind="}
{"level":"info","ts":1634674941.1951146,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"Starting Controller","reconciler group":"","reconciler kind":"Secret"}
{"level":"info","ts":1634674941.203209,"logger":"authorino.controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
{"level":"info","ts":1634674942.256725,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"Starting workers","reconciler group":"","reconciler kind":"Secret","worker count":1}
{"level":"info","ts":1634674942.2595103,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting workers","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig","worker count":1}
{"level":"info","ts":1634674942.5573237,"logger":"authorino","msg":"starting status update manager"}
{"level":"info","ts":1634674942.5718815,"logger":"authorino","msg":"attempting to acquire leader lease authorino/cb88a58a.authorino.3scale.net...\n"}
{"level":"info","ts":1634674959.4286165,"logger":"authorino","msg":"successfully acquired lease authorino/cb88a58a.authorino.3scale.net\n"}
{"level":"info","ts":1634674959.4316218,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting EventSource","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig","source":"kind source: /, Kind="}
{"level":"info","ts":1634674959.4348314,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting Controller","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
{"level":"debug","ts":1634674959.4354987,"logger":"authorino.controller-runtime.manager.events","msg":"Normal","object":{"kind":"ConfigMap","namespace":"authorino","name":"cb88a58a.authorino.3scale.net","uid":"fcefe0d5-87f6-4a01-8e80-7cf290bc269b","apiVersion":"v1","resourceVersion":"55791"},"reason":"LeaderElection","message":"authorino-controller-manager-5d86dbb56f-th8mf_c416d2c6-fe44-421b-a025-344a60b70614 became leader"}
{"level":"debug","ts":1634674959.5168512,"logger":"authorino.controller-runtime.manager.events","msg":"Normal","object":{"kind":"Lease","namespace":"authorino","name":"cb88a58a.authorino.3scale.net","uid":"2753012c-4a06-4cb3-93c9-857d092bf289","apiVersion":"coordination.k8s.io/v1","resourceVersion":"55792"},"reason":"LeaderElection","message":"authorino-controller-manager-5d86dbb56f-th8mf_c416d2c6-fe44-421b-a025-344a60b70614 became leader"}
{"level":"info","ts":1634674959.5356445,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Starting workers","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig","worker count":1}
```

#### Reconciling an `AuthConfig` and 2 related API key `Secret`s:

```jsonc
{"level":"info","ts":1634675069.1662104,"logger":"authorino.controller-runtime.manager.controller.authconfig.statusupdater","msg":"resource status updated","authconfig/status":"authorino/talker-api-protection"}
{"level":"info","ts":1634675069.2253225,"logger":"authorino.controller-runtime.manager.controller.authconfig.statusupdater","msg":"resource status updated","authconfig/status":"authorino/talker-api-protection"}
{"level":"info","ts":1634675069.3370266,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"resource reconciled","secret":"authorino/api-key-1"}
{"level":"info","ts":1634675069.3740098,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"resource reconciled","secret":"authorino/api-key-2"}
{"level":"info","ts":1634675069.9157252,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"resource reconciled","authconfig":"authorino/talker-api-protection"}
{"level":"info","ts":1634675069.9188104,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"resource reconciled","authconfig":"authorino/talker-api-protection"}
{"level":"info","ts":1634675069.9242647,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"resource reconciled","authconfig":"authorino/talker-api-protection"}
{"level":"info","ts":1634675070.1129518,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"resource reconciled","authconfig":"authorino/talker-api-protection"}
```

#### Enforcing `AuthConfig` while authenticating with Kubernetes authentication token:

<details>
  <summary><code>AuhConfig</code> composed of:</summary>

  - identity: k8s-auth, oidc, oauth2, apikey
  - metadata: http, oidc userinfo
  - authorization: opa, k8s-authz
  - response: wristband
</details>

```jsonc
{"level":"info","ts":1634830460.1486168,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"8157480586935853928","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":53144}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"http":{"id":"8157480586935853928","method":"GET","path":"/hello","host":"talker-api","scheme":"http"}}}}
{"level":"debug","ts":1634830460.1491194,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"8157480586935853928","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":53144}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"time":{"seconds":1634830460,"nanos":147259000},"http":{"id":"8157480586935853928","method":"GET","headers":{":authority":"talker-api",":method":"GET",":path":"/hello",":scheme":"http","accept":"*/*","authorization":"Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRsVWJZMENyVy1sZ0tFMVRMd19pcTFUWGtTYUl6T0hyWks0VHhKYnpEZUUifQ.eyJhdWQiOlsidGFsa2VyLWFwaSJdLCJleHAiOjE2MzQ4MzEwNTEsImlhdCI6MTYzNDgzMDQ1MSwiaXNzIjoiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImF1dGhvcmlubyIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhcGktY29uc3VtZXItMSIsInVpZCI6ImI0MGY1MzFjLWVjYWItNGYzMS1hNDk2LTJlYmM3MmFkZDEyMSJ9fSwibmJmIjoxNjM0ODMwNDUxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6YXV0aG9yaW5vOmFwaS1jb25zdW1lci0xIn0.PaP0vqdl5DPfErr84KfVhPdlsGAPgsw0NkDaA9rne1zXjzcO7KPPbXhFwZC-oIjSGG1HfRMSoQeCXbQz24PSATmX8l1T52a9IFeXgP7sQmXZIDbiPfTm3X09kIIlfPKHhK_f-jQwRIpMRqNgLntlZ-xXX3P1fOBBUYR8obTPAQ6NDDaLHxw2SAmHFTQWjM_DInPDemXX0mEm7nCPKifsNxHaQH4wx4CD3LCLGbCI9FHNf2Crid8mmGJXf4wzcH1VuKkpUlsmnlUgTG2bfT2lbhSF2lBmrrhTJyYk6_aA09DwL4Bf4kvG-JtCq0Bkd_XynViIsOtOnAhgmdSPkfr-oA","user-agent":"curl/7.65.3","x-envoy-internal":"true","x-forwarded-for":"10.244.0.11","x-forwarded-proto":"http","x-request-id":"4c5d5c97-e15b-46a3-877a-d8188e09e08f"},"path":"/hello","host":"talker-api","scheme":"http","protocol":"HTTP/1.1"}},"context_extensions":{"virtual_host":"local_service"},"metadata_context":{}}}
{"level":"debug","ts":1634830460.150506,"logger":"authorino.service.auth.authpipeline.identity.kubernetesauth","msg":"calling kubernetes token review api","request id":"8157480586935853928","tokenreview":{"metadata":{"creationTimestamp":null},"spec":{"token":"eyJhbGciOiJSUzI1NiIsImtpZCI6IkRsVWJZMENyVy1sZ0tFMVRMd19pcTFUWGtTYUl6T0hyWks0VHhKYnpEZUUifQ.eyJhdWQiOlsidGFsa2VyLWFwaSJdLCJleHAiOjE2MzQ4MzEwNTEsImlhdCI6MTYzNDgzMDQ1MSwiaXNzIjoiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImF1dGhvcmlubyIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhcGktY29uc3VtZXItMSIsInVpZCI6ImI0MGY1MzFjLWVjYWItNGYzMS1hNDk2LTJlYmM3MmFkZDEyMSJ9fSwibmJmIjoxNjM0ODMwNDUxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6YXV0aG9yaW5vOmFwaS1jb25zdW1lci0xIn0.PaP0vqdl5DPfErr84KfVhPdlsGAPgsw0NkDaA9rne1zXjzcO7KPPbXhFwZC-oIjSGG1HfRMSoQeCXbQz24PSATmX8l1T52a9IFeXgP7sQmXZIDbiPfTm3X09kIIlfPKHhK_f-jQwRIpMRqNgLntlZ-xXX3P1fOBBUYR8obTPAQ6NDDaLHxw2SAmHFTQWjM_DInPDemXX0mEm7nCPKifsNxHaQH4wx4CD3LCLGbCI9FHNf2Crid8mmGJXf4wzcH1VuKkpUlsmnlUgTG2bfT2lbhSF2lBmrrhTJyYk6_aA09DwL4Bf4kvG-JtCq0Bkd_XynViIsOtOnAhgmdSPkfr-oA","audiences":["talker-api"]},"status":{"user":{}}}}
{"level":"debug","ts":1634830460.1509938,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"8157480586935853928","config":{"Name":"api-keys","ExtendedProperties":[{"Name":"sub","Value":{"Static":null,"Pattern":"auth.identity.metadata.annotations.userid"}}],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":{"AuthCredentials":{"KeySelector":"APIKEY","In":"authorization_header"},"Name":"api-keys","LabelSelectors":{"audience":"talker-api","authorino.3scale.net/managed-by":"authorino"}},"KubernetesAuth":null},"reason":"credential not found"}
{"level":"debug","ts":1634830460.1517606,"logger":"authorino.service.auth.authpipeline.identity.oauth2","msg":"sending token introspection request","request id":"8157480586935853928","url":"http://talker-api:523b92b6-625d-4e1e-a313-77e7a8ae4e88@keycloak:8080/auth/realms/kuadrant/protocol/openid-connect/token/introspect","data":"token=eyJhbGciOiJSUzI1NiIsImtpZCI6IkRsVWJZMENyVy1sZ0tFMVRMd19pcTFUWGtTYUl6T0hyWks0VHhKYnpEZUUifQ.eyJhdWQiOlsidGFsa2VyLWFwaSJdLCJleHAiOjE2MzQ4MzEwNTEsImlhdCI6MTYzNDgzMDQ1MSwiaXNzIjoiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImF1dGhvcmlubyIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhcGktY29uc3VtZXItMSIsInVpZCI6ImI0MGY1MzFjLWVjYWItNGYzMS1hNDk2LTJlYmM3MmFkZDEyMSJ9fSwibmJmIjoxNjM0ODMwNDUxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6YXV0aG9yaW5vOmFwaS1jb25zdW1lci0xIn0.PaP0vqdl5DPfErr84KfVhPdlsGAPgsw0NkDaA9rne1zXjzcO7KPPbXhFwZC-oIjSGG1HfRMSoQeCXbQz24PSATmX8l1T52a9IFeXgP7sQmXZIDbiPfTm3X09kIIlfPKHhK_f-jQwRIpMRqNgLntlZ-xXX3P1fOBBUYR8obTPAQ6NDDaLHxw2SAmHFTQWjM_DInPDemXX0mEm7nCPKifsNxHaQH4wx4CD3LCLGbCI9FHNf2Crid8mmGJXf4wzcH1VuKkpUlsmnlUgTG2bfT2lbhSF2lBmrrhTJyYk6_aA09DwL4Bf4kvG-JtCq0Bkd_XynViIsOtOnAhgmdSPkfr-oA&token_type_hint=requesting_party_token"}
{"level":"debug","ts":1634830460.1620777,"logger":"authorino.service.auth.authpipeline.identity","msg":"identity validated","request id":"8157480586935853928","config":{"Name":"k8s-service-accounts","ExtendedProperties":[],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"object":{"aud":["talker-api"],"exp":1634831051,"iat":1634830451,"iss":"https://kubernetes.default.svc.cluster.local","kubernetes.io":{"namespace":"authorino","serviceaccount":{"name":"api-consumer-1","uid":"b40f531c-ecab-4f31-a496-2ebc72add121"}},"nbf":1634830451,"sub":"system:serviceaccount:authorino:api-consumer-1"}}
{"level":"debug","ts":1634830460.1622565,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"requesting pat","request id":"8157480586935853928","url":"http://talker-api:523b92b6-625d-4e1e-a313-77e7a8ae4e88@keycloak:8080/auth/realms/kuadrant/protocol/openid-connect/token","data":"grant_type=client_credentials","headers":{"Content-Type":["application/x-www-form-urlencoded"]}}
{"level":"debug","ts":1634830460.1670353,"logger":"authorino.service.auth.authpipeline.metadata.http","msg":"sending request","request id":"8157480586935853928","method":"GET","url":"http://talker-api.authorino.svc.cluster.local:3000/metadata?encoding=text/plain&original_path=/hello","headers":{"Content-Type":["text/plain"]}}
{"level":"debug","ts":1634830460.169326,"logger":"authorino.service.auth.authpipeline.metadata","msg":"cannot fetch metadata","request id":"8157480586935853928","config":{"Name":"oidc-userinfo","UserInfo":{"OIDC":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"},"Endpoint":"http://keycloak:8080/auth/realms/kuadrant"}},"UMA":null,"GenericHTTP":null},"reason":"Missing identity for OIDC issuer http://keycloak:8080/auth/realms/kuadrant. Skipping related UserInfo metadata."}
{"level":"debug","ts":1634830460.1753876,"logger":"authorino.service.auth.authpipeline.metadata","msg":"fetched auth metadata","request id":"8157480586935853928","config":{"Name":"http-metadata","UserInfo":null,"UMA":null,"GenericHTTP":{"Endpoint":"http://talker-api.authorino.svc.cluster.local:3000/metadata?encoding=text/plain&original_path={context.request.http.path}","Method":"GET","Parameters":[],"ContentType":"application/x-www-form-urlencoded","SharedSecret":"","AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"object":{"body":"","headers":{"Accept-Encoding":"gzip","Content-Type":"text/plain","Host":"talker-api.authorino.svc.cluster.local:3000","User-Agent":"Go-http-client/1.1","Version":"HTTP/1.1"},"method":"GET","path":"/metadata","query_string":"encoding=text/plain&original_path=/hello","uuid":"1aa6ac66-3179-4351-b1a7-7f6a761d5b61"}}
{"level":"debug","ts":1634830460.2331996,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"querying resources by uri","request id":"8157480586935853928","url":"http://keycloak:8080/auth/realms/kuadrant/authz/protection/resource_set?uri=/hello"}
{"level":"debug","ts":1634830460.2495668,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"getting resource data","request id":"8157480586935853928","url":"http://keycloak:8080/auth/realms/kuadrant/authz/protection/resource_set/e20d194c-274c-4845-8c02-0ca413c9bf18"}
{"level":"debug","ts":1634830460.2927864,"logger":"authorino.service.auth.authpipeline.metadata","msg":"fetched auth metadata","request id":"8157480586935853928","config":{"Name":"uma-resource-registry","UserInfo":null,"UMA":{"Endpoint":"http://keycloak:8080/auth/realms/kuadrant","ClientID":"talker-api","ClientSecret":"523b92b6-625d-4e1e-a313-77e7a8ae4e88"},"GenericHTTP":null},"object":[{"_id":"e20d194c-274c-4845-8c02-0ca413c9bf18","attributes":{},"displayName":"hello","name":"hello","owner":{"id":"57a645a5-fb67-438b-8be5-dfb971666dbc"},"ownerManagedAccess":false,"resource_scopes":[],"uris":["/hi","/hello"]}]}
{"level":"debug","ts":1634830460.2930083,"logger":"authorino.service.auth.authpipeline.authorization","msg":"evaluating for input","request id":"8157480586935853928","input":{"context":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":53144}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"time":{"seconds":1634830460,"nanos":147259000},"http":{"id":"8157480586935853928","method":"GET","headers":{":authority":"talker-api",":method":"GET",":path":"/hello",":scheme":"http","accept":"*/*","authorization":"Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRsVWJZMENyVy1sZ0tFMVRMd19pcTFUWGtTYUl6T0hyWks0VHhKYnpEZUUifQ.eyJhdWQiOlsidGFsa2VyLWFwaSJdLCJleHAiOjE2MzQ4MzEwNTEsImlhdCI6MTYzNDgzMDQ1MSwiaXNzIjoiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImF1dGhvcmlubyIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhcGktY29uc3VtZXItMSIsInVpZCI6ImI0MGY1MzFjLWVjYWItNGYzMS1hNDk2LTJlYmM3MmFkZDEyMSJ9fSwibmJmIjoxNjM0ODMwNDUxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6YXV0aG9yaW5vOmFwaS1jb25zdW1lci0xIn0.PaP0vqdl5DPfErr84KfVhPdlsGAPgsw0NkDaA9rne1zXjzcO7KPPbXhFwZC-oIjSGG1HfRMSoQeCXbQz24PSATmX8l1T52a9IFeXgP7sQmXZIDbiPfTm3X09kIIlfPKHhK_f-jQwRIpMRqNgLntlZ-xXX3P1fOBBUYR8obTPAQ6NDDaLHxw2SAmHFTQWjM_DInPDemXX0mEm7nCPKifsNxHaQH4wx4CD3LCLGbCI9FHNf2Crid8mmGJXf4wzcH1VuKkpUlsmnlUgTG2bfT2lbhSF2lBmrrhTJyYk6_aA09DwL4Bf4kvG-JtCq0Bkd_XynViIsOtOnAhgmdSPkfr-oA","user-agent":"curl/7.65.3","x-envoy-internal":"true","x-forwarded-for":"10.244.0.11","x-forwarded-proto":"http","x-request-id":"4c5d5c97-e15b-46a3-877a-d8188e09e08f"},"path":"/hello","host":"talker-api","scheme":"http","protocol":"HTTP/1.1"}},"context_extensions":{"virtual_host":"local_service"},"metadata_context":{}},"auth":{"identity":{"aud":["talker-api"],"exp":1634831051,"iat":1634830451,"iss":"https://kubernetes.default.svc.cluster.local","kubernetes.io":{"namespace":"authorino","serviceaccount":{"name":"api-consumer-1","uid":"b40f531c-ecab-4f31-a496-2ebc72add121"}},"nbf":1634830451,"sub":"system:serviceaccount:authorino:api-consumer-1"},"metadata":{"http-metadata":{"body":"","headers":{"Accept-Encoding":"gzip","Content-Type":"text/plain","Host":"talker-api.authorino.svc.cluster.local:3000","User-Agent":"Go-http-client/1.1","Version":"HTTP/1.1"},"method":"GET","path":"/metadata","query_string":"encoding=text/plain&original_path=/hello","uuid":"1aa6ac66-3179-4351-b1a7-7f6a761d5b61"},"uma-resource-registry":[{"_id":"e20d194c-274c-4845-8c02-0ca413c9bf18","attributes":{},"displayName":"hello","name":"hello","owner":{"id":"57a645a5-fb67-438b-8be5-dfb971666dbc"},"ownerManagedAccess":false,"resource_scopes":[],"uris":["/hi","/hello"]}]}}}}
{"level":"debug","ts":1634830460.2955465,"logger":"authorino.service.auth.authpipeline.authorization.kubernetesauthz","msg":"calling kubernetes subject access review api","request id":"8157480586935853928","subjectaccessreview":{"metadata":{"creationTimestamp":null},"spec":{"nonResourceAttributes":{"path":"/hello","verb":"get"},"user":"system:serviceaccount:authorino:api-consumer-1"},"status":{"allowed":false}}}
{"level":"debug","ts":1634830460.2986183,"logger":"authorino.service.auth.authpipeline.authorization","msg":"access granted","request id":"8157480586935853928","config":{"Name":"my-policy","OPA":{"Rego":"fail := input.context.request.http.headers[\"x-ext-auth-mock\"] == \"FAIL\"\nallow { not fail }\n","OPAExternalSource":{"Endpoint":"","SharedSecret":"","AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"JSON":null,"KubernetesAuthz":null},"object":true}
{"level":"debug","ts":1634830460.3044975,"logger":"authorino.service.auth.authpipeline.authorization","msg":"access granted","request id":"8157480586935853928","config":{"Name":"kubernetes-rbac","OPA":null,"JSON":null,"KubernetesAuthz":{"Conditions":[],"User":{"Static":"","Pattern":"auth.identity.sub"},"Groups":null,"ResourceAttributes":null}},"object":true}
{"level":"debug","ts":1634830460.3052874,"logger":"authorino.service.auth.authpipeline.response","msg":"dynamic response built","request id":"8157480586935853928","config":{"Name":"wristband","Wrapper":"httpHeader","WrapperKey":"x-ext-auth-wristband","Wristband":{"Issuer":"https://authorino-oidc.authorino.svc:8083/authorino/talker-api-protection/wristband","CustomClaims":[],"TokenDuration":300,"SigningKeys":[{"use":"sig","kty":"EC","kid":"wristband-signing-key","crv":"P-256","alg":"ES256","x":"TJf5NLVKplSYp95TOfhVPqvxvEibRyjrUZwwtpDuQZw","y":"SSg8rKBsJ3J1LxyLtt0oFvhHvZcUpmRoTuHk3UHisTA","d":"Me-5_zWBWVYajSGZcZMCcD8dXEa4fy85zv_yN7BxW-o"}]},"DynamicJSON":null},"object":"eyJhbGciOiJFUzI1NiIsImtpZCI6IndyaXN0YmFuZC1zaWduaW5nLWtleSIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MzQ4MzA3NjAsImlhdCI6MTYzNDgzMDQ2MCwiaXNzIjoiaHR0cHM6Ly9hdXRob3Jpbm8tb2lkYy5hdXRob3Jpbm8uc3ZjOjgwODMvYXV0aG9yaW5vL3RhbGtlci1hcGktcHJvdGVjdGlvbi93cmlzdGJhbmQiLCJzdWIiOiI4NDliMDk0ZDA4MzU0ZjM0MjA4ZGI3MjBmYWZmODlmNmM3NmYyOGY3MTcxOWI4NTQ3ZDk5NWNlNzAwMjU2ZGY4In0.Jn-VB5Q_0EX1ed1ji4KvhO4DlMqZeIl5H0qlukbTyYkp-Pgb4SnPGSbYWp5_uvG8xllsFAA5nuyBIXeba-dbkw"}
{"level":"info","ts":1634830460.3054585,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"8157480586935853928","authorized":true,"response":"OK"}
{"level":"debug","ts":1634830460.305476,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"8157480586935853928","authorized":true,"response":"OK"}
```

#### Enforcing `AuthConfig` while authenticating with API key:

<details>
  <summary><code>AuhConfig</code> composed of:</summary>

  - identity: k8s-auth, oidc, oauth2, apikey
  - metadata: http, oidc userinfo
  - authorization: opa, k8s-authz
  - response: wristband
</details>

```jsonc
{"level":"info","ts":1634830413.2425854,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"7199257136822741594","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":52702}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"http":{"id":"7199257136822741594","method":"GET","path":"/hello","host":"talker-api","scheme":"http"}}}}
{"level":"debug","ts":1634830413.2426975,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"7199257136822741594","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":52702}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"time":{"seconds":1634830413,"nanos":240094000},"http":{"id":"7199257136822741594","method":"GET","headers":{":authority":"talker-api",":method":"GET",":path":"/hello",":scheme":"http","accept":"*/*","authorization":"APIKEY ndyBzreUzF4zqDQsqSPMHkRhriEOtcRx","user-agent":"curl/7.65.3","x-envoy-internal":"true","x-forwarded-for":"10.244.0.11","x-forwarded-proto":"http","x-request-id":"d38f5e66-bd72-4733-95d1-3179315cdd60"},"path":"/hello","host":"talker-api","scheme":"http","protocol":"HTTP/1.1"}},"context_extensions":{"virtual_host":"local_service"},"metadata_context":{}}}
{"level":"debug","ts":1634830413.2428744,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"7199257136822741594","config":{"Name":"k8s-service-accounts","ExtendedProperties":[],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"reason":"credential not found"}
{"level":"debug","ts":1634830413.2434332,"logger":"authorino.service.auth.authpipeline","msg":"skipping config","request id":"7199257136822741594","config":{"Name":"keycloak-jwts","ExtendedProperties":[],"OAuth2":null,"OIDC":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"},"Endpoint":"http://keycloak:8080/auth/realms/kuadrant"},"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":null},"reason":"context canceled"}
{"level":"debug","ts":1634830413.2479305,"logger":"authorino.service.auth.authpipeline.identity","msg":"identity validated","request id":"7199257136822741594","config":{"Name":"api-keys","ExtendedProperties":[{"Name":"sub","Value":{"Static":null,"Pattern":"auth.identity.metadata.annotations.userid"}}],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":{"AuthCredentials":{"KeySelector":"APIKEY","In":"authorization_header"},"Name":"api-keys","LabelSelectors":{"audience":"talker-api","authorino.3scale.net/managed-by":"authorino"}},"KubernetesAuth":null},"object":{"apiVersion":"v1","data":{"api_key":"bmR5QnpyZVV6RjR6cURRc3FTUE1Ia1JocmlFT3RjUng="},"kind":"Secret","metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"v1\",\"kind\":\"Secret\",\"metadata\":{\"annotations\":{\"userid\":\"john\"},\"labels\":{\"audience\":\"talker-api\",\"authorino.3scale.net/managed-by\":\"authorino\"},\"name\":\"api-key-1\",\"namespace\":\"authorino\"},\"stringData\":{\"api_key\":\"ndyBzreUzF4zqDQsqSPMHkRhriEOtcRx\"},\"type\":\"Opaque\"}\n","userid":"john"},"creationTimestamp":"2021-10-21T14:45:54Z","labels":{"audience":"talker-api","authorino.3scale.net/managed-by":"authorino"},"managedFields":[{"apiVersion":"v1","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:api_key":{}},"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{},"f:userid":{}},"f:labels":{".":{},"f:audience":{},"f:authorino.3scale.net/managed-by":{}}},"f:type":{}},"manager":"kubectl-client-side-apply","operation":"Update","time":"2021-10-21T14:45:54Z"}],"name":"api-key-1","namespace":"authorino","resourceVersion":"8979","uid":"c369852a-7e1a-43bd-94ca-e2b3f617052e"},"sub":"john","type":"Opaque"}}
{"level":"debug","ts":1634830413.248768,"logger":"authorino.service.auth.authpipeline.metadata.http","msg":"sending request","request id":"7199257136822741594","method":"GET","url":"http://talker-api.authorino.svc.cluster.local:3000/metadata?encoding=text/plain&original_path=/hello","headers":{"Content-Type":["text/plain"]}}
{"level":"debug","ts":1634830413.2496722,"logger":"authorino.service.auth.authpipeline.metadata","msg":"cannot fetch metadata","request id":"7199257136822741594","config":{"Name":"oidc-userinfo","UserInfo":{"OIDC":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"},"Endpoint":"http://keycloak:8080/auth/realms/kuadrant"}},"UMA":null,"GenericHTTP":null},"reason":"Missing identity for OIDC issuer http://keycloak:8080/auth/realms/kuadrant. Skipping related UserInfo metadata."}
{"level":"debug","ts":1634830413.2497928,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"requesting pat","request id":"7199257136822741594","url":"http://talker-api:523b92b6-625d-4e1e-a313-77e7a8ae4e88@keycloak:8080/auth/realms/kuadrant/protocol/openid-connect/token","data":"grant_type=client_credentials","headers":{"Content-Type":["application/x-www-form-urlencoded"]}}
{"level":"debug","ts":1634830413.258932,"logger":"authorino.service.auth.authpipeline.metadata","msg":"fetched auth metadata","request id":"7199257136822741594","config":{"Name":"http-metadata","UserInfo":null,"UMA":null,"GenericHTTP":{"Endpoint":"http://talker-api.authorino.svc.cluster.local:3000/metadata?encoding=text/plain&original_path={context.request.http.path}","Method":"GET","Parameters":[],"ContentType":"application/x-www-form-urlencoded","SharedSecret":"","AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"object":{"body":"","headers":{"Accept-Encoding":"gzip","Content-Type":"text/plain","Host":"talker-api.authorino.svc.cluster.local:3000","User-Agent":"Go-http-client/1.1","Version":"HTTP/1.1"},"method":"GET","path":"/metadata","query_string":"encoding=text/plain&original_path=/hello","uuid":"97529f8c-587b-4121-a4db-cd90c63871fd"}}
{"level":"debug","ts":1634830413.2945344,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"querying resources by uri","request id":"7199257136822741594","url":"http://keycloak:8080/auth/realms/kuadrant/authz/protection/resource_set?uri=/hello"}
{"level":"debug","ts":1634830413.3123596,"logger":"authorino.service.auth.authpipeline.metadata.uma","msg":"getting resource data","request id":"7199257136822741594","url":"http://keycloak:8080/auth/realms/kuadrant/authz/protection/resource_set/e20d194c-274c-4845-8c02-0ca413c9bf18"}
{"level":"debug","ts":1634830413.3340268,"logger":"authorino.service.auth.authpipeline.metadata","msg":"fetched auth metadata","request id":"7199257136822741594","config":{"Name":"uma-resource-registry","UserInfo":null,"UMA":{"Endpoint":"http://keycloak:8080/auth/realms/kuadrant","ClientID":"talker-api","ClientSecret":"523b92b6-625d-4e1e-a313-77e7a8ae4e88"},"GenericHTTP":null},"object":[{"_id":"e20d194c-274c-4845-8c02-0ca413c9bf18","attributes":{},"displayName":"hello","name":"hello","owner":{"id":"57a645a5-fb67-438b-8be5-dfb971666dbc"},"ownerManagedAccess":false,"resource_scopes":[],"uris":["/hi","/hello"]}]}
{"level":"debug","ts":1634830413.3367748,"logger":"authorino.service.auth.authpipeline.authorization","msg":"evaluating for input","request id":"7199257136822741594","input":{"context":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":52702}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"time":{"seconds":1634830413,"nanos":240094000},"http":{"id":"7199257136822741594","method":"GET","headers":{":authority":"talker-api",":method":"GET",":path":"/hello",":scheme":"http","accept":"*/*","authorization":"APIKEY ndyBzreUzF4zqDQsqSPMHkRhriEOtcRx","user-agent":"curl/7.65.3","x-envoy-internal":"true","x-forwarded-for":"10.244.0.11","x-forwarded-proto":"http","x-request-id":"d38f5e66-bd72-4733-95d1-3179315cdd60"},"path":"/hello","host":"talker-api","scheme":"http","protocol":"HTTP/1.1"}},"context_extensions":{"virtual_host":"local_service"},"metadata_context":{}},"auth":{"identity":{"apiVersion":"v1","data":{"api_key":"bmR5QnpyZVV6RjR6cURRc3FTUE1Ia1JocmlFT3RjUng="},"kind":"Secret","metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"v1\",\"kind\":\"Secret\",\"metadata\":{\"annotations\":{\"userid\":\"john\"},\"labels\":{\"audience\":\"talker-api\",\"authorino.3scale.net/managed-by\":\"authorino\"},\"name\":\"api-key-1\",\"namespace\":\"authorino\"},\"stringData\":{\"api_key\":\"ndyBzreUzF4zqDQsqSPMHkRhriEOtcRx\"},\"type\":\"Opaque\"}\n","userid":"john"},"creationTimestamp":"2021-10-21T14:45:54Z","labels":{"audience":"talker-api","authorino.3scale.net/managed-by":"authorino"},"managedFields":[{"apiVersion":"v1","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:api_key":{}},"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{},"f:userid":{}},"f:labels":{".":{},"f:audience":{},"f:authorino.3scale.net/managed-by":{}}},"f:type":{}},"manager":"kubectl-client-side-apply","operation":"Update","time":"2021-10-21T14:45:54Z"}],"name":"api-key-1","namespace":"authorino","resourceVersion":"8979","uid":"c369852a-7e1a-43bd-94ca-e2b3f617052e"},"sub":"john","type":"Opaque"},"metadata":{"http-metadata":{"body":"","headers":{"Accept-Encoding":"gzip","Content-Type":"text/plain","Host":"talker-api.authorino.svc.cluster.local:3000","User-Agent":"Go-http-client/1.1","Version":"HTTP/1.1"},"method":"GET","path":"/metadata","query_string":"encoding=text/plain&original_path=/hello","uuid":"97529f8c-587b-4121-a4db-cd90c63871fd"},"uma-resource-registry":[{"_id":"e20d194c-274c-4845-8c02-0ca413c9bf18","attributes":{},"displayName":"hello","name":"hello","owner":{"id":"57a645a5-fb67-438b-8be5-dfb971666dbc"},"ownerManagedAccess":false,"resource_scopes":[],"uris":["/hi","/hello"]}]}}}}
{"level":"debug","ts":1634830413.339894,"logger":"authorino.service.auth.authpipeline.authorization","msg":"access granted","request id":"7199257136822741594","config":{"Name":"my-policy","OPA":{"Rego":"fail := input.context.request.http.headers[\"x-ext-auth-mock\"] == \"FAIL\"\nallow { not fail }\n","OPAExternalSource":{"Endpoint":"","SharedSecret":"","AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"JSON":null,"KubernetesAuthz":null},"object":true}
{"level":"debug","ts":1634830413.3444238,"logger":"authorino.service.auth.authpipeline.authorization.kubernetesauthz","msg":"calling kubernetes subject access review api","request id":"7199257136822741594","subjectaccessreview":{"metadata":{"creationTimestamp":null},"spec":{"nonResourceAttributes":{"path":"/hello","verb":"get"},"user":"john"},"status":{"allowed":false}}}
{"level":"debug","ts":1634830413.3547812,"logger":"authorino.service.auth.authpipeline.authorization","msg":"access granted","request id":"7199257136822741594","config":{"Name":"kubernetes-rbac","OPA":null,"JSON":null,"KubernetesAuthz":{"Conditions":[],"User":{"Static":"","Pattern":"auth.identity.sub"},"Groups":null,"ResourceAttributes":null}},"object":true}
{"level":"debug","ts":1634830413.3558292,"logger":"authorino.service.auth.authpipeline.response","msg":"dynamic response built","request id":"7199257136822741594","config":{"Name":"wristband","Wrapper":"httpHeader","WrapperKey":"x-ext-auth-wristband","Wristband":{"Issuer":"https://authorino-oidc.authorino.svc:8083/authorino/talker-api-protection/wristband","CustomClaims":[],"TokenDuration":300,"SigningKeys":[{"use":"sig","kty":"EC","kid":"wristband-signing-key","crv":"P-256","alg":"ES256","x":"TJf5NLVKplSYp95TOfhVPqvxvEibRyjrUZwwtpDuQZw","y":"SSg8rKBsJ3J1LxyLtt0oFvhHvZcUpmRoTuHk3UHisTA","d":"Me-5_zWBWVYajSGZcZMCcD8dXEa4fy85zv_yN7BxW-o"}]},"DynamicJSON":null},"object":"eyJhbGciOiJFUzI1NiIsImtpZCI6IndyaXN0YmFuZC1zaWduaW5nLWtleSIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MzQ4MzA3MTMsImlhdCI6MTYzNDgzMDQxMywiaXNzIjoiaHR0cHM6Ly9hdXRob3Jpbm8tb2lkYy5hdXRob3Jpbm8uc3ZjOjgwODMvYXV0aG9yaW5vL3RhbGtlci1hcGktcHJvdGVjdGlvbi93cmlzdGJhbmQiLCJzdWIiOiI5NjhiZjViZjk3MDM3NWRiNjE0ZDFhMDgzZTg2NTBhYTVhMGVhMzAyOTdiYmJjMTBlNWVlMWZmYTkxYTYwZmY4In0.7G440sWgi2TIaxrGJf5KWR9UOFpNTjwVYeaJXFLzsLhVNICoMLbYzBAEo4M3ym1jipxxTVeE7anm4qDDc7cnVQ"}
{"level":"info","ts":1634830413.3569078,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"7199257136822741594","authorized":true,"response":"OK"}
{"level":"debug","ts":1634830413.3569596,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"7199257136822741594","authorized":true,"response":"OK"}
```

#### Enforcing `AuthConfig` while authenticating with invalid API key:

<details>
  <summary><code>AuhConfig</code> composed of:</summary>

  - identity: k8s-auth, oidc, oauth2, apikey
  - metadata: http, oidc userinfo
  - authorization: opa, k8s-authz
  - response: wristband
</details>

```jsonc
{"level":"info","ts":1634830373.2066543,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"12947265773116138711","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":52288}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"http":{"id":"12947265773116138711","method":"GET","path":"/hello","host":"talker-api","scheme":"http"}}}}
{"level":"debug","ts":1634830373.2068064,"logger":"authorino.service.auth","msg":"incoming authorization request","request id":"12947265773116138711","object":{"source":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":52288}}}}},"destination":{"address":{"Address":{"SocketAddress":{"address":"127.0.0.1","PortSpecifier":{"PortValue":8000}}}}},"request":{"time":{"seconds":1634830373,"nanos":198329000},"http":{"id":"12947265773116138711","method":"GET","headers":{":authority":"talker-api",":method":"GET",":path":"/hello",":scheme":"http","accept":"*/*","authorization":"APIKEY invalid","user-agent":"curl/7.65.3","x-envoy-internal":"true","x-forwarded-for":"10.244.0.11","x-forwarded-proto":"http","x-request-id":"9e391846-afe4-489a-8716-23a2e1c1aa77"},"path":"/hello","host":"talker-api","scheme":"http","protocol":"HTTP/1.1"}},"context_extensions":{"virtual_host":"local_service"},"metadata_context":{}}}
{"level":"debug","ts":1634830373.2070816,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"12947265773116138711","config":{"Name":"keycloak-opaque","ExtendedProperties":[],"OAuth2":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"},"TokenIntrospectionUrl":"http://keycloak:8080/auth/realms/kuadrant/protocol/openid-connect/token/introspect","TokenTypeHint":"requesting_party_token","ClientID":"talker-api","ClientSecret":"523b92b6-625d-4e1e-a313-77e7a8ae4e88"},"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":null},"reason":"credential not found"}
{"level":"debug","ts":1634830373.207225,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"12947265773116138711","config":{"Name":"api-keys","ExtendedProperties":[{"Name":"sub","Value":{"Static":null,"Pattern":"auth.identity.metadata.annotations.userid"}}],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":{"AuthCredentials":{"KeySelector":"APIKEY","In":"authorization_header"},"Name":"api-keys","LabelSelectors":{"audience":"talker-api","authorino.3scale.net/managed-by":"authorino"}},"KubernetesAuth":null},"reason":"the API Key provided is invalid"}
{"level":"debug","ts":1634830373.2072473,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"12947265773116138711","config":{"Name":"k8s-service-accounts","ExtendedProperties":[],"OAuth2":null,"OIDC":null,"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"}}},"reason":"credential not found"}
{"level":"debug","ts":1634830373.2072592,"logger":"authorino.service.auth.authpipeline.identity","msg":"cannot validate identity","request id":"12947265773116138711","config":{"Name":"keycloak-jwts","ExtendedProperties":[],"OAuth2":null,"OIDC":{"AuthCredentials":{"KeySelector":"Bearer","In":"authorization_header"},"Endpoint":"http://keycloak:8080/auth/realms/kuadrant"},"MTLS":null,"HMAC":null,"APIKey":null,"KubernetesAuth":null},"reason":"credential not found"}
{"level":"info","ts":1634830373.2073083,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"12947265773116138711","authorized":false,"response":"UNAUTHENTICATED","object":{"code":16,"status":302,"message":"Redirecting to login"}}
{"level":"debug","ts":1634830373.2073889,"logger":"authorino.service.auth","msg":"outgoing authorization response","request id":"12947265773116138711","authorized":false,"response":"UNAUTHENTICATED","object":{"code":16,"status":302,"message":"Redirecting to login","headers":[{"Location":"https://my-app.io/login"}]}}
```

#### Deleting `AuthConfig` and related API key `Secret`s:

```jsonc
{"level":"info","ts":1634675366.8794427,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"object has been deleted, deleted related configs","authconfig":"authorino/talker-api-protection"}
{"level":"info","ts":1634675366.9552257,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"resource reconciled","secret":"authorino/api-key-1"}
{"level":"info","ts":1634675366.9749846,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"resource reconciled","secret":"authorino/api-key-2"}
```

#### Shutting down the service:

```jsonc
{"level":"info","ts":1634675390.0908618,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Shutdown signal received, waiting for all workers to finish","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
{"level":"info","ts":1634675390.0909622,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"Shutdown signal received, waiting for all workers to finish","reconciler group":"","reconciler kind":"Secret"}
{"level":"info","ts":1634675390.0968425,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"Shutdown signal received, waiting for all workers to finish","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
{"level":"info","ts":1634675390.101609,"logger":"authorino.controller-runtime.manager.controller.secret","msg":"All workers finished","reconciler group":"","reconciler kind":"Secret"}
{"level":"info","ts":1634675390.10166,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"All workers finished","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
{"level":"info","ts":1634675390.1017516,"logger":"authorino.controller-runtime.manager.controller.authconfig","msg":"All workers finished","reconciler group":"authorino.3scale.net","reconciler kind":"AuthConfig"}
```
