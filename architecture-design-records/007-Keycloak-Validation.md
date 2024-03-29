# Validating KeyCloak for AYR

## Context
The choice of authentication and authorisation for AYR was partly dictated by the choice of TDR to use Keycloak for their web app and authentication.

In order to validate this choice, we review the potential choices below (borrowing heavily from the analysis done by TDR) and add a validation process concerned with the impact of KeyCloak on specific use cases for AYR. 

##
Initial mapping of capabilities based on prior research by TDR:

| KEY:   5=Excellent, 1=Poor                | Silhouette & Own Code                                            | Score | Cognito                                                      | Score | Auth0                                                          | Score | Keycloak                                                                                                                                | Score |
|-------------------------------------------|------------------------------------------------------------------|-------|--------------------------------------------------------------|-------|----------------------------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------------------------------|-------|
| Can   customise sign-in                   | Yes, build directly                                              | 5     | Yes; by building a new UI using the Server SDK               | 3     | Not much                                                       | 2     | Yes                                                                                                                                     | 5     |
| Vendor   lock-in concern                  | None                                                             | 5     | Yes; also AWS                                                | 1     | Yes; easier swap than C                                        | 2     | Open source, can be deployed   anywhere                                                                                                 | 5     |
| Gradated   Access component reuse options | Build own service so more work   investment long-term            | 3     | Yes                                                          | 5     | Unknown, but plausible                                         | 3     | Unknown, but plausible                                                                                                                  | 3     |
| Dependency   maintenance / sustainability | Long-term commitment ongoing,   sole risk                        | 1     | Dependent on using same   approach, otherwise 5, shared risk | 4     | Know it's used elsewhere,   complexity unknown but shared risk | 4     | This is third party software so   the development risk is not ours but we do have to host it and host the   database                    | 2     |
| Know   where is data stored               | TNA / AWS Region                                                 | 4     | London / AWS Region                                          | 4     | Eire? Brexit risks                                             | 3     | The data is stored in AWS so we   control exactly where it is stored                                                                    | 4     |
| Data   Risk Management                    | All our risk & work to   maintain & recover                      | 1     | shared risk - AWS best practice   support                    | 4     | shared risk, unknown config work                               | 3     | Shared risk - Data maintenance   is carried out by the application. Recovery is our risk                                                | 2     |
| 2FA   & Password resets                   | All own work to provide                                          | 3     | Existing options                                             | 5     | Existing options                                               | 4     | Existing options including SMS   for no extra cost                                                                                      | 5     |
| Config                                    | Yes, build directly, but time   required initially               | 4     | Complicated (but SH made it work   in 2days)                 | 3     | Unknown, but plausible                                         | 3     | Main app runs in a docker   container. Could be deployed to ECS in a couple of days                                                     | 2     |
| Easier   for Dev Ops to support           | At face value yes, but small   risk that not everything is known | 4     | Easy to auth, but more fiddly                                | 3     | Easier with login but risk of   also being fiddly              | 4     | This is no more difficult to   support than our existing ECS services but it is another service to support.                             | 3     |
| Works   with other clients                | Silhouette (2 trips)                                             | 3     | Has more API requests                                        | 3     | Poss balanced but untested                                     | 4     | Keycloak supports separate   clients and auth flows in the same way as auth0 so I'll give it the same   score                           | 4     |
| Changing   our minds                      | Yes with work                                                    | 3     | Yes with work                                                | 3     | Yes with work, easier to move to   Octa                        | 3     | As per Suzanne's comment, this   is another OIDC provider so switching is the same for all of them                                      | 3     |
| Cost   effective in use                   | Time, but not tied into   contracts                              | 4     | Free for now, <50k users                                     | 4     | Most expensive, £100pm, poss   more                            | 2     | £70-80 per month for hosting                                                                                                            | 3     |
| Total   cost of ownership long term       | Entire risk & dev time                                           | 1     | Shared risk & dev                                            | 3     | Shared risk & dev                                              | 3     | Shared risk & dev                                                                                                                       | 3     |
| Standard   Tech components                | Some                                                             | 3     | Some                                                         | 3     | Some                                                           | 3     | Some                                                                                                                                    | 3     |
| Other   dept usage                        | Own bespoke build                                                | 1     | Digi services in use with   limitations                      | 2     | Not aware of use                                               | 3     | This is also used in other gov   depts so I've raised Auth0 to 3 and set keycloak to be the same                                        | 3     |
| Failure   impact of component             | Risks, but own build to fix                                      | 2     | Risks, but more known                                        | 2     | Risks, unknown                                                 | 1     | We have full control over the   upgrade cycle of the software and the hosting. They are our risks but we have   full control over them. | 3     |
| Security   risk mitigation                | Unknown, but should be able to   build with time                 | 3     | Known token issue threat                                     | 4     | Seems most secure                                              | 5     | As secure as auth0, it is an   OIDC provider                                                                                            | 4     |
| Supports   GDPR                           | We need work to prove                                            | 3     | Established                                                  | 5     | Established                                                    | 5     | We need work to prove                                                                                                                   | 3     |
| Adheres to GDS guidance for choosing a tech| On balance                                                      | 3     | On balance                                                   | 3     | On balance                                                     | 3     | On balance                                                                                                                              | 3     |
| Supports   WCAG                           | Can build what's needed with   time                              | 5     | Lack of customisation                                        | 3     | Limited customisation                                          | 3     | Complete customisation including   existing themes for the GDS design system                                                            | 5     |
| Total   Scores                            | Silhouette with own                                              | 61    | Cognito                                                      | 67    | Auth0                                                          | 63    | Keycloak                                                                                                                                | 68    |

[gds guidance]: https://www.gov.uk/service-manual/technology/choosing-technology-an-introduction
[auth providers]: https://www.silhouette.rocks/docs/config-introduction
[Silhouette]: https://www.silhouette.rocks/
[SAML]: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-saml-idp.html
[OpenID Connect]: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-identity-provider.html#cognito-user-pools-oidc-providers
[here]: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-ui-customization.html
[sign in]: images/cognito-sign-in.png
[getting started]: https://aws.amazon.com/cognito/dev-resources/

## KeyCloak and Access Your Records Requirements

| KEY:   5=Excellent, 1=Poor                | Score | 
|-------------------------------------------|-------|
| Django integration						            |  3    |
| Federated identity with TDR               |  5    |
| AYR Permission model                      |  4    |
| AYR Search engine integration             |  5    |
| AYR Data permissions for bag-its          |  4    |

The main benefits in using KeyCloak remain its suitability for integration with TDR in providing federated identity for users. 

It also provided the basis for a permission model that meets the identified roles for AYR users (Frequenter, Back-Stager and Front-liner).

It remains at this stage of the alpha an open question whether Django is the best fit in terms of web application framework. Integration with KeyCloak has raised questions around the availability of libraries / tools for this integration as well as the best place to model fien-grained permission control.

The potential for integration with OpenSearch and processes for data retrieval from TRE also seem to speak in favour of KeyCloak as a solution. However, here any other OpenID Connect Provider solution would likely work just as well.
