---
title: 'DBCLS BioHackathon 2025 report: Towards Systematic Matching of RDF Life Sciences Databases through Automatically Extracted Schemas'
title_short: 'BioHackJP25:shapes-alignment'
tags:
  - Semantic web
  - RDF Schemas
  - Shape Expressions
  - Schema Matching
  - SPARQL
  - Interoperability
authors:
  - name: Javier Millán Acosta
    affiliation: 1
    orcid:0000-0001-5608-781X
  - name: Last Author
    orcid: 0000-0000-0000-0000
    affiliation: 2
affiliations:
  - name: Department of Translational Genomics, Maastricht University, Maastricht, The Netherlands
    index: 1
    ror: 02jz4aj89
date: 15 September 2025
cito-bibliography: paper.bib
event: BH25JP
biohackathon_name: "DBCLS BioHackathon 2025"
biohackathon_url:   "https://2025.biohackathon.org/"
biohackathon_location: "Mie, Japan, 2025"
group: shapes-alignment
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/BH25-shapes-alignment
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Millán Acosta \emph{et al.}
---
# Abstract

Life sciences databases can be exposed in RDF to Even if the underlying ontologies are shared, different Resource Description Framework (RDF) resources present
As part of the DBCLS BioHackathon 2025, we here report our work on a methodology to systematically retrieve Shape Expressions from different RDF resources to describe their actual abstract syntax and match them to compare the similarity of their structures. This allows to identify gaps and opportunities for the interoperability of different RDF resources and facilitate data integration tasks like data retrieval via federated SPARQL querying across different triplestore databases and the continuous integration of asyncronously developed RDF graph resources.

## Author information

# Introduction

Shape languages guide RDF resource authoring and integration through the prescription of the conditions required to validate them in terms of their abstract structural components or abstract syntax. The Shapes Constraint Language (SHACL) and Shape Expressions (ShEx) language both use sets of constraints to formally express the topology of RDF nodes and their properties in RDF data [@citesAsAuthority:shaclSpec;@citesAsAuthority:shexSpec]. While they use different syntax and expressivity, they are broadly intertranslatable for high-level schema matching tasks [@obtainsSupportFrom:commonFoundations].

However, the adoption of shape languages in the life sciences RDF resources landscape is still limited: shape languages are not yet part of the standard development and maintenance workflows of most RDF resources in the life sciences domain, which hinders their interoperability and integration.

The main challenges in RDF schema matching and interoperability can be categorized into two key areas:

**Consistency**: How reliably do  a particular shape pattern appear across an RDF graph? This relates to the trustworthiness and consistency scores associated with automatically extracted shapes.

**Alignment**: How similar are shapes between different RDF resources? This addresses the identification of common design patterns and structural similarities across databases. The main challenge is detecting and quantifying these design patterns to enable meaningful comparison and integration between heterogeneous RDF sources.

In the recent years, some work has addressed the automatic generation of shapes and shape expressions [rdf-config, shexer, more refs] to formalize the actual structure of a dataset even if shapes validation is not part of its development or maintenance workflows, which opens possibilities to retrieve shape schemas in automated workflows for continuous integration tasks.

Discuss work below shortly:

- sheXer paper  [@citesAsDataSource:shexer] explains the algorithm and python implementation to mine RDF graphs for their shape expressions.
- ELIXIR and Japan BioHackathon: 
 - [@citesAsRelated:bh23Integration]:
 - [@citesAsRelated:bh23Enhancement]:
 - [@citesAsRelated:bh24Toward]:
 - [@citesAsRelated:bh24Discovered]:


