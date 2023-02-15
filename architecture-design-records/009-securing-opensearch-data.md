# Securing OpenSearch Data

This document explores options for securing AYR records at different levels of
the application stack.

OpenSearch [Document-Level Security](https://opensearch.org/docs/latest/security/access-control/document-level-security/)
(DLS) could be used to secure access to AYR records at the lowest level, but
there appear to be limitations with how 3rd party authentication providers
such as Keycloak can be used with the AWS managed OpenSearch service.

## Options

The following diagram shows the stack being considered for this evaluation:

```
       +------+
       | User |
       +------+
           |
+---------------------+                                  +----------+
|    Application      |------- username/password ------->| Keycloak |
| (WebApp, curl, ...) |<---------- token ----------------|          |
+---------------------+                                  |          |
     .             |                                     |          |
     .           token                                   |          |
     .             |                                     |          |
     .        +----------+                               |          |
  token       | REST API |-------- token --------------->|          |
(DLS only)    +----------+                               |          |
     .             |                                     |          |
     .      either token (if using DLS)                  |          |
     .      or credentials (if not using DLS)            |          |
     .             |                                     |          |
   +-----------------+                                   |          |
   |   OpenSearch    |------------ token --------------->|          |
   +-----------------+           (DLS only)              +----------+
```

Options to control user access to OpenSearch data (with and without use of the
OpenSearch DLS) are:

<table>
    <tr>
        <th rowspan="2" colspan="2">Configuration Option</th>
        <th colspan="2">OpenSearch Deployment Model</th>
        <th rowspan="2">Notes</th>
    </tr>
    <tr>
        <th>Self Hosted</th>
        <th>AWS Managed Service</th>
    </tr>
    <tr>
        <td>1</td>
        <td>Internal OpenSearch users/groups without DLS</td>
        <td>✅</td>
        <td>✅</td>
        <td>Easy deployment, but REST API can see all OpenSearch records and must apply group filtering using Keycloak token; OpenSearch must be a secured endpoint that only the REST API service can access</td>
    </tr>
    <tr>
        <td>2</td>
        <td>External Keycloak user/group configuration without DLS</td>
        <td>✅</td>
        <td>✅<br><sup>❗But not API</sup></td>
        <td>AWS managed OpenSearch may support SAML (and perhaps not OIDC) 3rd party authentication integration for the OpenSearch Dashboard only; OpenSearch API access is not supported, which precludes DLS support</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Internal OpenSearch user/group configuration with DLS</td>
        <td>✅</td>
        <td>✅<br><sup>❗Duplicates Keycloak config</sup></td>
        <td>Would require mechanism (manual or automated) to keep internal OpenSearch user/group model in sync with external Keycloak user/group model</td>
    </tr>
    <tr>
        <td>4</td>
        <td>External Keycloak user/group federation with DLS</td>
        <td>✅</td>
        <td>❌</td>
        <td>Federated user access to the OpenSearch API does not appear to be supported with AWS-hosted OpenSearch</td>
    </tr>
</table>

## Option Selection

Option 1 is currently the preferred approach for the following reasons:

* Using an AWS-managed OpenSearch service is preferred over the complexity of
deploying and maintaining a self-hosted OpenSearch deployment; this leaves
options 1, 2 and 3 as possible solutions
* Option 3 can be excluded as the required synchronisation of user and group
configuration from Keycloak to OpenSearch is duplication
* Option 2 can be excluded as authentication support is not at the API level
* Until such time AWS Managed OpenSearch supports DLS at the API level, the
best approach is currently deemed to be option 1 (filtering records at the API
level)
