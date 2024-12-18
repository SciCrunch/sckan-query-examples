# Exploring SCKAN Using Example Queries

* [Query Examples in Jupyter Notebook](example-queries/sckan-sparql-query-examples.ipynb)

# Introduction

The NIH Common Fund's Stimulating Peripheral Activity to Relieve Conditions ([SPARC](https://commonfund.nih.gov/sparc)) initiative is a large-scale program designed to advance the development of precise and effective bioelectronic neuromodulation therapies for various diseases and conditions. A key component of the SPARC project, the S[PARC Connectivity Knowledgebase of the Autonomic Nervous System](https://sparc.science/resources/6eg3VpJbwQR4B84CjrvmyD) (SCKAN, [RRID:SCR_026088](https://scicrunch.org/resolver/RRID:SCR_026088)), is a dynamic knowledge base of ANS connectivity that contains information about the origins, terminations, and routing of ANS projections between the central nervous system and end organs. The distillation of SPARC's connectivity knowledge into SCKAN involves a rigorous curation process to capture connectivity information provided by experts, published literature, textbooks, and SPARC scientific data. SCKAN is being used within SPARC to generate interactive connectivity maps available via the [SPARC Portal](https://sparc.science/apps/maps?id=7d8ad680).

![1734510186322](image/README/1734510186322.jpg)

> **Figure 0:** A high-level depiction of SCKAN’s knowledge construction flow, starting with the extraction and distillation of connectivity knowledge from different sources, followed by the curation and transformation of this knowledge into formal representations, and finally, making the knowledgebase available via different graph database endpoints.

**Figure 0** provides a high-level depiction of SCKAN’s knowledge construction flow. To know more about SCKAN and the details of its knowledge construction flow depicted in the figure, refer to the following article.

* Imam FT, Gillespie TH, Ziogas I, Surles-Zeigler MC, Tappan S, Ozyurt BI, Boline J, de Bono B, Grethe JS, Martone ME (2024). [Developing a Multiscale Neural Connectivity Knowledgebase of the Autonomic Nervous System](https://www.biorxiv.org/content/10.1101/2024.10.25.620360v3). bioRxiv 2024.10.25.620360. **doi:** https://doi.org/10.1101/2024.10.25.620360

In this document, we provide the background knowledge needed to write SPARQL queries to explore SCKAN. We also include a set of competency quries (CQs) to demonstrate how SCKAN can be used to ask questions relevant to ANS connectivity.

# Background

In this section, we provide the background knowledge needed to understand how SCKAN models its connectivity along with the list of phenotypic relations (predicates) needed for writing SPARQL queries.

## How SCKAN Models its Connectivity?

SCKAN models connectivity at the neuronal level, where connections are represented based on the locations of neuronal cell bodies, axon segments, dendrites, and axon terminals. Each connection represents a population of PNS neurons with a given PNS phenotype (e.g. sympathetic pre-ganglionic) that projects to a given set of targets, where the exact cell type identity may not be known. A neuron population in SCKAN refers to a set of neurons with shared phenotypic properties without specifying the neron types. The connectivity of a neuron population is modeled based on its locational phenotypes using the following form:

* **Neuron population $N$ at $A$ projects to $B$ via $C$**, where
  * $A$ represents a set of anatomical regions containing the somata;
  * $B$ represents a set of anatomical regions containing the axon terminals; and
  * $C$ denotes the set of distinct anatomical region(s) between $A$ and $B$ that contain the axon segments, including nerves.

SCKAN also supports modelling of the partial orders of the axonal paths between the origin and the destination of the neuron populations. The following diagram in **Figure 1** depicts the connectivity model of a generic neuron population in SCKAN.

![1734468121787](image/README/1734468121787.jpg)

> **Figure 1:** SCKAN's connectivity model of a neuron population based on its axonal projection path.

In addition to specifying the anatomical locations of neuronal segments, each neuron population in SCKAN can be further specified with other relevant phenotypic properties, e.g.:

* **ANS Subdivision**: Sympathetic or Parasympathetic, Preganglinic or Post-ganglionic
* **Circuit Role**: Intrinsic, Motor, Sensory, or Projection
* **Functional Circuit Role**: Whether the connection is Excitatory or Inhibitory
* **Species and Sex**: The specificity of Species and Sex the connection was observed in

A given population subserves a single PNS phenotype, e.g., sympathetic pre or post ganglionics, but may have multiple origins, primarily to indicate that a connection arises from multiple spinal cord segments, as well as multiple terminal locations. SCKAN currently does not model whether a neuron population projecting to multiple regions does so via branching axons or whether distinct subpopulations project to distinct targets. We use the axon sensory terminal to indicate the peripheral process of pseudo-monopolar dorsal root ganglia connections.

The connectivity of a single neuron population includes distinct anatomical structures along its projection paths from its origin(s) to its destination(s). The diagram on the left in **Figure 2** shows a connectivity pathway of a parasympathetic preganglionic neuron population in SCKAN, originating at the superior salivatory nucleus and terminating at the pterygopalatine ganglion. In this example, the axon reach its target via the facial nerve, followed by the greater petrosal nerve and then the vidian nerve.

![1734468443373](image/README/1734468443373.jpg)

> **Figure 2:** Neural connectivity examples from SCKAN. The left diagram illustrates the connectivity pathway of a parasympathetic preganglionic population, while the diagram on the right depicts a sympathetic innervation circuit of the ovary, formed by synaptic connections between pre- and postganglionic populations. Each node in the diagrams is labeled with the corresponding anatomical region, drawn from standard vocabulary sources such as UBERON or InterLex (ILX), along with its CURIE (compact URI).

SCKAN facilitates the modeling of connectivity circuits by specifying synaptic forward connections between first-order and second-order neuron populations. Preganglionic neuron populations, originating in the CNS and terminating in ANS ganglia, represent the first-order connections in SCKAN. Postganglionic populations, originating in the ANS ganglia and terminating in a specific part of a target organ, represent the second-order connections. When a forward connection is specified between two populations, the synapse location within the circuit can be identified as the common structure(s) where the preganglionic connection terminates (the location of the axon presynaptic element) and the postganglionic connection originates.

The diagram on the right in **Figure 2** shows an example of a sympathetic innervation circuit of the ovary, formed by the synaptic connections between a sympathetic preganglionic population (`femrep:36`) and two postganglionic populations (`femrep:37` and `femrep:40-1`), observed in female rats. The preganglionic population (`femrep:36`) originates from two adjacent locations (T9-T10 segments of thoracic spinal cord) and terminates in three locations (T12-L1 paravertebral sympathetic chain) via the  white communicating rami of T9-10. The origins (the somata locations) of the postganglionic populations (`femrep:37` and `femrep:40-1`) are synaptically co-located with the preganglionic terminals, and their axons travel via the ovarian nerve plexus, with one terminating in the left ovary and the other in the right ovary. All the nodes representing the anatomical regions in **Figure 2** are drawn from standard vocabulary sources including UBERON ([RRID:SCR_010668](https://scicrunch.org/resolver/RRID:SCR_010668)), InterLex (ILX, [RRID:SCR_016178](https://scicrunch.org/resolver/RRID:SCR_016178)).

Refer to the [SCKAN article](https://www.biorxiv.org/content/10.1101/2024.10.25.620360v3) to know more about SCKAN along with its modeling principles, curation process, vocabualary management, and competency queries.

## The Phenotypic Relations in SCKAN

To get started with writing SPARQL queries for SCKAN we simply need to know about the phenotypic relations (predicates) supported by [Simple SCKAN](https://github.com/SciCrunch/sparc-curation/blob/master/docs/simple-sckan/readme.md) as listed in the tables below.

### Locational Phenotypes in SCKAN

| Phenotypic Relation                       | Description                                                                                                                                                                                                    |
| ----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **hasSomaLocation**                 | Expresses a relation between a neuron population and its origin i.e., location of the cell body                                                                                                                |
| **hasAxonLocation**                 | Expresses a relation between a neuron population and its axon location                                                                                                                                        |
| **hasDendriteLocation**             | Expresses a relation between a neuron population and its dendrite location                                                                                                                                    |
| **hasAxonTerminalLocation**         | Expresses a relation between a neuron population and its axon terminal location (i.e., the location of the axon presynaptic element)                                                                           |
| **hasAxonSensoryLocation**          | Expresses a relation between a neuron population and its sensory axon terminal location (i.e., the location of the axon sensory subcellular element)                                                          |
| **hasAxonLeadingToSensoryTerminal** | Expresses a relation between a neuron population and the anatomical structure where its axon distally projects to the axon sensory terminal (i.e., the axon sensory subcellular element).                      |
| **hasConnectedLocation**            | Expresses a relation between a neuron population and any of its connected location listed above. This property serves as the super property of the properties listed above and not used for direct assertions. |

### **Other Phenotypes in SCKAN**

| Phenotypic Relation                | Description                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **hasNeuronalPhenotype**     | Expresses a relation between a neuron population and its ANS phenotype<br />- **Pre-Ganglionic or Post-Ganglionic**, **Sympathetic** or **Parasympathetic**<br />- Combinations like  **Sympathetic  Pre-Ganglionic** or **Parasympathetic Post-Ganglionic**                                    |
| **hasFunctionalCircuitRole** | Expresses a relation between a neuron population and its immediate effect on postsynaptic cells<br />- **Excitatory** or **Inhibitory**                                                                                                                                                                           |
| **hasCircuitRole**           | Expresses a relation between a neuron population and its circuit role phenotype<br />- Possible phenotypes are: **Intrinsic**, **Motor**, **Sensory**, or **Projection**                                                                                                                               |
| **hasProjection**            | Expresses a relation between a neuron population and a brain region where the neuron population sends axons towards<br />- **Spinal cord descending projection**, **Spinal cord ascending projection**<br />- **Intestino fugal projection**, **Anterior projecting**, **Posterior projecting** |
| **isObservedInSpecies**      | Expresses a relationship between a neuron type and a taxon. Used when a neuron population has een observed in a specific species.<br />- **Species** from **NCBI Taxonomy**                                                                                                                                         |
| **hasPhenotypicSex**         | Expresses a relationship between a neuron type and a biological sex. Used when a neuron population has been observed in a specific sex.<br />- **Male** or ****Female** **from **PATO**                                                                                                                      |
| **hasForwardConnection**     | Expresses a relationship to specify the synaptic forward connection from a pre-ganglionic neuron population to a post-ganglionic neuron population.                                                                                                                                                                             |

## SCKAN Competency Queries

SCKAN was designed to answer questions relevant to ANS connectivity. To demonstrate the functionality of SCKAN we developed a set of competency queries (CQ). In this section, we work through the CQs and examine the results. For each question, we provide the SPARQL query and a visualization of the query results. All of the SPARQL queries listed in this section are written using Simple SCKAN.

* Navigate to [this directory](./competency-queries/CQ1-Query.rq)for the raw query results in CSV format, along with the visual diagrams produced by the SPARQL queries included in this section.
* For the first three CQs we used [RAWGraphs](https://github.com/rawgraphs/rawgraphs-app) to produce the visual query results. RawGraphs is an open-source tool to create customized vector-based visualizations on top of $d3.js$ library.
* For the fourth CQ, we developed a custom visualizer utilizing [GraphViz](https://graphviz.org/), an open-source graph visualization library to represent structural information as diagrams.

### *CQ-1: What connections terminate in the bladder? What are the origins of those connections and what are the exact parts of the organ the connections terminate. What nerves are involved in those connections?*

The SPARQL query in **Listing 0** retrieves the required information by filtering the set of neuron populations in SCKAN with specific connections to the urinary bladder. The query identifies populations with their soma located in one region (Region A), their axon terminal or axon sensory terminal located in another region (Region B), and the axon potentially passing through or being located in a third region (Region C) which must be a nerve. The focus of the query is on neuron populations where Region B must satisfy the constraint of being a part of the bladder.

```SPARQL
## CQ1: What connections terminate in bladder? What are the origins of those connections and what are the exact parts of the organ the connections terminate? What nerves are involved in those connections?
## Query for CQ1: This query returns the neuron populations with their origin (soma) located in Region A, axon terminal in Region B, and axon located in region C which is a nerve. Region B of the populations must be a part of the bladder. 

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ilxtr: <http://uri.interlex.org/tgbugs/uris/readable/>

## Select distinct neuron populations and their associated regions
SELECT DISTINCT ?Neuron_ID 
?A_IRI ?A_Label ?B_IRI ?B_Label ?C_IRI ?C_Label 
?Target_Organ_IRI ?Target_Organ_Label
{        
    ## Filter to only include neuron populations connected to the urinary bladder.
    FILTER (?Target_Organ_Label = 'urinary bladder').
  
    ## Filter to only include C (Axon Location) that is a nerve, nerve plaxus or fiber. 
    FILTER (?C_Type in ('nerve', 'nerve plexus', 'nerve fiber'))

    ## Only consider neuron populations from SCKAN. 
    VALUES ?SCKAN_Neuron { ilxtr:NeuronSparcNlp ilxtr:NeuronApinatSimple }
  
    ?Neuron_ID rdfs:subClassOf+ ?SCKAN_Neuron;
               ilxtr:hasSomaLocation ?A_IRI;
               ilxtr:hasAxonLocation | ilxtr:hasAxonToSensoryTerminal ?C_IRI;   
               ilxtr:hasAxonTerminalLocation | ilxtr:hasAxonSensoryLocation ?B_IRI.
  
    ?A_IRI rdfs:label ?A_Label. ?B_IRI rdfs:label ?B_Label. ?C_IRI rdfs:label ?C_Label.
  
    ## Check if region C is a subclass* of a nerve, nerve plexus, fiber.
    ?C_IRI rdfs:subClassOf*/rdfs:label ?C_Type.
                                        
    ## Check if Region B is part of the target organ, either directly or transitively.
    ?B_IRI (ilxtr:isPartOf*)/rdfs:label ?Target_Organ_Label.
    ?Target_Organ_IRI rdfs:label ?Target_Organ_Label
}
ORDER BY ?Neuron_ID ?A_Label ?B_Label ?C_Label
```

> **Listing 0:** The SPARQL query for CQ1 to retrieve the connections projecting to the urinary bladder, detailing their origins (Region A), termination points within the bladder (Region B), and axon locations within associated nerves (Region C). [View the SPARQL code here](./competency-queries/CQ1-Query.rq).

#### **Raw Query Result for *CQ-1***

The following table shows the first 14 rows of the query result for *CQ-1* ([view the full result in CSV](./competency-queries/CQ1-Results.csv)). The query for all populations that terminate in the bladder returns connections contained within $6$ distinct neuron populations that have axon terminals in some part of the bladder.

| Neuron_ID                                                         | A_IRI                                         | A_Label                           | B_IRI                                         | B_Label                                        | C_IRI                                         | C_Label                 | Target_Organ_IRI                              | Target_Organ_Label    |
| ----------------------------------------------------------------- | --------------------------------------------- | --------------------------------- | --------------------------------------------- | ---------------------------------------------- | --------------------------------------------- | ----------------------- | --------------------------------------------- | --------------------- |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-1  | http://purl.obolibrary.org/obo/UBERON_0016508 | pelvic ganglion                   | http://uri.interlex.org/base/ilx_0738433      | Dome of the Bladder                            | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-1  | http://purl.obolibrary.org/obo/UBERON_0016508 | pelvic ganglion                   | http://purl.obolibrary.org/obo/UBERON_0001258 | neck of urinary bladder                        | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://uri.interlex.org/base/ilx_0793663      | Arteriole in connective tissue of bladder neck | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://uri.interlex.org/base/ilx_0793663      | Arteriole in connective tissue of bladder neck | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://uri.interlex.org/base/ilx_0738433      | Dome of the Bladder                            | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://uri.interlex.org/base/ilx_0738433      | Dome of the Bladder                            | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://purl.obolibrary.org/obo/UBERON_0001258 | neck of urinary bladder                        | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://purl.obolibrary.org/obo/UBERON_0002860 | first sacral dorsal root ganglion | http://purl.obolibrary.org/obo/UBERON_0001258 | neck of urinary bladder                        | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://uri.interlex.org/base/ilx_0793663      | Arteriole in connective tissue of bladder neck | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://uri.interlex.org/base/ilx_0793663      | Arteriole in connective tissue of bladder neck | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://uri.interlex.org/base/ilx_0738433      | Dome of the Bladder                            | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://uri.interlex.org/base/ilx_0738433      | Dome of the Bladder                            | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://purl.obolibrary.org/obo/UBERON_0001258 | neck of urinary bladder                        | http://uri.interlex.org/base/ilx_0793559      | bladder nerve           | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder       |
| http://uri.interlex.org/tgbugs/uris/readable/neuron-type-keast-10 | http://uri.interlex.org/base/ilx_0793360      | sixth lumbar dorsal root ganglion | http://purl.obolibrary.org/obo/UBERON_0001258 | neck of urinary bladder                        | http://purl.obolibrary.org/obo/UBERON_0018675 | pelvic splanchnic nerve | http://purl.obolibrary.org/obo/UBERON_0001255 | urinary bladder<br /> |

> **Table 0:** The first 14 rows of the query result for *CQ-1* ([view the full result in CSV](./competency-queries/CQ1-Results.csv))

#### Visual Query Result for CQ-1

The visual query results in Figure~\ref{fig:viz-cq1} shows the origins of these connections (right panel) and the nerves through which they traverse, including bladder nerve, pelvic splanchnic nerve, and hypogastric nerve.

![1734515894426](image/README/1734515894426.jpg)

> **Figure 3:** A visual query result for *CQ-1*, showing the neural pathways terminating in the bladder, illustrating their origins (nodes on the right), nerve involvement (nodes in the middle), and specific bladder regions targeted (nodes on the left). Nodes are color-coded to distinguish the connections based on the terminal structures in the bladder.



Explore the [Query examples in Jupyter Notebook](example-queries/sckan-sparql-query-examples.ipynb).
