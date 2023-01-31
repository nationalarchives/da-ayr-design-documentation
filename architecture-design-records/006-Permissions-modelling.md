# Permissions Modelling

Date: 31-01-2023

## Status

In Progress

## Context

### Keycloack authorisation system

Keycloak is an open-source identity and access management solution that works by establishing a secure communication
channel between a client (e.g. a web application) and a server.

Once the user logs in, Keycloak creates an access token, which contains information about the user's identity and the
resources they have access to. The client can then use this token to make decisions on whether the user can access
protected resources on the server.

Keycloack offers different authorisation strategies, which can also be combined to model complex configurations.
The main ones are:

* **RBAC** - Role Based Access Control
* **GBAC** - Group Based Access Control
* **ABAC** - Attribute Based Access Control

#### **RBAC**

Keycloak implements RBAC by assigning roles to users, which represent a set of permissions for accessing certain
resources.
These roles are then mapped to specific resources, so that users with the appropriate role can access those resources.
For example, an administrator role might be granted access to all resources, while a user role might only have access to
a subset of resources.

#### **GBAC**

In Keycloak, you can grant roles to groups. That is a powerful capability where members of a group are automatically
granted roles for the group they belong to.
Different from roles, group information is not automatically included in tokens. For that, you should associate a
specific protocol mapper with your client (or a client scope with the same mapper).

#### **ABAC**

Attribute based access control involves using the different attributes associated with an identity (represented by a
token), as well as
information about the authentication context, to enforce access to resources. It is probably the most flexible access
control mechanism.

### The three archetypes

The three archetypes describe three different scenarios where users have access to different resources.

* Frequenter - a user from a government department, who needs access to their own records and metadata. Example: User
  A (external to TNA) from Y Department must be able to access records from Y Department.

* Back-stager - a user from a government department who needs access to their own records and metadata, and from another
  department. Example: User B  (external to TNA)  from Z Department has access to records from Z Department. Due to an
  inquiry, they
  will also need access to records from X Department for a period of time.

* Front-liner - a user from TNA, who needs access to records and metadata across all departments. Example: User C (
  internal to TNA) from TNA’s FOI team gets an FOI request. They need to log in and access all the
  records we hold from all departments.

The final question that the system will need to answer is:
Can the user access records and metadata from government department X?

The general property of a role mapped to a group is: to be able to access data from their own group.

## Decision

Based on my current understanding, for a first iteration I think a reasonable choice would be to create groups, where
each group represents a government department.
Each group will be then assigned to a role that has access to the resources of that group.

As a general example, a user that belongs to Government Department X will have only access to

www.ayr.com/department/x/records
www.ayr.com/department/x/metadata

Any attempt to access any other resource should result in a `401 Not Authorised error`

This describes the "frequenter" use case.

The "back-stager" is simply a user that belongs to their own department group as well as other groups, which are unique
for each user.

The "frontliner" is a user that belongs to all the groups.

## Consequences

As previously mentioned, the ideal solution would be to have Keycloack exclusively control access to resources.
The general workflow is:

* Keycloack issues a token to the client.
* The client request access for a combination of resource/user
* Keycloack gives the appropriate response.

While this seems to be the preferred approach, it will require further work, as the library in use for authentication
 -- [mozilla-django-oidc](https://mozilla-django-oidc.readthedocs.io/en/stable/index.html) --
does not offer authorisation capabilities out the box. Moreover, Django authorisation system would be completely bypassed,
raising the question on whether Django would be the best framework for this type of setup.

A hybrid approach is also possible, where Django authorisation system makes decision based on Keycloack authentication response.
Access control is still managed in Keycloack.
