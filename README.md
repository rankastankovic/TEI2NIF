# TEI2NIF: TEI transformation to LLOD : ELTeC level-2 use case for por, slv, srp 
The contents of this repository is the result of virtual mobility within NexusLingurum COST Action CA CA18209. The main goal  was to publish 300 novels from ELTeC - European Literary Text Collection from period 1840-1920., 100 per 3 languages (Serbian, Slovenian adn Portugese) as open linked data according to best practice and guidelines fostered by NexusLinguarum. 

Namely, ELTeC novels in so-called level-2 format were developed within COST Action CA16204 Distant Reading for European Literary History (2017 - 2022) and they are valuable resource to be published in LLOD. 
Three data collection were used:	https://github.com/COST-ELTeC/ELTeC-por/tree/master/level2, https://github.com/COST-ELTeC/ELTeC-slv/tree/master/level2, https://github.com/COST-ELTeC/ELTeC-srp/tree/master/level2.

Metadata for selected novels are linked with already available in Wikidata, named WikiELTeC, as described in "From ELTeC Text Collection Metadata and Named Entities to Linked-data (and Back)" by Milica Ikonic Nešić, Ranka Stanković, Christof Schoch, Mihailo Škorić, http://www.lrec-conf.org/proceedings/lrec2022/workshops/LDL/pdf/2022.ldl2022-1.2.pdf

The NLP Interchange Format (NIF), designed to facilitate the integration of NLP tools in knowledge extraction pipelines, provides support for part-of-speech tagging, lemmatization and entity annotation, enabling ELTeC level-2 layers transformation. 
Python code, including Jupyter notebook, is prepared for export from XML/TEI into NIF, available in colab subfolder. For Wikidata management mkwikidata library was used and for working with RDF  rdflib .

Samples of NIF files are in NIF subfolder.
