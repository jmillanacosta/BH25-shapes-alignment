---
title: 'DBCLS BioHackathon 2025 report: Identifying Requirements for the Systematic Matching of RDF Life Sciences Databases through Automatically Extracted Schemas'
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
    orcid: 0000-0001-5608-781X
  - name: Last Author
    orcid: 0000-0000-0000-0000
    affiliation: 2
affiliations:
  - name: Department of Translational Genomics, Maastricht University, Maastricht, The Netherlands
    index: 1
    ror: 02jz4aj89
date: 27 November 2025
cito-bibliography: paper.bib
event: BH25JP
biohackathon_name: "DBCLS BioHackathon 2025"
biohackathon_url:   "https://2025.biohackathon.org/"
biohackathon_location: "Mie, Japan, 2025"
group: shapes-alignment
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/jmillanacosta/BH25-shapes-alignment/
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Millán Acosta \emph{et al.}
---
# Abstract

Life sciences databases can be exposed as Resource Description Framework (RDF) linked data. However, even if the underlying vocabularies or ontologies are shared, different RDF resources present different structures, intended usability, modeling choices, and levels of data completeness. This limits in practice their interoperability and adds significant overhead to the construction of queries or querying tools that can traverse multiple RDF resources.

As part of the DBCLS BioHackathon 2025, we report our work towards defining a methodology to systematically retrieve RDF schemas from different RDF resources that describe their actual structure, and compare them to expose gaps and opportunities for interoperability. We identified several formal representations of RDF schemas and the tools already developed to extract them in automated workflows. We then identified key requirements for schema matching workflows, including the need to measure structural consistency within a dataset and alignment across datasets, and prototyped two methodologies based on Shape Expressions using the sheXer and rudof libraries.

Several factors pose a challenge for the use of shape languages in continuous integration of asynchronously developed RDF graph resources: the complexity of highly expressive shape languages like SHACL and ShEx, their slow rate of adoption as schema-documenting artifacts by linked dataset providers, and the technical limitations around their automated extraction. A lightweight methodology to programmatically extract high-level VoID descriptions of RDF resources is proposed as a complementary approach to guide schema extraction and matching workflows.

Although the focus of this work is on automated extraction of schemas, we hope that by demonstrating the practical benefits of service description graphs for cross-resource integration and interoperability linked dataset providers will be motivated to incorporate "well-known" VoID graphs, SHACL, ShEx or rdf-config as part of their standard release practices.

## Author information

# Introduction

Linked datasets differ in intended purpose of the dataset, modeling choices, ontology usage, and data completeness. These variations limit the ability to infer dataset structure and interoperability from the used ontologies or documentation alone, and producing a practical overview of available data across linked datasets still requires exhaustively inspecting the RDF, SPARQL endpoints, reading documentation, and manually constructing queries to probe for content, limiting the findability and accessibility of these resources.

In previous hackathons, the development of linked data querying tools such as pyBioDatafuse, a python package for the construction of knowledge graphs from linked and non-linked data, showed that manual authoring of queries for annotators remains the main bottleneck when developing tools that consume linked data.

Dataset providers often publish SPARQL examples, but these remain limited and fit for specific purposes and do not expose the actual structure of the underlying RDF graphs. Moreover, data completeness varies across resources and even within a single resource, causing irregular populations of properties, inconsistent use of identifiers, and unpredictable coverage of key attributes that make it hard to discover linked data.

Because of these factors, just knowing the used ontologies and text-based dataset documentation does not give a full picture of the actual graph. They do not reveal which classes are instantiated, how frequently properties occur for entities of a given OWL class, or where structural gaps appear. As a result, two datasets using the same ontology may have very different practical structures, and datasets using different ontologies may still have overlapping patterns that are not visible from documentation alone. This limits any attempt to assess interoperability without inspecting the concrete RDF graphs in detail.

