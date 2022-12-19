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

### Snapshot Release Type for extension packages
Extensions, as opposed to derivatives, add/modify core components - concepts, descriptions and axioms/relationships. These changes affect the Snapshot state of components, as well as potenitally affecting classification and Necessary Normal Form calculation (for example non-leaf concept addition).

An Extension (adding/modifying core components) published as an extension package should only be a Delta or a Full [RF2 Release Type](https://confluence.ihtsdotools.org/display/DOCRELFMT/3.2+Release+Types). Such a package can be applied to a referenced base edition, and then a Snapshot calculated using the [Module Dependency Reference Set](https://confluence.ihtsdotools.org/display/DOCRELFMT/5.2.4.2+Module+Dependency+Reference+Set). Because an Extension affects the Snapshot state in this way, potentially requiring reclassification and Necessary Normal Form recalculation, a Snapshot extension package of an Extension cannot be simply appended to a base edition's Snapshot. For this reason, the Snapshot Release Type for extension packages of SNOMED CT Extensions should not be used.

In the simpler Derivative case (map, refset, even language translation with additional descriptions) a Snapshot extension package being simply appended to a Snapshot base edition is safe and simple because the derivative simply adds new components and does not affect the state of existing components.

The Snapshot Release Type for an edition package does not have these issues as the base edition content is included in the resolved Snapshot Release Type of the package.

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

### Packaging types for extensions and derivatives
The major distinction between extensions and derivatives is that extensions add/modify core components, where derivatives are limited to reference sets providing non-core "bolt on" content.

#### Extensions
For editions, most implementers want a ready to use package rather than parts and assembly instructions. Assembling an edition from an extension package and a base edition is not a trivial exercise, requiring content to be combined and a snapshot calculated based on the [Module Dependency Reference Set](https://confluence.ihtsdotools.org/display/DOCRELFMT/5.2.4.2+Module+Dependency+Reference+Set).

For this reason *edition packaging is preferred as the primary distribution method extensions*. Extensions may also be provided in extension packaging, however if only one is made availabe edition packaging is preferred.

The rationale for this recommendation is that 
- implementers desiring an extension package are in the minority and more capable of disassembiling an edition package if necessary
- the majority of implementers would prefer a ready to use edition package, and the task of assembling that from an extension and base edition package and calculating a snapshot based on the [Module Dependency Reference Set](https://confluence.ihtsdotools.org/display/DOCRELFMT/5.2.4.2+Module+Dependency+Reference+Set) is burdensome.

#### Derivatives
Derivatives are designed to bolt on to an edition, and as such are either "pre-mixed" into an edition package or provided as an extension package.

When provided by themselves, they make most sense provided as an extension package so they can be "bolted onto" a number of different compatible editions. Therefore the *preferred packaging of a derivative is an extension package*.

Note that this preference does not preclude prepackaging one or more derivatives in edition packages where useful.

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
