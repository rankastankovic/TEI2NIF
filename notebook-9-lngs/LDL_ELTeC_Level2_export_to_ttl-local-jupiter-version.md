# Export srpELTeC to NIF

* Virtual mobility Grant holder: ***Ranka Stanković ***
* Virtual mobility grant: E-COST-GRANT-CA18209-4349b689, "European literary text collection ELTEC transformation and publishing as linguistic linked
open data: use case for 3 languages"



* Start and end date: 01/08/2022 to 15/10/2022
* COST Action: CA18209
* collaborator: Milica Ikonić Nešić
* Suported by Max Ionov, Christian Chiarcos and others from WG1, Task 5

**Description**: Source files are 300 novels from ELTeC collection (100 per 3 languages: sr, sl, pt) coded in XML/TEI level-2  https://github.com/COST-ELTeC/ELTeC-srp/tree/master/level2 (POS taged, lemmatised, with annotated named entities, also supplied by metadata in TeiHeader and in Wikidata https://www.wikidata.org/wiki/Wikidata:WikiProject_ELTeC)

## install & import part


```python
# for importing/clonning repository with novels
#!pip install gitpython
```


```python
#!pip3 install rdflib sparqlwrapper pydotplus graphviz
```


```python
#!pip install mkwikidata
```


```python
#ovo sam odustala
#!pip install spacyopentapioca # for NEL
```


```python
# os,sys,glob
import os
import os.path
from os import path
import sys
import glob 
import locale
import spacy
import string

# xml 
from lxml import etree
# rdf
import rdflib
from rdflib import Graph
from rdflib.namespace import RDF, RDFS, XSD, OWL, DCAT, FOAF
from rdflib import URIRef, BNode, Literal
import networkx as nx
import io
import pydotplus
from IPython.display import display, Image
from rdflib.tools.rdf2dot import rdf2dot
import mkwikidata
import re

ITSRDF=rdflib.Namespace("http://www.w3.org/2005/11/its/rdf#")
NIF = rdflib.Namespace("http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core#")
# NERD= rdflib.Namespace("http://nerd.eurecom.fr/ontology#")
DC = rdflib.Namespace("http://purl.org/dc/elements/1.1/") 
DCT = rdflib.Namespace("http://purl.org/dc/terms/") 
MS = rdflib.Namespace("http://w3id.org/meta-share/meta-share/") 
WD = rdflib.Namespace("http://www.wikidata.org/entity/")
#WD = rdflib.Namespace("http://www.wikidata.org/wiki/")
WDT = rdflib.Namespace("http://www.wikidata.org/prop/direct/")
DBO = rdflib.Namespace("https://dbpedia.org/ontology/")
OLIA = rdflib.Namespace("http://purl.org/olia/discourse/olia_discourse.owl#")
prov_uri = URIRef("http://llod.jerteh.rs/ELTEC/srp/NIF/")



nlp_nel = spacy.blank('en')
#nlp_nel.add_pipe('opentapioca')
```

# Metadata description

For metadata analysed: https://link.springer.com/chapter/10.1007/978-3-319-18818-8_20, lexmeta : https://lexbib.elex.is/wiki/LexMeta, 
https://github.com/pennyl67/LexMeta/blob/main/lexmeta.ttl

Metadata for collections described in Wikidata: https://www.wikidata.org/wiki/Wikidata:WikiProject_ELTeC (first edition, printed edition, ELTeC edition)

Key values prepared for this edition:
* ID - dct:identifier
* title - dct:title (from teiHeader : titleStmt)
* author (name) - ms:author
* authorQID - dc:creator,
* novelQID - linked as edition of novel 
* publisher - dct:publisher 
* licence - ms:LicenceTerms
* year - ms:publicationDate
* language - dc:Language, ms:Language
* collection - wdt:P1433 




--------------------------------------------

# Ontology mapping



Initially used:
https://github.com/NERD-project/nerd-ontology/blob/master/nerd.owl  but replaced with: 
http://purl.org/olia/discourse/olia_discourse.owl