Earlier hackathon work showed that machine-readable Shape Language descriptions of RDF resources have potential to allow automatic discovery and matching of resources for data integration [@citesAsRelated:bh23Integration;@citesAsRelated:bh23Enhancement;@citesAsRelated:bh24Toward;@citesAsRelated:bh24Discovered;@citesAsRelated:fernandez-alvarez_extracting_2024]. However, their adoption in the life science RDF landscape is limited, leaving most graphs without shapes-based model descriptions.

The main challenges in RDF schema matching and interoperability can be categorized into two key areas:

1. Measuring consistency: an assessment of how stable and repeatable a structural pattern is within one RDF graph. Consistency (coverage) scores could indicate how often a given combination of classes, properties, and value types actually occurs across instances. These scores could reflect the practical, observable structure of the dataset, independent of its ontology declarations.  
2. Measuring alignment: a comparison of extracted structural patterns across datasets to identify overlaps or equivalents. Curated mappings and declared cross-references feed into this stage by linking classes or properties that are known to correspond. Alignment enables the detection of candidate correspondences and potential walkable paths across RDF graphs once structural patterns are available.  
3. Providing tooling and explicitly exposing schemas: few providers publish class partitions in their service descriptions, or shape language definitions, and extraction methods remain resource-intensive.


**Outcomes of this hackathon and tools of interest**

