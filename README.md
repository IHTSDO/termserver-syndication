# Terminology Server Syndication Standard

As part of a strategy to move towards wider adoption of healthcare terminology servers which implement the HL7 FHIR Terminology Services, a more effective way of distributing terminology would be from server to server through a syndication feed.

Based on existing work implemented by the [Australian Digital Health Agency](https://www.healthterminologies.gov.au/specs/v2/conformant-server-apps/syndication-api/syndication-feed/), this feed extends the [Atom Syndication Standard](https://tools.ietf.org/html/rfc4287). It is not specific to SNOMED CT and can be used for any relevant healthcare standard,

The additions here are from SNOMED International and are in the sct namespace, `http://snomed.info/syndication/sct-extension/1.0.0`, for distributing SNOMED CT content.

## Details

### SNOMED CT Specific extensions

A live feed is now available from SNOMED International - [Members Distribution and Licensing Service](https://mlds.ihtsdotools.org/api/feed)

Example feed can be seen in this repo - [feed-example.xml](feed-example.xml).


The extensions to the specification are as follows:

``` xml
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

### Edition/Extension/Derivative feeds

While all types of entries can be put into one feed, it is also possible to host these entries in separate feeds. For examples

- an editions feed - edition packages of available SNOMED CT Editions/Extensions
- an extensions feed - extension packages of available SNOMED CT Extensions
- a derivatives feed - extension packages of available SNOMED CT Derivatives

Given that most implementers prefer a ready to use package to parts and assembly instructions, most implementers will be interested in edition packages rather than extension packages.

Providing a feed containing only edition packages without extension packages would be the least confusing entry point for most implementers with the least learning and decision making required.

Additional feeds for extension packages and derivatives would be useful for other implementations and are still worth providing.

A complete feed that is the union of these three feeds can also be potentially provided.
  
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
4. The [RF2 Release Type](https://confluence.ihtsdotools.org/display/DOCRELFMT/3.2+Release+Types) Full/Snapshot should be included as a bracketed suffix to the title

For example, titles following these conventions are

- SNOMED CT-International Edition 2023-04-30 (RF2 SNAPSHOT)
- SNOMED CT-International Spanish Extension 2023-10-31 (RF2 FULL)

### File links

In each feed entry should always have one `link rel="alternate"` link. This will generally be the SNOMED CT package to download.

There can also be one or more `link rel="related"` entries for any related files, including Release Notes or other relevant files.

Each link will have the following attributes:

- `type` - the type of file being downloaded, e.g. `type="application/zip"` or `type="application/pdf"`
- `href` - the URL link to download the file
- `length` - the size of the file in bytes, e.g. `length="533422481"`
- Checksum validity. **either** of the two following values can be used (chosen by the distributor):
  - `sct:md5Hash` the MD5 hash function producing a 128-bit hash value used to check the validity of the file, e.g. `sct:md5Hash="0290ad7f6e431063166afd91cd2b8c37"`
  - `ncts:sha256Hash` the SHA256 hash function producing a 256-bit hash value used to check the validity of the file, e.g. `sha256Hash="902ec61f6ada329bfd6bc487fe7adab0"`
 
### RF2 Package Types

In each feed entry, the **category** will indicate the type of RF2 package. The expected types are as follows:

- **SCT_RF2_ALL**: Containing the Snapshot and Full RF2 release files (and potentially the Dela Rf2 files but this is not mandatory)
- **SCT_RF2_FULL**: Containing only Full RF2 release files
- **SCT_RF2_SNAPSHOT**: Containing only Snapshot RF2 release files
- **SCT_RF2_DELTA**: Containing only Delta RF2 release files

Most available RF2 packages will contain all the necessary release files and will, therefore, have a feed entry item as follows:
```
<category term="SCT_RF2_ALL" label="SNOMED CT RF2 All" scheme="http://ns.electronichealth.net.au/ncts/syndication/asf/scheme/1.0.0" />
```

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