* http://nerd.eurecom.fr/ontology#Person  (mapped to http://purl.org/olia/discourse/olia_discourse.owl#Person)
* http://nerd.eurecom.fr/ontology#Location (mapped to http://purl.org/olia/discourse/olia_discourse.owl#Space)
* http://nerd.eurecom.fr/ontology#Organization  (mapped to http://purl.org/olia/discourse/olia_discourse.owl#Organization)
* http://nerd.eurecom.fr/ontology#Event  (mapped to http://purl.org/olia/discourse/olia_discourse.owl#Event)
* ROLE - Names of posts and job titles (profession, nobility, office, military)
* DEMO - DEMO -  Demonyms, names of kinds of people: national, regional, political (Frenchwoman; German; Parisiens;...) 
* WORK - titles of books, songs, plays, newspaper, paintings, sculptures  and other creations

Additional:
* https://dbpedia.org/ontology/Person, https://www.wikidata.org/wiki/Q5, https://schema.org/Person
* https://dbpedia.org/ontology/Place, https://www.wikidata.org/wiki/Q7884789, https://schema.org/Place
* https://dbpedia.org/ontology/Organisation, https://www.wikidata.org/wiki/Q43229, https://schema.org/Organization
* https://dbpedia.org/ontology/Event, https://www.wikidata.org/wiki/Q1656682, https://schema.org/Event
* https://dbpedia.org/ontology/Profession, https://www.wikidata.org/wiki/Q28640, https://schema.org/Occupation
* demonym as property only: https://dbpedia.org/ontology/demonym, https://www.wikidata.org/wiki/Q217438
* https://dbpedia.org/ontology/Work, https://www.wikidata.org/wiki/Q386724, https://schema.org/CreativeWork, 


Rizzo, Giuseppe, Raphaël Troncy, Sebastian Hellmann, and Martin Bruemmer. "NERD meets NIF: Lifting NLP extraction results to the linked data cloud." In LDOW. 2012. http://ceur-ws.org/Vol-937/ldow2012-paper-02.pdf 

# General classes: Token, Sentence, NamedEntity

TEI+RDF+LLOD literature

https://content.iospress.com/articles/semantic-web/sw222859

* P. Ruiz Fabo, H. Bermúdez Sabel, C. Martínez Cantón and E. González-Blanco, The diachronic Spanish sonnet corpus: TEI and linked open data encoding, data distribution, and metrical findings, Digital Scholarship in the Humanities (2020). doi:10.1093/llc/fqaa035.


* S. Tittel, H. Bermúdez-Sabel and C. Chiarcos, Using RDFa to link text and dictionary data for medieval French, in: Proceedings of the Eleventh International Conference on Language Resources and Evaluation (LREC-2018), European Language Resources Association (ELRA), 2018.

* Khan, Anas Fahad, Christian Chiarcos, Thierry Declerck, Daniela Gifu, Elena González-Blanco García, Jorge Gracia, Maxim Ionov et al. "When Linguistics Meets Web Technologies. Recent advances in Modelling Linguistic Linked Data." (2021).



```python
# manage reading token properties from token (xml node) and generate graf triples
class Token:    

    def __init__(self, token,cur_index):
        #self.id = token.xpath("./@xml:id",namespaces={'xml':'http://www.w3.org/XML/1998/namespace'})[0]
        #\n*\s*([^\s\n]*)\n*
        self.text =  re.sub('\s+', '', token.text, re.MULTILINE)

        self.pos = token.xpath("./@pos")[0]
        self.lemma = "" if self.pos == "PUNCT" or 'lemma' not in token.attrib else token.xpath("./@lemma")[0]
        self.join = token.xpath("./@join")
        # fro NER
        self.parent = token.getparent()
        self.first_NER_node=False
        if  self.parent.xpath("name()")=="rs" :
           if ( list(self.parent)[0]==token and self.parent.xpath("./@type")): # for NER important 
              self.first_NER_node = True
        self.index_start=cur_index
        self.index_end=self.index_start+len(self.text)

    # create graph triples for this token
    def init_gtoken(self,g,base_url):
      gtoken= URIRef(base_url+"#char={0},{1}".format(self.index_start,self.index_end))
      g.add( (gtoken, RDF.type, NIF.Word ) )
      g.add( (gtoken, RDF.type, NIF.RFC5147String  ) )
     # g.add( (gtoken,RDF.type, NIF.OffsetBasedString) ) Christian sugestion to remove
      g.add( (gtoken, NIF.anchorOf, Literal(self.text, datatype=XSD.string)) )
      g.add( (gtoken, NIF.beginIndex, Literal(self.index_start, datatype=XSD.nonNegativeInteger)) )
      g.add( (gtoken, NIF.endIndex, Literal(self.index_end, datatype=XSD.nonNegativeInteger)) )
      g.add( (gtoken, NIF.posTag, Literal(self.pos, datatype=XSD.string)))
      if self.lemma!="":
        g.add( (gtoken, NIF.lemma, Literal(self.lemma, datatype=XSD.string)))      

      return gtoken
```


