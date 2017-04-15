# openrefine-tutorial
This tutorial taught as part of <a href="https://libraries.indiana.edu/seminars-and-workshops">Indiana University Scholars Commons Workshop Series</a>. Offers sample OpenRefine projects and lessons for trying clean-up tasks, reconciliation to Linked Data, and exporting as RDF.

We're skipping the process to create a project in OpenRefine due to location of OpenRefine app on computers used in workshop series.

The dataset used in this tutorial is from the Indiana University Libraries, Digital Collection Services' <a href="https://github.com/iulibdcs/cushman_photos">Github repository for the Charles W. Cushman Photograph Collection</a>. 

OpenRefine is a powerful tool but very particular. In this tutorial you will:
* Use a large dataset project to try cleaning tasks
* Use a small dataset project to try reconciling text to Linked Data URIs
  * Map Linked Data for RDF output

## Cushman-Dataset-Full.openrefine.tar.gz
Practice cleaning data. Import this project file into OpenRefine and you will have a dataset of 14,425 records.
* Clean **Corporate Names 1**
    * Facet > Text facet
    * Cluster on facet results
    * Review any results and select corrections, then merge selected and close or re-cluster
* Clean **Other 1**
* Clean **Other 2**
* See how many rows are missing **City**
    * Facet > Text facet
    * Scroll down to end of list for (blank) facet
* See how many rows are missing **State/Province**
* Format **Start Date**: 
    * <a href="https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language">GREL</a> expression to use: `value.toDate().toString('yyyy-MM-dd')`
    * Edit cells > Transform
    * Enter excpression above
    * Double-check year is accurate in formatted date
* Review at steps taken in Undo/Redo

## Cushman-Dataset-Small.openrefine.tar.gz
Practice using reconciliation services to turn text strings into Linked Data URIs based on controlled vocabularies. Map fields in dataset to RDF properties and export as Linked Data RDF. Import this project file into OpenRefine and you will have a dataset of 20 records.

### Wikidata Reconciliation Service
* **Corporate Names 1**: choose Reconcile > Start reconciling...
* Click Add Standard Service button and enter: `https://tools.wmflabs.org/openrefine-wikidata/en/api`
* This adds the reconciliation service. Choose Reconcile > Start reconciling... again and select 'Wikidata Reconciliation for OpenRefine (en)' from left-side menu
  * Reconcile against 'public company' in list of type options or 'no particular type' at bottom
* Review results for how much was reconciled (and how much was not)
* Try reconciling on **Topcial Subject Headings 1** and see how some matches happened but many still do not retrieve a match

### Thesaurus of Graphic Materials (TGM) Reconciliation Service

#### Add RDF Extension
You will need the RDF Extension for OpenRefine, available at https://github.com/fadmaa/grefine-rdf-extension/downloads.  Once you download and extract the file, rename the directory `rdf-extension` and move it to your `extensions` folder in OpenRefine, then restart OpenRefine. You should see an 'RDF' drop-down menu in the upper right of OpenRefine when viewing a project if the extension installed successfully. See <a href="https://github.com/OpenRefine/OpenRefine/wiki/Installing-Extensions">OpenRefine Wiki - Installing Extensions</a> for further help.

In order to use the TGM reconcilation service, you can download and extract the controlled vocabulary to load as a local file for use in OpenRefine from the Library of Congress: http://id.loc.gov/static/data/vocabularygraphicMaterials.ttl.zip

#### Add TGM Reconciliation Service using `tgm.ttl` file:
* Under RDF > Add reconciliation service > Based on RDF file...
    * Name: TGM
    * Upload file: `tmg.ttl` (wherever it is located)
    * File format: Turtle
    * Label properties
        * Other: http://www.loc.gov/mads/rdf/v1#authoritativeLabel
        * You can open `tgm.ttl` and look at example record in vocab to see that this property is being used for the term label

#### Reconcile against TGM:
* **Genre 1**: Reconcile > Start reconciling...
* Select 'TGM' from left-side menu
* Use default type (Topic)
* Should result in 100% exact matches for all records with genre

### RDF Mapping
* Under RDF > Edit RDF Skeleton
* Add prefixes:
  * loc: http://id.loc.gov/vocabulary/resourceTypes/
  * dc: http://purl.org/dc/elements/1.1/
  * edm: http://www.europeana.eu/schemas/edm/

Goal is to end up with RDF mapping like the following:
```javascript
<http://purl.dlib.indiana.edu/iudl/archives/cushman/P01411> a foaf:Image ;
	dc:date "Sat Sep 03 00:00:00 EST 1938" ;
	dc:title "A-5 = G.G. bridge from south end." ;
	edm:hasType <http://id.loc.gov/vocabulary/graphicMaterials/tgm000464> ;
	dc:subject "Mountains" ;
	dc:creator "Charles W. Cushman" .
}
```

* Add Creator column to dataset to include 'Charles W. Cushman' value
    * On column with text values (such as **IU Archives Number**) select Edit column > Add column based on this column
    * Supply a new column name and in the Expression dialog box enter (with single quotes): `'Charles W. Cushman'`
* Make Genre URI its own column
    * **Genre 1**: Edit column > Add column based on this column
    * Give new column name: Genre URI
    * Add following in Expression dialog box: `cell.recon.match.id`
* Under RDF > Edit RDF Skeleton
    * Change value of subject to be PURL with content as a URI
        * Add RDF type of foaf:Image
    * Add properties to match each property listed in goal
    * Add values for each property
    	* Use **Start Date** as text for dc:date
    	* Use **Description from Notebook** as text for dc:title
    	* Use **Genre URI** as URI for edm:hasType
    	* Use **Topical Subject Headings 1** as text for dc:subject
    	* Use **Creator** as text for dc:creator
    * Use RDF Preview tab as you go to verify that mapping is resulting in desired Linked Data RDF output
    * Click OK in RDF Schema Alignment to save mapping
* Export > RDF as Turtle

## Other Resources
* <a href="http://is.gd/refine">OpenRefine Tutorial from Enipedia</a>
    * Provides dataset to create project along with various tasks for cleaning
* <a href="http://openrefine.org/documentation.html">OpenRefine Documentation for User</a>
    * Includes FAQ, OpenRefine wiki, many other tutorials, and a book recommendation
* <a href="https://docs.google.com/document/d/1rvVOc69NJtNacTqgmkOliOBGDdYI4J1-qqIcRuFrou0/edit?usp=sharing">OpenRefine Instructions: A Guide to Creating an OpenRefine Project Using the Charles W. Cushman Photograph Collection Metadata Dataset</a>
    * Instructions for creating an OpenRefine project and reconciling the full Cushman dataset
