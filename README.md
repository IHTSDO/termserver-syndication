# Terminology Server Syndication Standard - DRAFT

**Everything here is currently draft and should not be considered to be agreed as a standard nor should it be implemented yet**

As part of a strategy to move towards wider adoption of healthcare terminology servers which implement the HL7 FHIR Terminology Services, a more effective way of distributing terminology would be from server to server through a syndication feed.

Based on existing work implemented by the [Australian Digital Health Agency](https://www.healthterminologies.gov.au/specs/v2/conformant-server-apps/syndication-api/syndication-feed/), this feed extends the [Atom Syndication Standard](https://tools.ietf.org/html/rfc4287). It is not specific to SNOMED CT and can be used for any relevant healthcare standard,

The additions here are from SNOMED International and are in the sct namespace, http://snomed.info/syndication/sct-extension/1.0.0, for distributing SNOMED CT content. 

## Details

Example feed can be seen in this repo - [feed-example.xml](feed-example.xml)

The extensions to the specification are as follows:

```
<sct:packageDependency>
     <sct:editionDependency></sct:editionDependency>
     <sct:derivativeDependency></sct:derivativeDependency>
</sct:packageDependency>
```

These are SNOMED CT specific extensions to specify any package dependency for a given release package that are not included in the package itself. Dependencies can be on **editions** or other **derivatives**.

[Edition packages](https://confluence.ihtsdotools.org/display/DOCGLOSS/edition) by definition include all their dependencies and therefore do not have a `sct:packageDependency` element.

[Extension packages](https://confluence.ihtsdotools.org/display/DOCGLOSS/extension) always depend on other packages' content not included in the extension package and therefore always have a `sct:packageDependency` element.

[Edition packages](https://confluence.ihtsdotools.org/display/DOCGLOSS/edition) and [Extension packages](https://confluence.ihtsdotools.org/display/DOCGLOSS/extension) can therefore be distinguished by the absence or presence of a `sct:packageDependency` element.

Within a `sct:packageDependency` element
- **sct:editionDependency** states any necessary edition dependency and version using the SNOMED CT URI standard, e.g. `http://snomed.info/sct/900000000000207008/version/20220731`. Where a package is transitively dependent on multiple editions, only the direct non-transitive dependencies should be stated. Downloading packages would complete when there are no further dependencies.
- **sct:derivativeDependency** states any necessary derivative dependency, as they are often published outside of SNOMED CT editions. Similarly, this should also follow the SNOMED CT URI standard for module and version

### Naming conventions
To aid human readability and distinguishing between packages, the following naming conventions should be applied.
1. Edition packages
     1. Should contain the word edition
     2. Should NOT contain the word extension
2. Extension packages
     1. Should contain the word extension
     2. Should NOT contain the word edition
3. The version of the package should be included in the title
4. The [RF2 Release Type](https://confluence.ihtsdotools.org/display/DOCRELFMT/3.2+Release+Types) Full/Delta/Snapshot should be included as a bracketed suffix to the title

For example, titles following these conventions are
- SNOMED CT-International Edition 2022-07-31 (RF2 SNAPSHOT)
- SNOMED CT-International Spanish Extension 2022-10-31 (RF2 FULL)

## Authentication
The standard does not mandate any authentication and this is left to the implementer of the Atom feed provider to implement whatever is needed depending on any terminology product license requirements.

## Use Cases

Syndication of terminologies has a number of use cases. A few examples are:

Automatic distribution of content products by their owners:

- International content products
- National content products
- Regional content products

Automatic import of selected products and dependencies as updates become available:

- Automatic update of existing terminology server instances
  - Browser server containing both published content and daily builds
- Automatic provision of new terminology server instances
  - Autoscaling