- As a result of discussions during this hackathon, the truthfulness/consistency values extracted by **sheXer**[@citation:shexer] were converted into explicit axioms in the generated ShEx. In previous releases they were exposed as comments, but with release [2.6.5.1](https://github.com/weso/shexer/tree/2.6.5.1) the user can set an annotation predicate to include the measure in the generated graph. The following snippet demonstrates how to display consistency values as annotations:

```python
from shexer.shaper import Shaper
from shexer.consts import TURTLE

def run():

    namespaces = {"http://example.org/": "ex",
             "http://www.w3.org/XML/1998/namespace/": "xml",
             "http://www.w3.org/1999/02/22-rdf-syntax-ns#": "rdf",
             "http://www.w3.org/2000/01/rdf-schema#": "rdfs",
             "http://www.w3.org/2001/XMLSchema#": "xsd",
             "http://xmlns.com/foaf/0.1/": "foaf",
             "http://weso.es/shexer/ontology/" : "shexer"
             }

    raw_graph = """
    @prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    @prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
    @prefix ex: <http://example.org/> .
    @prefix foaf: <http://xmlns.com/foaf/0.1/> .
    @prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
    
    ex:Jimmy a foaf:Person ;  # Complete
        foaf:age "23"^^xsd:integer ;
        foaf:name "Jimmy" ;
        foaf:familyName "Jones" .
    
    ex:Sarah a foaf:Person ;  # Complete implicit type for age
        foaf:age 22 ;
        foaf:name "Sarah" ;
        foaf:familyName "Salem" .
    
    ex:Bella a foaf:Person ;  # Missing familyName
        foaf:age "56"^^xsd:integer ;
        foaf:name "Isabella" .
    
    ex:David a foaf:Person ;  # Missing age and use knows
        foaf:name "David" ;
        foaf:familyName "Doulofeau" ;
        foaf:knows ex:Sarah .
    
    ex:HumanLike foaf:name "Person" ;  # foaf properties, but not explicit type.
        foaf:familyName "Maybe" ;
        foaf:age 99 ;
        foaf:knows ex:David .
    
    ex:x1 rdf:type foaf:Document ;
        foaf:depiction "A thing that is nice" ;
        foaf:title "A nice thing" .
    
    
    ex:x2 rdf:type foaf:Document ;
        foaf:title "Another thing" .
    
    """

    shaper = Shaper(
        raw_graph=raw_graph,
        namespaces_dict=namespaces,
        all_classes_mode=True,
        input_format=TURTLE,
        disable_comments=False,
        comments_to_annotations=True)

    str_result = shaper.shex_graph(string_output=True)
    print(str_result)

if __name__ == "__main__":
    run()
```
Results in:

```ttl
PREFIX ex: <http://example.org/>
PREFIX xml: <http://www.w3.org/XML/1998/namespace/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX shexer: <http://weso.es/shexer/ontology/>
PREFIX : <http://weso.es/shapes/>
:Document
{
   foaf:title  xsd:string    //  shexer:ratio_property_usage  1.00  ;
   rdf:type  [foaf:Document]    //  shexer:ratio_property_usage  1.00  ;
   foaf:depiction  xsd:string  ?  //  shexer:ratio_property_usage  0.50
}
:Person
{
   foaf:name  xsd:string    //  shexer:ratio_property_usage  1.00  ;
   rdf:type  [foaf:Person]    //  shexer:ratio_property_usage  1.00  ;
   foaf:familyName  xsd:string  ?  //  shexer:ratio_property_usage  0.75  ;
   foaf:age  xsd:integer  ?  //  shexer:ratio_property_usage  0.75  ;
   foaf:knows  @:Person  ?  //  shexer:ratio_property_usage  0.25
```

- This project also led to the **rudof** release 0.1.120, adding a schema comparison module to compare shapes by computing property-set overlaps across mapped subject types in a ShEx or SHACL document. A standardized method for feeding the mapping files that could connect the shape subjects for comparison could not ne implemented in this hackathon due to time constraints, and further work is needed to extend the overlap analysis to the set of object values for the shapes. Future developments of this module will allow for the comparison of shapes within and across linked datasets based on their declared or extracted ShEx, potentially allowing for fine-grained interoperability validation workflows when authoring RDF datasets. The example reproduced below from the [Short introduction ro Rudof](https://colab.research.google.com/drive/1XuxohKDNn4UsuRKokyjH2bAlZEyyYhnl#scrollTo=QtCQjCyUhkoI) jupyter notebook displays the `shapes_comparison` functionality in `pyrudof`, the python bindings-generated version of the `rudof` rust library.

  ```python
  schema1 = """
  PREFIX : <http://example.org/>
  PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
  :Person {
     :name xsd:string ;
     :age xsd:integer ;
     :weight xsd:float ;
     :worksFor .
  }
  """;


  schema1 = """
  PREFIX : <http://example.org/>
  PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
  :Person {
     :name xsd:string ;
     :age xsd:integer ;
     :weight xsd:float ;
     :worksFor .
  }
  """;


  result = rudof.compare_schemas_str(
     schema1, schema2,
     "shex", "shex",
     "shexc", "shexc",
     None, None,
     "http://example.org/Person", "http://example.org/Person"
     )
  ```
  Results in
  ```json
  {
    "equal_properties": {
      "http://example.org/name": {
        "description1": {
          "iri_ref": "http://example.org/name",
          "value_constraint": {
            "Datatype": "http://www.w3.org/2001/XMLSchema#string"
          }
        },
        "description2": {
          "iri_ref": "http://example.org/name",
          "value_constraint": {
            "Datatype": "http://www.w3.org/2001/XMLSchema#string"
          }
        }
      },
      "http://example.org/worksFor": {
        "description1": {
          "iri_ref": "http://example.org/worksFor"
        },
        "description2": {
          "iri_ref": "http://example.org/worksFor"
        }
      }
    },
    "properties1": {
      "http://example.org/weight": {
        "description": {
          "iri_ref": "http://example.org/weight",
          "value_constraint": {
            "Datatype": "http://www.w3.org/2001/XMLSchema#float"
          }
        }
      },
      "http://example.org/age": {
        "description": {
          "iri_ref": "http://example.org/age",
          "value_constraint": {
            "Datatype": "http://www.w3.org/2001/XMLSchema#integer"
          }
        }
      }
    },
    "properties2": {
      "http://example.org/birthDate": {
        "description": {
          "iri_ref": "http://example.org/birthDate",
          "value_constraint": {
            "Datatype": "http://www.w3.org/2001/XMLSchema#date"
          }
        }
      }
    }
  ```

- **rdf-config** was evaluated as a human-readable overview of RDF resource schemas. It consists of a simple YAML structure where the schema of a linked dataset is presented as a tree linking subject types, properties, and possible object type values, together with some examples of possible instances. An exhaustive collection of manually curated rdf-config schemas is available for the resources in the RDF Portal, and a workflow was identified that parses **voID** graphs detailing class, property and data partitions into rdf-config-like YAMLs. However, there are yet no packaged automated rdf-config extraction tools we could test out during the hackathon.

- **VoID generator**: A VoID generator tool was also evaluated as a method to extract high-level VoID service descriptions from RDF resources exposing class and property partitions, which can be used to guide schema extraction and matching workflows. Previous work has documented the practical limitations of this approach mostly due to SPARQL endpoint timeouts [@citesAsRelated:bh24Discovered], but detailed VoID graph exposures remain a promising complementary approach to guide automated schema discovery and extraction workflows. The usability of VoID graphs to guide SPARQL queries is shown in work like SPARQL-editor [@citesAsRecommendedReading:emonet_sparqleditor], where existing VoID descriptions are queried when instantiating a YASGUI query builder interface to provide the available classes in the target endpoint.

- **shex-consolidator**: We identified shex-consolidator [@citesAsRelated:fernandez-alvarez_extracting_2024] as a tool for merging multiple ShEx schemas into a single consolidated schema. This tool will be particularly valuable once an ecosystem of automatically generated shapes emerges or when tooling for interconverting between different shape formats becomes more established. The consolidation functionality allows merging shapes associated with the same label across multiple input schemas, with configurable acceptance thresholds for constraint inclusion based on their support across datasets.

**Discussion**

During this hackathon, we collected several formal representations of RDF schemas and evaluated existing tools for their automated extraction in workflows. We prototyped two methodologies based on Shape Expressions using the sheXer and rudof libraries, addressing key requirements for schema matching workflows: measuring structural consistency within datasets through coverage scores, and enabling alignment comparisons across datasets through property-set overlap analysis.

However, two major goals were not implemented and will be addressed in future hackathons:

1. Defining a workflow for the automated identification of candidate entities across datasets (mappings workflow)  
2. Defining a workflow for the automated construction of traversal paths between related nodes across multiple RDF schema graphs.

The work completed during this hackathon also highlighted persistent challenges that must be addressed for wider adoption of shape-based schema matching. The high expressivity of shape languages like SHACL and ShEx, while powerful, results in complex schema definitions that become increasingly difficult to parse and compare programmatically when constraints involve deeply nested structures, pointing at the sheXer methodology of consolidating the consistency of shapes as the most practical solution. However, the rate of adoption of shapes as schema-documenting artifacts by linked dataset providers remains slow. Furthermore, technical limitations around automated extraction, particularly SPARQL endpoint timeouts, continue to constrain the practical application of these methods.

As a complementary approach to dataset providers exposing VoID, shapes, or rdf-config, we propose as future hackathon work developing a lightweight methodology to programmatically extract (sampled) high-level VoID descriptions from candidate datasets. VoID descriptions can provide draft class and property partition information to guide initial schema extraction and matching workflows. While less expressive than full descriptions, these could be simpler to generate and process, and may serve as an intermediate step toward broader adoption of formal schema documentation in linked dataset releases.

Addressing the gaps identified in this hackathon project will help improve schema matching, reduce dependence on manual query inspection, and support more robust interoperability assessments. Continuous work is needed to standardize schema extraction procedures and encourage providers to expose schemas as sufficient metadata to make their RDF graphs discoverable and comparable. Ultimately, this will help linked dataset users and providers identify structural gaps and potential integration points between RDF resources, integrating extracted schemas into data pipelines and supporting federated querying and integration of independently and asynchronously developed RDF graphs.


## Acknowledgements

...

## References
