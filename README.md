# Graph-based iiRDS sample

This repository contains sample [iiRDS](https://www.iirds.org/) packages and a walkthrough for querying their metadata as an RDF knowledge graph.

The example follows the scenario from **“Hey, DITA! Talk to me!”**:

- a fictional fan is documented in a DITA-derived iiRDS package
- a supplier provides a second iiRDS package for the fan’s induction motor
- Apache Jena Fuseki loads the package metadata and the iiRDS ontology
- SPARQL queries find relevant topics without reading the DITA or HTML content
- one additional relationship connects the two manufacturers’ component identifiers
- a property-path query then finds related supplier information

The sample is intended for technical writers, information architects, knowledge-modeling practitioners, and developers experimenting with graph-based knowledge hubs that can be used for retrieval-augmented generation (Graph RAG).

## Repository contents

| File | Description |
| --- | --- |
| [`pi_fan.iirds`](./pi_fan.iirds) | iiRDS package for the fictional PI fan, including metadata and the fan documentation content. |
| [`iiRDS-OT-1769434483100.iirds`](./iiRDS-OT-1769434483100.iirds) | Supplier iiRDS package containing metadata for documentation about the fan’s induction motor. |

The packages are binary ZIP containers with the `.iirds` extension. The metadata is stored separately from the delivered content, in an RDF file named `metadata.rdf`.

> **Note**
> The packages are demonstration data. They are not a complete product documentation set and should not be used as operating or maintenance instructions for a real fan.

## What this example demonstrates

The example progresses through four increasingly useful graph operations:

- **Metadata filtering** – Find topics that relate to a component such as the rotor.
- **Ontology-aware classification** – Find topics relevant to the product lifecycle phase `iirds:Use`, even when the topic is tagged with a more specific phase such as `iirds:Maintenance`.
- **Cross-package linking** – Add the fact that the supplier’s `nline:Motor` is a component of the fan manufacturer’s `pi:Drive`.
- **Graph traversal** – Follow the component relationship to retrieve supplier information related to the same lifecycle phase.

All queries operate on the metadata graph. They do not need to inspect the DITA topics, generated HTML, or supplier PDF to determine which information units are relevant.

## What does this example not demonstrate
The example illustrates how metadata is queried on database level. It does not illustrate how graphs are used in content delivery portals. As most content delivery portals are not open-source, we cannot illustrate the benefits for end users in actual content delivery portals. However, the information queried on database level can be used to provide related links in delivery portals or as context for chat bots. For more information about actual projects, see [iiRDS best practice projects](https://www.iirds.org/tools/best-practices).  

## Prerequisites

You need:

- An archive tool to unizip the iiRDS packages
- Download of [Apache Jena Fuseki](https://jena.apache.org/download/)
- Download of the sample iiRDS packages in this repo
- Download of the [iiRDS schema](https://github.com/iirds-consortium/models/blob/main/iirds-core.rdf)


## Loading the iiRDS package
1. Start Apache Fuseki.

    For example, on Windows double-click the fuseki-server.bat.
    
1. In your internet browser, open the local server address.

   For example, http://localhost:3030
   
1. Rename the pi_fan.iirds package to pi_fan.zip.
1. Unzip the pi_fan.zip file.
1. In the Apache Jena Fuseki UI tab of your browser, click **add data**.    
1. Click **select files**.
1. In the file picker, select pi_fan/META-INF/metadata.rdf.
1. Click **upload now**.

The metadata of the pi_fan.iirds package is loaded into the FuseKi triple store.


## Find all topics about the rotor
Requirements: 
- You have succesfull completed "Loading the iiRDS package"

1. In the Apache Jena Fuseki UI tab of your browser, click **query**.
1. In the input box, enter the following SPARQL query and click ***Run query*** icon on the right.

```PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX iirds: <http://iirds.tekom.de/iirds#>
PREFIX pi: <https://www.i4icm.de/pifan#>

	SELECT ?topic ?title
	WHERE
		{ ?topic iirds:relates-to-component <https://www.i4icm.de/pifan#Rotor> . 
		  ?topic iirds:title ?title .	}
```
The result table lists all topics that have a relation to the rotor component. The authro assigned the metadata rotor to the topic.
 

## Find all topics about using the PI Fan
Requirements: 
- You have succesfull completed "Loading the iiRDS package"

1. In the Apache Jena Fuseki UI tab of your browser, click **query**.
1. In the input box, enter the following SPARQL query and click ***Run query*** icon on the right. 

 ```PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX iirds: <http://iirds.tekom.de/iirds#>
PREFIX pi: <https://www.i4icm.de/pifan#>

	SELECT ?topic ?title ?lphase
	WHERE
		{ ?topic iirds:relates-to-product-lifecycle-phase ?lphase .
  		  ?lphase rdf:type iirds:Use .
		  ?topic iirds:title ?title .	}
```

The result table lists all topics that are relevant when using the PI fan. The metadata Use was never assigned to a topic and is derived from the more detailed product lifecicle phases that were assigned to the topic. The iiRDS schema does allow to find all topics that are about use due to the class hierarchy. The iiRDS package was build using the [DITA OT iiRDS plugin](https://www.dita-ot.org/plugins/org.iirds.dita.package) which added some schema information about direct parent classes. 

## Adding the iiRDS schema
Requirements: 
- You have succesfull completed "Loading the iiRDS package"

1. In the Apache Jena Fuseki UI tab of your browser, click **add data**.    
1. Click **select files**.
1. In the file picker, select iirds-core.rdf.
1. Click **upload now**.

The full iiRDS schema is loaded into the triple store. That allows more sophisticated queries. 

## Adding supplier documentation
Requirements: 
- You have succesfull completed "Loading the iiRDS package"

1. Rename the iiRDS-OT-1769434483100.iirds package to iiRDS-OT-1769434483100.zip.
1. Unzip the iiRDS-OT-1769434483100.zip file.
1. In the Apache Jena Fuseki UI tab of your browser, click **add data**.    
1. Click **select files**.
1. In the file picker, select iiRDS-OT-1769434483100/META-INF/metadata.rdf.
1. Click **upload now**.
		  
The metadata of the iiRDS-OT-1769434483100.iirds package is loaded into the FuseKi triple store.

## Integrating supplier documentation
To integrate the supplier documentation we need to integrate the suppliers component into the component tree of the PI fan.

Requirements: 
- You have succesfull completed "Loading the iiRDS package"

1. In the Apache Jena Fuseki UI tab of your browser, click **edit**.
1. Click ***list current graphs***.
1. To select the graph in the list, click ***default***.
1. In the text editor field of the graph, add a the following triple and click ***save***:

``` j.0:Drive iirds:has-component <urn:uuid:5428c90e-bfa6-4b3c-9445-515987034bfb> . ```

## Finding related information for a topic
We want to find information that is relevant for the topic "Checking the power supply". Relevant documentation is everything that is about the same livecycle phase of the product and is related to the components in the topic.

Requirements: 
- You have succesfull completed "Loading the iiRDS package"
- You have succesfull completed "Adding the iiRDS schema"
- You have succesfull completed "Adding supplier documentation"
- You have succesfull completed "Integrating supplier documentation"

1. In the Apache Jena Fuseki UI tab of your browser, click **query**.
1. In the input box, enter the following SPARQL query and click ***Run query*** icon on the right. 

 ```PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX iirds: <http://iirds.tekom.de/iirds#>
PREFIX pi: <https://www.i4icm.de/pifan#>

	SELECT DISTINCT ?infou ?comp ?title ?lph
	WHERE
		{ ?topic iirds:title "Checking the power supply" .	
		  ?topic iirds:relates-to-component	?comp .
  		  ?topic iirds:relates-to-product-lifecycle-phase	?lph .
  		  ?infou rdf:type iirds:InformationUnit .
  		  ?infou iirds:relates-to-product-lifecycle-phase	?lph .
  	      ?infou (iirds:relates-to-component|^iirds:has-component)+ ?comp .
    	  ?infou iirds:title ?title .
		}
```
The result table lists all information units that are related by similar product lifecycle phases and components. The components relation anticipates part-of relations. The results find the supplier document as it is about a part of the drive which is relevant for checking the power supply as there could be a problem with the wiring.


## Related resources

- [iiRDS Consortium](https://www.iirds.org/)
- [iiRDS version 1.2 materials](https://www.iirds.org/materials/version-12)
- [iiRDS specification](https://www.iirds.org/fileadmin/iiRDS_specification/20251103-1.3-release/index.html)
- [Apache Jena Fuseki quick start](https://jena.apache.org/documentation/fuseki2/fuseki-quick-start.html)
- [W3C SPARQL 1.1 Query Language](https://www.w3.org/TR/sparql11-query/)

## License and Copyright

All files are included solely for testing, demonstration, and research purposes.
Their inclusion does not imply that the maintainer owns, licenses, or endorses
the underlying content.

Each file should be used subject to the copyright, licensing, and other terms
specified by its original rights holder. Where available, attribution and source
information are provided alongside the file.

If you are a rights holder and believe that material in this repository
infringes your rights, please open an issue or contact the repository maintainer
at [contact address] with:

- the URL or path of the allegedly infringing material;
- identification of the copyrighted work;
- your contact details; and
- a statement explaining the basis of your request.

We will review legitimate notices promptly and, where appropriate, remove or
disable access to the material.