```python
# manage reading sentence properties from sent (xml) and generate graph triples
class Sentence:    

    def __init__(self, sent,cur_index):
        #self.id = sent.xpath("./@xml:id",namespaces={'xml':'http://www.w3.org/XML/1998/namespace'})[0]
        self.tokens=sent.xpath(".//*[local-name()='w' or local-name()='pc']")
        self.text=""
        self.index_start=cur_index
        
        self.otokens = []
        # loop all tokens in sentence
        for token in self.tokens:
          otoken=Token(token,cur_index)
          self.otokens.append(otoken)
          # concatenate to sent
          self.text +=otoken.text      
          cur_index+=len(otoken.text)
          if not otoken.join:
            self.text+=" "
            cur_index+=1
        self.index_end = cur_index   

    # create graph triples for this sentence
    def init_gsent(self,g,base_url):
      gsent= URIRef(base_url+"#char={0},{1}".format(self.index_start,self.index_start+len(self.text))) # add sent ID
      g.add( (gsent, RDF.type, NIF.Sentence) )
      g.add( (gsent, RDF.type,NIF.Context) )
      g.add( (gsent, RDF.type, NIF.RFC5147String) )
     # g.add( (gsent, RDF.type, NIF.OffsetBasedString) )
      # NIF.anchorOf, NIF.endIndex later
      g.add( (gsent, NIF.beginIndex, Literal(self.index_start, datatype=XSD.nonNegativeInteger)	))

      
      return gsent
```


