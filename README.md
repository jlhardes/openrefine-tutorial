# openrefine-tutorial
Offers sample OpenRefine projects and lessons for trying clean-up tasks, reconciliation to Linked Data, and exporting as RDF.

Dataset from Indiana University Libraries, Digital Collection Services <a href="https://github.com/iulibdcs/cushman_photos">Github repository for the Charles W. Cushman Photograph Collection</a>. 

OpenRefine is a powerful tool but very particular. In this tutorial you will:
* Use a large dataset project to try cleaning tasks
* Use a small dataset project to try reconciling text to Linked Data URIs
  * Map Linked Data for RDF output

## Cushman-Dataset-Full
Practice cleaning data. Import this project into OpenRefine and you will have a dataset of 14,425 records.
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
    * Enter string above
    * Double-check year is accurate in formatted date
* Review at steps taken in Undo/Redo