# Shape Expression comparison
Steps:
- Map namespaces and ontology IRIs in RDFPortal (Yasunori's work)
- Add feature to sheXer to serialize machine-readable trustworthiness scores for node and triples constraints
- Extract shapes with sheXer
- Add feature to rudof to compare the intersection of two shapes and return a similarity score
- Apply to use cases
- analyze results


## Challenges

-
- ebi xref restful api (Renato): 

# Use cases

## EBI pathways and uniprot proteins --> No sparql for reactome

## diseases in MESH, OMIM <-- pubchem <-- uniprot

ebisearch, diff domains . Links between domains --> 

PRIDE RDF

## Pubchem and SIB around Protein, cross-domain aspects: 

https://pubchem.ncbi.nlm.nih.gov/docs/rdf-federated-query#

- UniProt

- 

## MESH and PubChem RDF

```
- PubChemInchiKey inchikey:AAAHDTMMJGSHKG-UHFFFAOYSA-N:
  ## pc_inchikey_type_*.ttl
  - a: sio:CHEMINF_000399  # InChIKey generated by software version 1.0.4
  ## pc_inchikey_value_*.ttl
  ### This language tag has not been removed somehow.
  - sio:SIO_000300:
    - inchikey_has_value: '"AAAHDTMMJGSHKG-UHFFFAOYSA-N"@en'
  ## pc_inchikey2compound_*.ttl
  - sio:SIO_000011:
    - inchikey_is_attribute_of: PubChemCompound
  ## pc_inchikey_topic.ttl
  - dcterms:subject:
    - inchikey_topic: mesh:M0000001
```

```
- PubChemDisease disease:DZID2725:
  - a: obo:DOID_4         # Disease
  - a: obo:MONDO_0000001  # disease or disorder  ## Both of these are assinged.
  - skos:prefLabel:
    - disease_pref_label: "Chromosome 18 ring"
  - skos:altLabel*:
    - disease_alt_label: "ring 18 chromosome syndrome"
  - skos:closeMatch*:
    # UMLS, OMIM, Orphanet, MedGen, MeSH, GARD, NCIT, Mondo, DOID, HP
    - disease_close_match: <https://uts.nlm.nih.gov/uts/umls/concept/C0265475>
  - skos:relatedMatch*:
    # UMLS, OMIM, Orphanet, MedGen, MeSH, GARD, NCIT, Mondo, DOID, HP
    - disease_related_match: <https://uts.nlm.nih.gov/uts/umls/concept/C2931809>
```

  ```
  - PubChemReference reference:PMID10021376:
  ## pc_reference_contenttype.ttl
  - prism:contentType:
    - reference_content_type: "Letter"
  ## pc_reference_title*.ttl
  - dcterms:title:
    - reference_title: '"Intracellular signalling: PDK1--a kinase at the hub of things"@en'
  ## pc_reference2chemical_disease*.ttl
  - cito:discusses*:
    - reference_discusses: <http://id.nlm.nih.gov/mesh/M0011758>
  ## pc_reference_citation*.ttl
  - dcterms:bibliographicCitation:
    - reference_bibliographic_citation: "C Belham, S Wu, J Avruch; Current biology : CB; 1999 Feb; 9(3):R93-6"
  ## pc_reference_date.ttl
  - dcterms:date:
    - reference_date: 1999-02-11-04:00
  ## pc_reference_author*.ttl
  - dcterms:creator:
    - reference_creator: PubChemAuthor
  ## pc_reference2meshheading*.ttl
  - fabio:hasSubjectTerm*:
    - reference_subject_term: <http://id.nlm.nih.gov/mesh/D000818>
  ## pc_reference2meshheading_primary
  - fabio:hasPrimarySubjectTerm*:
    - reference_primary_subject_term: <http://id.nlm.nih.gov/mesh/D011499>
  ## pc_reference_discusses_by_textming_*.ttl
  - vocab:discussesAsDerivedByTextMining*:
    - reference_disscusses_as_derived_by_text_mining: [PubChemCompound, PubChemGene, PubChemDisease]
  ## pc_reference_startingpage*.ttl
  - prism:startingPage:
    - reference_starting_page: "2667"
  ## pc_reference_endingpage*.ttl
  - prism:endingPage:
    - reference_ending_page: "2691"
  ## pc_reference_pagerange*.ttl
  - prism:pageRange:
    - reference_page_range: "2667-2691"
  ## pc_reference_fundingagency*.ttl
  - frapo:hasFundingAgency:
    - reference_funding_agency: PubChemOrganization
  ## pc_reference_grant*.ttl
  - frapo:isSupportedBy:
    - reference_grant: PubChemGrant
  ## pc_reference_identifier*.ttl
  ### Can dcterms:identifer have a URI as an object?
  - dcterms:identifier:
    - reference_identifier: <https://doi.org/10.2210/pdb4pwh/>
  ## pc_reference_issn*.ttl
  - prism:issn:
    - reference_issn: "1756-6606"
  ## pc_reference_issue*.ttl
  - prism:issueIdentifier:
    - reference_issue_identifier: "1"
  ## pc_reference_journal_book*.ttl
  - dcterms:isPartOf:
    - reference_journal_book: [PubChemJournal, PubChemBook]
  ## pc_reference_lang*.ttl
  - dcterms:language:
    - reference_language: "English"
  ## pc_reference_publication*.ttl
  - prism:publicationName:
    - reference_publication: "American Journal of Potato Research"
  ## pc_reference_source*.ttl
  - dcterms:source:
    - reference_source: <https://pubmed.ncbi.nlm.nih.gov/>
  ```

 - MESH concept layer, headings, Links are explained in PubChem documentation, where and how.

The use cases for this work are all resources collected 
Tables can be added in the following way, though alternatives are possible:

```markdown
Table: Note that table caption is automatically numbered and should be
given before the table itself.

| Header 1 | Header 2 |
| -------- | -------- |
| item 1 | item 2 |
| item 3 | item 4 |
```

This gives:

Table: Note that table caption is automatically numbered and should be
given before the table itself.

| Header 1 | Header 2 |
| -------- | -------- |
| item 1   | item 2   |
| item 3   | item 4   |


# Other main section on your manuscript level 1

Lists can be added with:

1. Item 1
2. Item 2

# Citation Typing Ontology annotation

You can use [CiTO](http://purl.org/spar/cito/2018-02-12) annotations, as explained in [this BioHackathon Europe 2021 write up](https://raw.githubusercontent.com/biohackrxiv/bhxiv-metadata/main/doc/elixir_biohackathon2021/paper.md) and [this CiTO Pilot](https://www.biomedcentral.com/collections/cito).
Using this template, you can cite an article and indicate _why_ you cite that article, for instance DisGeNET-RDF [@citesAsAuthority:Queralt2016].

The syntax in Markdown is as follows: a single intention annotation looks like
`[@usesMethodIn:Krewinkel2017]`; two or more intentions are separated
with colons, like `[@extends:discusses:Nielsen2017Scholia]`. When you cite two
different articles, you use this syntax: `[@citesAsDataSource:Ammar2022ETL; @citesAsDataSource:Arend2022BioHackEU22]`.

Possible CiTO typing annotation include:

* citesAsDataSource: when you point the reader to a source of data which may explain a claim
* usesDataFrom: when you reuse somehow (and elaborate on) the data in the cited entity
* usesMethodIn
* citesAsAuthority
* citesAsEvidence
* citesAsPotentialSolution
* citesAsRecommendedReading
* citesAsRelated
* citesAsSourceDocument
* citesForInformation
* confirms
* documents
* providesDataFor
* obtainsSupportFrom
* discusses
* extends
* agreesWith
* disagreesWith
* updates
* citation: generic citation

# Results

# Discussion

...

## Acknowledgements

...

## References
