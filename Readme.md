# vistecture: Service Architecture Tool

A tool for visualizing and analyzing distributed (micro) service oriented architectures.

![Kiku](templates/example.jpg)

## Define your application architecture:

Describe your architecture in JSON (use .json) or YAML (use .yml). You can do this in two ways
- one json file or
- in multiple json files that are all in one directory. (prefered for structuring bigger definitions)

### Installation:

Download a published release from github:

E.g. for mac:
(For linux use "vistecture-linux" and for windows "vistecture.exe")

```
curl -LOk "https://github.com/AOEpeople/vistecture/releases/download/0.2.beta/vistecture"
chmod +x vistecture

# dowload the templates
curl -LOk "https://github.com/AOEpeople/vistecture/releases/download/0.2.beta/templates.zip"
unzip templates.zip

```


And then discover the command:

```
vistecture help
```

You can also clone the repository and use golang tools.


### Example project definition ( example.yml ):

```
---
name: Ports and Adapters DDD Architecture
applications:
- name: domain
  group: component-internal-bounded-context
  technology: scala
  display:
    bordercolor: "#c922b3"
  summary: Short description
  description: |
        Use markdown to describe the service.
        * one
        * tow
  provided-services:
  - name: domain-objects
    type: inbound-port
  - name: repository-interfaces
    type: outbound-port
  - name: eventpublish-interfaces
    type: outbound-port
  infrastructure-dependencies:
  - type: mysql
- name: application
  group: component-internal-bounded-context
  description: Application Use Cases
  provided-services:
  - name: application-services
    type: inbound-port
    description: Main Application API. Also internaly works with eventpublish-interfaces
    dependencies:
    - reference: domain.domain-objects
    - reference: domain.repository-interfaces
      relationship: uses
  - name: eventpublish-interfaces
    type: outbound-port
  - name: domainEventAdapter
    type: "(logic)"
    description: Listen for (some) domain events. Also internaly works with eventpublish-interfaces
    dependencies:
    - reference: domain.eventpublish-interfaces
      relationship: implements

```


## Usage

Currently the main feature is generating graphviz compatible graph descriptions that can be used by any of the graphviz layouters like this:

Complete Graph
```
> vistecture --config=pathtojson graph | dot -Tpng -o graph.png
```

Graph for one Application and its direct dependencies (including infrastructure dependencies):
```
> vistecture --config=pathtojson graph --application=applicationame | dot -Tpng -o graph.png
```

You can also render a documentation - expecting the dot command is executable for the application it will embedd svg images:

```
> vistecture --config=pathtojson documentation > documentation.html
```

Check for cyclic dependencies and get a first impact analysis:

```
> vistecture --config=pathtojson analyze
```

## Domain Language / Concepts:

This tool defines:

**Application:**
A Application is normaly something that offers one or more service-components (or interfaces).
Normaly a Application is something that is deployed seperate - and has a seperate build and integration pipeline.

- Supported Categories: external (rendered in red)
- Supported Technologies: go, scala, magento, akeneo, php, anypoint, keycloak (they will get a nice icon)

**Service:**
An Application offers services (more specific service components - but we use services here).
An application can offer one or more services.
Services are used by other systems or humans. They can be public or just internal.

**Dependency:**
A Application or a service can have dependencies. You can add dependency on service level to emphazize that the dependency is only required for a certain service.
(This is used for impact analyses).
A depdendency creates a reference to either a application - or more exact to a service. The relation is of a certain relationship type.

**supported releationship types:**
- customer-supplier: (use this where a strong dependency extsis that the supplier delivers whats required by the customer. A stronger collaboration between the teams of the components need to exist.)
- openhost: The used api is designed for integration. Its nice and offers multiple services and the format well thought. Most modern Rest API should fall under this section. This is the default
- published-language:  (like openhost + documentation / api formats are published (documented) )
- conformist: (use this to emphazise also a strong dependency that we need the services provided. But there is no chance to influence the interface - so the downstream component is forced to be conform to whatever is provided - and need to make it work.)
- acl: (Anti coruption layer: If the provided interface is complex or very different from the applications bounded context internal model. The acl emphazizes that the downstream component takes care to isolate his domain with a acl pattern)

(See https://www.aoe.com/tech-radar/strategic-domain-driven-design.html )

**Business Services:**
Several service components are typically composed to business services.
For example an ecommerce shop business service may consist of services from  ecommerce application, login application, search application.


## Todos

-  [X] Graph for single application including infrastructure
-  [ ] Introduce useful resilience pattern types for the dependencies
-  [ ] Introduce Business Services as Composite of Services
-  [ ] Better Impact Analysis for application failures - inlcuing resilience evaluation
-  [ ] Create complete documentation
-  [ ] Generate useful artefacts for infrastructure pipeline (e.g. consul acls, service discovery tests...)

## Development

Use "make all" to build the binaries.

To run test:`

```
go test ./tests/...
```



## Tip

Disable CSP Header in Jenkins to allow inline styles (required for a direct view of the generated documentation as jenkins artefact)
Open Jenkins script console and type:

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
```