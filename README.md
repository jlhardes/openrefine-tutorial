# openrefine-tutorial
This tutorial taught as part of <a href="https://libraries.indiana.edu/seminars-and-workshops">Indiana University Scholars Commons Workshop Series</a>. Offers sample OpenRefine projects and lessons for trying clean-up tasks, reconciliation to Linked Data, and exporting as RDF.

We're using <a href="http://openrefine.org/download.html">OpenRefine 2.6</a>.

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
* See how many rows are missing **State/Province**
    * Facet > Text facet
    * Scroll down to end of list for (blank) facet
* See how many rows are missing **City**
* See how many rows contain unidentifiable characters � in **City**
    * Ideally character encoding issues are something to fix with original data before OpenRefine import, but you can do the following if that is not an option
    * Text filter
    * Paste � to search for rows containing that character
    * Facet > Text facet
    * In Text Facet result, click city containing unidentifiable characters
    * In Text Facet result, mouseover city name and click "edit" option
    * Replace unidentifiable character with correct character (from text editor using Unicode UTF-8)
    * Examples:
        * Ciudad Ju�rez > Ciudad Juárez
        * Dh�los > Dhílos
* Format **Start Date**: 
    * Edit cells > Transform
    * Enter <a href="https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language">GREL</a> expression: `value.toDate().toString('yyyy-MM-dd')`
    * Double-check year is accurate in formatted date (HINT: you can change anything within "yyyy-MM-dd" to actual numbers and Cushman only took photographs in the 1900s)
* Review steps taken in Undo/Redo tab on left side of project view

## Cushman-Dataset-Small.openrefine.tar.gz
Practice using reconciliation services to turn text strings into Linked Data URIs based on controlled vocabularies. Map fields in dataset to RDF properties and export as Linked Data RDF. Import this project file into OpenRefine and you will have a dataset of 20 records.

### Wikidata Reconciliation Service
* **Corporate Names 1**: choose Reconcile > Start reconciling...
* Click Add Standard Service button and enter: `https://tools.wmflabs.org/openrefine-wikidata/en/api`
* This adds the reconciliation service. Choose Reconcile > Start reconciling... again and select 'Wikidata Reconciliation for OpenRefine (en)' from left-side menu
  * Reconcile against 'public company' in list of type options or 'no particular type' at bottom
* Review results for how much was reconciled (and how much was not)
* To see URI values that were matched:
  * Edit column > Add column based on this column
  * In Expression, enter the URI value up to the identifier: `"https://www.wikidata.com/wiki/"` and then `+ cell.recon.match.id`
  * Name the new column
  * Any terms that were matched to a URI from the Wikidata Reconciliation Service will show the corresponding URI for that term
* Try reconciling on **Topical Subject Headings 1** and see how some matches happened but many still do not retrieve a match

### Geocoding (retrieving coordinates)
These instructions follow closely with those provided in the <a href="http://enipedia.tudelft.nl/wiki/Google_Refine_Tutorial#Geocoding_names_and_addresses">OpenRefine Tutorial from Enipedia - Geocoding names and addresses</a> linked at the end of this tutorial.
* Location info is split into separate columns but we need something combined together to query against Google Maps for coordinates
* Duplicate the Country column for use as a Full Address
  * Edit column > Add column based on this column
  * Name the new column
* In that new column
  * Edit cells > Transform
  * Enter `cells["columnName"].value + ", " + cells["columnName"].value` to put together the values from City, State/Province, and Country columns
  * Edit column > Add column by fetching urls
  * In Expression, enter `"http://maps.google.com/maps/api/geocode/json?sensor=false&address=" + escape(value, "url")`, set Throttle delay to 500 milliseconds, and name the new column
* This request will take 3-4 minutes for these 20 records and Google limits the number of requests per day but this should return JSON output in each row with an address
* To pull out just the coordinate information from this data
  * Edit column > Add column based on this column
  * In Expression, enter `with(value.parseJson().results[0].geometry.location, pair, pair.lat +", " + pair.lng)`
  * Name the new column
  * Split that new column to divide up latitude and longitude
    * Edit column > Split into several columns (use "," as separator)
    * Rename these columns using Edit column > Rename this column

### Thesaurus of Graphic Materials (TGM) Reconciliation Service

#### Add RDF Extension
You will need the RDF Extension 0.9 for OpenRefine, available at https://github.com/fadmaa/grefine-rdf-extension/releases.  Once you download and extract the file, move it to your `extensions` folder in OpenRefine, then restart OpenRefine. You should see an 'RDF' drop-down menu in the upper right of OpenRefine when viewing a project if the extension installed successfully. See <a href="https://github.com/OpenRefine/OpenRefine/wiki/Installing-Extensions">OpenRefine Wiki - Installing Extensions</a> for further help.

In order to use the TGM reconcilation service, you can download and extract the controlled vocabulary from the Library of Congress to load as a local file for use in OpenRefine: http://id.loc.gov/static/data/vocabularygraphicMaterials.ttl.zip

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

Goal is to end up with RDF output like the following:
```
<http://purl.dlib.indiana.edu/iudl/archives/cushman/P01411> a foaf:Image ;
	dc:date "1938-09-03"^^<http://www.w3.org/2001/XMLSchema#date> ;
	dc:title "A-5 = G.G. bridge from south end." ;
	edm:hasType <http://id.loc.gov/vocabulary/graphicMaterials/tgm000464> ;
	dc:subject "Mountains" ;
	dc:creator "Charles W. Cushman" .
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
			* Make sure this cell's content is used as date ('YYYY-MM-DD')
    	* Use **Description from Notebook** as text for dc:title
    	* Use **Genre URI** as URI for edm:hasType
    	* Use **Topical Subject Headings 1** as text for dc:subject
    	* Use **Creator** as text for dc:creator
    * Use RDF Preview tab as you go to verify that mapping is resulting in desired Linked Data RDF output
    * Click OK in RDF Schema Alignment to save mapping
* Export > RDF as Turtle

## Other Resources
* <a href="https://github.com/OpenRefine/OpenRefine/wiki/Recipes">Recipes from OpenRefine Wiki</a>
    * Useful recipes for achieving certain tasks in OpenRefine
* <a href="http://is.gd/refine">OpenRefine Tutorial from Enipedia</a>
    * Provides dataset to create project along with various tasks for cleaning
* <a href="http://openrefine.org/documentation.html">OpenRefine Documentation for User</a>
    * Includes FAQ, OpenRefine wiki, many other tutorials, and a book recommendation
* <a href="https://docs.google.com/document/d/1rvVOc69NJtNacTqgmkOliOBGDdYI4J1-qqIcRuFrou0/edit?usp=sharing">OpenRefine Instructions: A Guide to Creating an OpenRefine Project Using the Charles W. Cushman Photograph Collection Metadata Dataset</a>
    * Instructions for creating an OpenRefine project and reconciling the full Cushman dataset