```python
# manage NE named entities: PERS LOC ORG EVENT ROLE DEMO WORK 
class NamedEntity:

  def __init__(self, parent,cur_index):
      self.index_start=cur_index
      self.text=""
      for ne in parent.getchildren():
         self.text +=ne.text
         self.join = ne.xpath("./@join") 
         if not self.join:
            self.text+=" "
      self.text=self.text.rstrip()      
      self.index_end=self.index_start+len(self.text)
      if (parent.xpath("./@type")): 
         self.ne_type = parent.xpath("./@type")[0] 
  
  # create graph triples for this token
  def init_gne(self,g, base_url):
      gne= URIRef(base_url+"#char={0},{1}".format(self.index_start,self.index_end) )   
      g.add( (gne, RDF.type, NIF.RFC5147String) )  
     # g.add( (gne, RDF.type, NIF.OffsetBasedString) )
     # g.add((gne, RDF.type, NIF.EntityOccurrence )) not in NIF2.0 FK and CC sugested to eliminate and search by properties
      g.add( (gne, NIF.anchorOf,Literal(self.text, datatype=XSD.string)));
      g.add( (gne, NIF.beginIndex,Literal(self.index_start, datatype=XSD.nonNegativeInteger)))  
      g.add( (gne, NIF.endIndex,Literal(self.index_end, datatype=XSD.nonNegativeInteger)))  
      if self.ne_type in ['PERS', 'PERSON']:
         g.add( (gne, ITSRDF.taClassRef, OLIA.Person))         
         g.add( (gne, ITSRDF.taClassRef, WD.Q5))
         g.add( (gne, ITSRDF.taClassRef, DBO.Person))
      elif self.ne_type in ['LOC', 'LOCATION', 'PLACE', 'GPE']:
        # g.add( (gne, ITSRDF.taClassRef, NERD.Location))
        g.add( (gne, ITSRDF.taClassRef, OLIA.Space))
        g.add( (gne, ITSRDF.taClassRef, WD.Q7884789))
        g.add( (gne, ITSRDF.taClassRef, DBO.Place))
      elif self.ne_type in ['ORG', 'ORGANISATION']:
        g.add( (gne, ITSRDF.taClassRef, OLIA.Organization))
        g.add( (gne, ITSRDF.taClassRef, WD.Q43229))
        g.add( (gne, ITSRDF.taClassRef, DBO.Organisation))
      elif self.ne_type in ['EVENT']:
        g.add( (gne, ITSRDF.taClassRef, OLIA.Event))
        g.add( (gne, ITSRDF.taClassRef, WD.Q1656682))
        g.add( (gne, ITSRDF.taClassRef, DBO.Event))
      elif self.ne_type in ['ROLE']:
        # g.add( (gne, ITSRDF.taClassRef, Literal("<" +self.ne_type+">", datatype=XSD.string ))) # just string
        g.add( (gne, ITSRDF.taClassRef, WD.Q28640))    # profession, not exact, it could be title as well
        g.add( (gne, ITSRDF.taClassRef, DBO.Profession))
      elif self.ne_type in ['DEMO']:
        # g.add( (gne, ITSRDF.taClassRef, Literal("<" +self.ne_type+">", datatype=XSD.string ))) # just string
        g.add( (gne, ITSRDF.taClassRef, WD.Q217438 ))
        g.add( (gne, ITSRDF.taClassRef, DBO.demonym)) # check this
      elif self.ne_type in ['WORK']:
        # g.add( (gne, ITSRDF.taClassRef, Literal("<" +self.ne_type+">", datatype=XSD.string ))) # just string
        g.add( (gne, ITSRDF.taClassRef, WD.Q386724 ))
        g.add( (gne, ITSRDF.taClassRef, DBO.Work))
      else :  # something not expected
         g.add( (gne, ITSRDF.taClassRef, Literal("<" +self.ne_type+">", datatype=XSD.string ))) 

      # added for Tapioca 21.10.2022
#     nel= getTapioca(self.text)
#      if nel != None:
#        g.add( (gne, ITSRDF.taIdentRef, URIRef("http://www.wikidata.org/entity/"+nel) ))
      return gne     
```


```python
collectionQIDs ={ 
    "srp":"Q106936149",  # extended Q109123373 
    "por":"Q111095586",  
    "slv":"Q111046825",  
    "hun":"Q111136029",
    "eng":"Q111271624", 
    "fra":"Q111271916", 
    "deu":"Q111238785", 
    "rom":"Q116197427", 
    "pol":"Q116197411",
    "spa":"Q116245956"}  
lng2s ={ 
    "srp":"sr",  
    "por":"pt",  
    "slv":"sl",  
    "hun":"hu",
    "eng":"en", 
    "fra":"fr", 
    "deu":"de", 
    "rom":"ro",
    "pol":"pl",
    "spa":"es"}
```

# Class Novel


