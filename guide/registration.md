---
layout: page
title: Registration and Deployment
permalink: /guide/registration
parent: Step-by-step Guide
nav_order: 3
---

# Registration and Deployment

{: .no_toc }

When the MAS middleware is finished, the DiSSCo team will help you register your Machine Annotation
Service. The Orchestration Service is a DiSSCo service that creates deployment files for the MAS
middleware. Once a MAS is registered, it will be available in the DiSSCo test environment. To
register a MAS, the DiSSCo team needs the following information.

- TOC
{:toc}

# Create a pull request

MASs are all available on our
MAS [GitHub repository](https://github.com/DiSSCo/demo-enrichment-service-image). When your service
is ready, you should create a pull request on this repo with your new service. Your pull request
should meet the following requirements:

- **Code Quality** : Code should be run through a code quality checker ("linter"). By default, the
  DiSSCo pipeline runs code through the linter SonarQube and requires 0 code smells to move forward.
- **Requirements.txt**: Please include a `requirements.txt` file
- **Valid Dockerfile**: Your service must include a valid Dockerfile
- **Workflow file**: Include a GitHub workflow yaml file for your service in the `.github/workflows`
  directory. It shoud be based
  on [existing workflow files](https://github.com/DiSSCo/demo-enrichment-service-image/blob/main/.github/workflows/osm-georeferencing-pipeline.yml)

## Deploying with Docker

When you've completed developing your MAS middleware, you're ready to containerize it. These
services are deployed as a container image in DiSSCo. As such, make sure
there is a valid Docker file in the source code.

Pushing to the main branch triggers GitHub actions. During this pipeline, the image is built and
deployed to the DiSSCo AWS container registry. This image is then managed by kubernetes and is
launched when a user schedules your service.

{: .warning}
DiSSCo is only responsible for the deployment of the MAS middleware. The value service
remains the responsibility of the developer.

# Environmental Variables and Secrets

If you use environmental variables or secrets in your MAS middleware, you need to send them to
DiSSCo so they can be injected into the application.

{: .note}
>
> **Enviromental variables** can be used to set parameters for an algorithm or feature toggle
> specifics
> parts of the MAS middleware. They are not encrypted and can be read by anyone once the MAS is
> registered.
>
> **Secret variables** are often used to authenticate with a service or are otherwise values you
> don't want exposed. They are encrypted and - naturally - are not publicly available.

For environmental variables, you can send the DiSSCo team the value of the variable and the name of
the variable as it is used in code.

For secret variables, we encrypt the values and inject them into the application. For this, the
DiSSCo development team adds the secrets to the DiSSCo Secret Store on AWS. This is a
manual action which hasn't been automated (yet).

The secret should first be securely provided to the DiSSCo development team, **along with the name
of the variable as it is used in the code**. For the transfer of the secrets, several options are
available, all of which should include a two-factor authentication.
Examples include:

- A zip-file can be secured with a password and send via email. The password should then come
  through a different medium
  such as a text message or a chat message.
- It is also possible to use a website as https://onetimesecret.com/ preferably with a passphrase
  which is provided through a different
  medium.

The DiSSCo team will then insert the secret into our Secret Store. The variable can then be injected
into the program as an environmental variable.

## Example

Alice wants to send the DiSSCo team a password used in her code. She has the following `.env` file:

```text
API_KEY=2bbd58fe-2654-4049-be60-4f1302a76c53
API_ENDPOINT=https://api.code.com/specimens
```

Alice sends the value of `API_KEY` through https://onetimesecret.com/, and includes the password in
another email. Because it is not sensitive information, she can send the value of `API_ENDPOINT` by
email. At the same time, she also informs the DiSSCo of the name of the variables.

The DiSSCo team inserts the `API_KEY` in the secret store. Registering the MAS, the DiSSCo team
includes the environmental variables and secrets, which

The resulting kubernetes file will thus have all the information needed to access the secret and
inject it into the running service. The following yaml will be generated and used in the deployment
of the MAS.

```yaml
env:
  - name: API_ENDPOINT
    value: https://api.code.com/specimens
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        key: alice-api-key
        name: mas-secrets
```

Where `secretKeyRef` references the name of the secret (not the value) and the name of the secret
key store in AWS.

When the MAS is registered, the environmental variables and secrets will appear through the API as
the following:

```json
{
  "ods:hasEnvironmentalVariables": [
    {
      "schema:name": "API_ENDPOINT",
      "schema:value": "https://api.code.com/specimens"
    }
  ],
  "ods:hasSecretVariables": [
    {
      "schema:name": "API_KEY",
      "ods:secretKeyRef": "alice-api-key"
    }
  ]
}
```

# Filters

The filters you determined in the [data modelling](/mas-developers-documentation/guide/data-model)
step must be sent to DiSSCo.

# Batching Permitted

MAS providers can opt to enable their service for batch annotations, a feature particularly useful
for services that can be applied to multiple specimens simultaneously. Batch annotations help reduce
computational overhead by allowing DiSSCo to identify resources with identical input data. Instead
of running the MAS separately for each resource, DiSSCo applies the annotation across all relevant
resources, streamlining the process and conserving resources.

However, it's important to note that a MAS must be specifically designed with batching in mind, as
it requires additional metadata to support this functionality. For detailed guidance on developing a
batch-enabled MAS, please refer to
the [batching](/mas-developers-documentation/guide/data/#batching-annotations) notes in the data
modelling section.

# Other Metadata

Additional metadata is useful for providing users with more information about the service. Please
provide as many of the following terms as possible.

- name of the service (required)
- description
- creativeWorkStatus - The current status of the service
- codeRepository - Link to code base of MAS
- programmingLanguage
- serviceAvailability - Availability commitment of the service provider as described in the SLA
- maintainer (Follows Agent data model)
- license
- Contact point (Description, email, url, phone)
- SLA Documentation

{: .note}
See
the [MAS JSON Schema](https://schemas.dissco.tech/schemas/fdo-type/machine-annotation-service/latest/machine-annotation-service.json)
or [MAS Terms Site](https://terms.dissco.tech/machine-annotation-service-terms)
for more information on the MAS data model.

# Moving to production - Service Level Agreements (SLA)

In order to move your MAS to the production environment, an SLA must be established between DiSSCo
and your organisation. This procedure is still in development - check back soon for more updates. 

# Moving Forward - Checklist

- You've successfully containerized your MAS using Docker
- You've sent the DiSSCo team your environmental variables and (securely!) sent your secrets
- You've sent the DiSSCo team the filters associated with your MAS
- You've sent the DiSSco team any additional metadata about your MAS

All set! The DiSSCo team will deploy your MAS middleware, and you should be able to use your MAS in
the sandbox environment! 