```python
# manage reading novel and generate graph triples
class Novel:    

    def __init__(self, file_path_name, lng):
       self.el = etree.parse(file_path_name)
       self.id=self.el.xpath("./@xml:id",namespaces={'xml':'http://www.w3.org/XML/1998/namespace'})[0].replace(".xml","")
       # do the senteces
       self.sentences = self.el.xpath("//*[local-name()='s']")
       self.text=""
       self.index_start=0
       self.lng=lng
       self.file_name =os.path.basename(file_path_name).replace(".xml",".txt")
       self.url="http://llod.jerteh.rs/ELTEC/"+lng+"/NIF/" +self.id +".txt"  #self.file_name

       #  metadata  
       self.titleStmt=self.el.xpath("//*[local-name()='titleStmt']")[0]  
       self.title=self.titleStmt.xpath("*[local-name()='title']")[0]   
       # author from TEI file
       self.author=self.titleStmt.xpath("*[local-name()='author']")[0]     
       self.publicationStmt=self.el.xpath("//*[local-name()='publicationStmt']")[0] #publicationStmt
       if self.publicationStmt.xpath("*[local-name()='publisher']") :
         pub = self.publicationStmt.xpath("*[local-name()='publisher']")[0]  
         self.publisher=pub.text
       else:
         self.publisher=""

    # graph initialisation for one novel 
    def  init_gnovel(self,g):      
       # for graph
       gnovel= URIRef(self.url) # +"{0}_{1}".format(0,) 
       #g.add( (gnovel, RDF.type, NIF.OffsetBasedString) )
       g.add( (gnovel, RDF.type, NIF.RFC5147String) )
       g.add( (gnovel, RDF.type, NIF.Context  ) ) 
       g.add( (gnovel, NIF.beginIndex, Literal("0",datatype=XSD.nonNegativeInteger) )) 
       g.add( (gnovel, DCT.identifier,  Literal(self.id,datatype=XSD.string)    ))
   
       g.add( (gnovel, DCT.title, Literal(self.title.text,datatype=XSD.string)) )
       # language
       g.add((gnovel,DC.Language, Literal(lng2s[lng],datatype=XSD.string)))
       g.add((gnovel,MS.Language, Literal(lng2s[lng],datatype=XSD.string)))
       # collection
       g.add((gnovel,WDT.P1433,URIRef("http://www.wikidata.org/entity/"+collectionQIDs[lng])))
       # from Wikidata read 
       query_result=getWiki(self.id,lng2s[lng],collectionQIDs[lng])
       for result in query_result["results"]["bindings"]:
         novelQID =  URIRef( wiki2entity(result["novel"]["value"])) if ("novel" in result) else None
         authorQID = URIRef( result["author"]["value"]) if ("author" in result) else None
         year =  result["year"]["value"] if ("year" in result) else None
         licence = URIRef(wiki2entity(result["licence"]["value"])) if ("licence" in result) else None
         g.add( (gnovel, WDT.P31, WD.Q3331189) ) # is edition
         g.add( (novelQID, WDT.P747, gnovel) ) # has edition
         g.add( (gnovel,DC.creator,authorQID) )
         g.add( (gnovel,MS.publicationDate, Literal(year,datatype=XSD.nonNegativeInteger)) )
         g.add( (gnovel,MS.LicenceTerms, licence) )

       g.add((gnovel,MS.author,Literal(wiki2entity(self.author.text),datatype=XSD.string)))
        
       if self.publisher !="" :
         g.add( (gnovel, DCT.publisher, Literal(wiki2entity(self.publisher),datatype=XSD.string)) )
        
       return gnovel
```





# Initialisation and metadata 

\


```python
def wiki2entity(urlWiki):
  return urlWiki.replace("http://www.wikidata.org/wiki/","http://www.wikidata.org/entity/")
def entity2wiki(urlWiki):
  return urlWiki.replace("http://www.wikidata.org/entity/","http://www.wikidata.org/wiki/")

```


```python
# read wikidata metapodataka
# novel_id - string (in wikidata "volume")
# lng2: sl, sr, pt,....
# collectionQID: Q111046825, Q106936149, Q111095586 
def getWiki(novel_id, lng2, collectionQID):
  query='''
  SELECT DISTINCT ?novel ?author ?edition (YEAR(?date) as ?year) ?licence
  WHERE {
    # published in (P1433) srpELTeC collection(Q106936149)
    ?novel wdt:P31 wd:Q7725634; # is literary work
         wdt:P747 ?edition; # has edition  
         wdt:P577  ?date;
         wdt:P50 ?author.
    OPTIONAL {?edition wdt:P275 ?licence.}
    ?edition wdt:P478 "'''+ novel_id +'''".   
    ?edition wdt:P1433 wd:'''+collectionQID+'''. # published in
    SERVICE wikibase:label { bd:serviceParam wikibase:language "'''+lng2+'''". }
  }
  '''
  query_result = mkwikidata.run_query(query, params={ })
  # print(query)
  return query_result
```


```python
# test wiki read
#getWiki("SRP18520", "sr", "Q106936149")
#getWiki("POR0065", "pt", "Q111095586")
```


```python
# get QID for named entity using opentapioca
# ne_text - named enetity
# added 21.10.2022.
# nlp_nel global object
# function not used, tapioca has a limited number of calls
def getTapioca(ne_text):
  doc = nlp_nel(ne_text)
  for ent_nel in doc.ents:
    return ent_nel.kb_id_   # return just first
```


```python
#print(getTapioca('Београд je Srbija'))
```

# Main function to write ttl file

Guidelines for developing NIF-based NLP services 
https://www.w3.org/2015/09/bpmlod-reports/nif-based-nlp-webservices/


```python
# create graph, ...
def write_gnovel(file_path_name,lng,sent_num):
  g = Graph()
  g.bind('itsrdf', ITSRDF)
  g.bind('nif', NIF)
  g.bind('olia', OLIA)
  g.bind('dc',DC) 
  g.bind('dct',DCT)  
  g.bind('ms',MS)  
  g.bind('wd', WD)
  g.bind('wdt', WDT)
  g.bind('dbo', DBO)
  g.bind('eltec', ELTEC)

  onovel = Novel(file_path_name,lng)
  
  # insert initial triples for novel
  gnovel = onovel.init_gnovel(g)
  gsent_before=None
  one=None

  scount=0 # just for testing, remove later
  cur_index=0 # position in text
  for sent in onovel.sentences: # loop all sentences in one novel
    scount+=1
    if scount > sent_num:
      break    # break after few sentences for test
    osent=Sentence(sent,cur_index)
    cur_index=osent.index_end 
    gsent = osent.init_gsent(g,onovel.url)  
    if gsent_before != None:
        g.add( (gsent, NIF.previousSentence, gsent_before ) )
        g.add( (gsent_before, NIF.nextSentence, gsent ) )
    g.add( (gsent, NIF.referenceContext, gnovel)) 
    gtoken_before=None 
    # loop all tokens in sentence
    for otoken in osent.otokens:
       gtoken = otoken.init_gtoken(g,onovel.url)
       g.add( (gtoken, NIF.referenceContext ,gnovel))
       g.add( (gtoken, NIF.sentence, gsent)) 
       g.add( (gsent, NIF.word, gtoken))
       # relate token: previous, next
       if gtoken_before!=None:
         g.add((gtoken_before,NIF.nextWord,gtoken))
         g.add((gtoken, NIF.previousWord, gtoken_before))
       # NER
       if  otoken.first_NER_node:
          one=NamedEntity(otoken.parent,otoken.index_start)
          gne = one.init_gne(g,onovel.url)
          g.add( (gne, NIF.referenceContext ,gnovel))       
       if (one):
          if ( otoken.index_start>=one.index_start and otoken.index_end<=one.index_end):
             g.add( (gtoken, NIF.subString,  gne) )
       gtoken_before =gtoken
       # end of token
  
    # finish sentence graph  sid,len(tokens),sent_text
    # anchorOf / isString
    g.add( (gsent, NIF.anchorOf  , Literal(osent.text, datatype=XSD.string)) )
    g.add( (gsent, NIF.endIndex  , Literal(osent.index_start+len(osent.text), datatype=XSD.string)) )

    # concatenate to novel
    onovel.text+= osent.text+" "
    cur_index+=1
    gsent_before = gsent
    # end of sentence
  
  # finish novel      
  g.add( (gnovel, NIF.endIndex  , Literal(len(onovel.text.rstrip()),datatype=XSD.nonNegativeInteger)))
  # Do we use NIF.anchorOf or NIF.isString
  g.add( (gnovel, NIF.isString  , Literal(onovel.text.rstrip(), datatype=XSD.string)) )
  #fn=file_path_name.replace(".xml",".txt").replace("\\","/").replace("/level2/","/NIF/")
  fn= os.path.dirname(file_path_name).replace("/level2","/NIF")+"/"+onovel.id+".txt"
  print(fn)
  f_txt = open(fn, "w")
  f_txt.write(onovel.text.rstrip())
  f_txt.close()
  # write RDF file
  #file_path_name_out=file_path_name.replace(".xml",".ttl").replace("\\","/").replace("/level2/","/NIF/")
  file_path_name_out=fn.replace(".txt",".ttl")
  g.serialize(destination=file_path_name_out)
```


```python

```

# Select language repo
This task TEI level-2: https://github.com/COST-ELTeC/ELTeC-srp, 
https://github.com/COST-ELTeC/ELTeC-slv, 
https://github.com/COST-ELTeC/ELTeC-por. 

Similar structure, possible for later adaptation, also level-2: https://github.com/COST-ELTeC/ELTeC-deu, https://github.com/COST-ELTeC/ELTeC-eng, https://github.com/COST-ELTeC/ELTeC-fra, 
https://github.com/COST-ELTeC/ELTeC-hun, https://github.com/COST-ELTeC/ELTeC-pol, https://github.com/COST-ELTeC/ELTeC-rom, https://github.com/COST-ELTeC/ELTeC-spa.


```python
# test on small portion of one file or list of files
lng="deu"
ELTEC = rdflib.Namespace("http://llod.jerteh.rs/ELTEC/"+lng+"/NIF/")
# file_name='D:/Korpusi/ELTEC/ELTeC-'+lng+'-master/level2/ROM096-L2.xml' # 96,100
p='D:/Korpusi/ELTEC/ELTeC-'+lng+'-master/level2/'
l=["DEU047"]
for f in l:
    file_name=p+f+".xml"
    el = etree.parse(file_name)
    novel_id=el.xpath("./@xml:id",namespaces={'xml':'http://www.w3.org/XML/1998/namespace'})[0]
    write_gnovel(file_name,lng,1000)
#novel_id
```

    D:/Korpusi/ELTEC/ELTeC-deu-master/NIF/DEU047.txt
    


```python
# list all xml files in level-2 only and generate NIF (~2 hours per language full files)
lng= "slv" # deu , eng, hun, pol, por, rom, slv, spa, srp,       fra (ne)
# depending on collection language
ELTEC = rdflib.Namespace("http://llod.jerteh.rs/ELTEC/"+lng+"/NIF/")
lstLevel2=sorted(glob.glob('D:/Korpusi/ELTEC/ELTeC-'+lng+'-master/level2/*.xml'), key=locale.strxfrm)
print (len(lstLevel2))
for inp_file in lstLevel2:
  print(inp_file)
  try:
      write_gnovel(inp_file,lng,1000) # for test use just few sentences
  except:
     print("........................... An exception occurred")


 
```

    100
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00024-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00024.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00048-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00048.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00058-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00058.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00072-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00072.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00090-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00090.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00092-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00092.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00094-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00094.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00098-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00098.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00099-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00099.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00103-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00103.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00111-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00111.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00112-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00112.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00122-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00122.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00126-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00126.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00132-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00132.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00135-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00135.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00136-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00136.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00172-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00172.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00174-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00174.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00187-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00187.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00194-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00194.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00216-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00216.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00217-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00217.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00227-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00227.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00231-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00231.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00234-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00234.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00240-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00240.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00273-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00273.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00278-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00278.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00279-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00279.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00310-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00310.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00324-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00324.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00325-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00325.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00345-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00345.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00352-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00352.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00355-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00355.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00363-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00363.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00364-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00364.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00373-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00373.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00391-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00391.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00398-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00398.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00401-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00401.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00406-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00406.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00410-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00410.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00417-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00417.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00421-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00421.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00425-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00425.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00452-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00452.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00454-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00454.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00455-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00455.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00456-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00456.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00458-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00458.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00459-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00459.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00460-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00460.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00461-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00461.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00473-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00473.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00476-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00476.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00478-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00478.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00483-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00483.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00496-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00496.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00497-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00497.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00498-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00498.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00502-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00502.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00506-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00506.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV00526-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV00526.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10001-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10001.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10002-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10002.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10003-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10003.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10004-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10004.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10005-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10005.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10006-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10006.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10007-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10007.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10008-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10008.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10009-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10009.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10010-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10010.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10011-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10011.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10012-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10012.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10013-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10013.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10014-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10014.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10015-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10015.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10016-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10016.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10017-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10017.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10018-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10018.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10019-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10019.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10020-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10020.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10021-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10021.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10022-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10022.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10023-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10023.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10024-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10024.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10025-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10025.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10026-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10026.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10027-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10027.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10028-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10028.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10029-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10029.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10030-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10030.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10031-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10031.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV10032-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV10032.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV20001-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV20001.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV20002-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV20002.txt
    D:/Korpusi/ELTEC/ELTeC-slv-master/level2\SLV30001-L2.xml
    D:/Korpusi/ELTEC/ELTeC-slv-master/NIF/SLV30001.txt
    


```python
#!zip -r NIF-eng.zip NIF-srp
```
