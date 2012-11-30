RDF
===
.. sectionauthor:: Alvaro Graves <alvaro@graves.cl>

This chapter shows how to use data available in RDF (Resource Description Framework)

(Very) brief description of what is RDF
---------------------------------------

The Resource Description Framework is a W3C specification to describe information 
on the Web. It is a graph-based language that uses **URIs** (Universal Resource Identifier)
to identify resources such as web pages, people, past wednesday and describe information
about such resources. Thus for example, Tim Berners-Lee (the inventor of the
Web) is identified by `http://www.w3.org/People/Berners-Lee/card#i
<http://www.w3.org/People/Berners-Lee/card#i>`_

The basic unit of information in RDF is a *triple*, which consists in three
Elements: The first one identifies a *subject* (the resource that is being
described), the second describes the *predicate*, a type of relation or attribute the subject has. Finally, the third describes
the *object*, which is the value or resource associated to the subject via the
predicate. For example, if we want to say that "the prefered name for Tim
Berners-Lee is 'Tim'" we can create the following triple:

 <http://www.w3.org/People/Berners-Lee/card#i> <http://www.w3.org/2004/02/skos/core#prefLabel> "Tim"

The "<" and ">" characters encloses a URI. Thus, we can add more information (say, "Tim has the twitter account http://twitter.com/timberners_lee")
simply by adding more triples

 <http://www.w3.org/People/Berners-Lee/card#i> <http://xmlns.com/foaf/0.1/account> <http://twitter.com/timberners_lee>

Graphically what we have is a graph where each URI can be linked to others of
can be given values in different values.

.. figure:: http://farm9.staticflickr.com/8301/7850931064_33a5aedb7c_o_d.png

Further Reading
^^^^^^^^^^^^^^^

For a more comprehensive and detailed description of RDF, please refer to the
`RDF Primer <http://www.w3.org/TR/rdf-primer/>`_ which describes in more details
URIs, blank nodes, literals, etc.


Obtaining data from SPARQL endpoints
------------------------------------

Data in RDF can be obtained as `Dereferenceable URIs
<http://en.wikipedia.org/wiki/Dereferenceable_URI>`_ but in most of the cases,
you might want to obtain it from a SPARQL endpoint, which is a service on the
web (usually open) where people can execute `SPARQL <http://en.wikipedia.org/wiki/SPARQL>`_ queries, which is a query
language for RDF. SPARQL has several similarities to SQL, making the adoption
from relational databases to RDF databases not so difficult. For example, we can
list all the people in a SPARQL endpoint by executing the following query::

 SELECT ?person ?name WHERE{
  ?person <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
  ?person <http://xmlns.com/foaf/0.1/name> ?name .
 }


Let's analyze this query: First, the reserved word ``SELECT`` defines that we want to bind the results to the
variables described using the question mark (``?person`` and ``?name``). After
the ``WHERE`` reserved word we describe the *graph patterns* we want to
specify. The first patterns contains a variable ``?person``, a predicate that 
says that this variable is of a certain type and an object which represents the class "Person". Then, this can be translated as "something that is of type Person".
 
The second pattern indicates that the same thing we want to retrieve **must
also** have a predicate http://xmlns.com/foaf/0.1/name (which associates a
resource with its name).
 
In simple terms, we ask for "something that is of type Person and it also has a
name". The results will be delivered in the variables described between the
``SELECT`` and ``WHERE`` reserved words. It is important to note that it is not
necessary to retrieve all the variables described in the graph patterns.
 
Simplifying SPARQL's syntax
^^^^^^^^^^^^^^^^^^^^^^^^^^^
 
 
The previous query can have a simpler syntax, first by using prefixes::
 
  PREFIX foaf: <http://xmlns.com/foaf/0.1/>
  PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  
  SELECT ?person ?name WHERE{
    ?person rdf:type foaf:Person .
    ?person foaf:name ?name .
  }
  
Also, since we are describing two patterns with the same subject, we don't need
to write it twice. We can use a semicolon for every pattern except the last one
(which has to have a period)::
  
  PREFIX foaf: <http://xmlns.com/foaf/0.1/>
  PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  
  SELECT ?person ?name WHERE{
    ?person rdf:type foaf:Person ;
            foaf:name ?name .
  }
    
  
Example: Querying information about Chilean poets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An importan source of RDF is `DBpedia <http://dbpedia.org>`_, a project focused
on extracting data from Wikipedia and publish it as RDF.
We can create a query to obtain the name of Chilean poets by querying::
  
  PREFIX dcterms: <http://purl.org/dc/terms/>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
  SELECT ?poet ?poetName WHERE{
       ?poet dcterms:subject <http://dbpedia.org/resource/Category:Chilean_poets>;
              rdfs:label ?poetName .
       FILTER (LANG(?poetName) = "en")
  }
  
The ``FILTER`` operator forces to obtain only the names in English, avoiding names in other languages. You can go to DBpedia's SPARQL endpoint (http://dbpedia.org/sparql) and execute
the query. The result should be something similar to
  
+--------------------------------------------------+-------------------------+
|                 poet                             |      poetName           |
+==================================================+=========================+
| http://dbpedia.org/resource/Nicanor_Parra        |    "Nicanor Parra"      |
+--------------------------------------------------+-------------------------+
| http://dbpedia.org/resource/V%C3%ADctor_Jara     |  "Víctor Jara"          |
+--------------------------------------------------+-------------------------+
| http://dbpedia.org/resource/Eduardo_Parra_Pizarro| "Eduardo Parra Pizarro" |
+--------------------------------------------------+-------------------------+
| http://dbpedia.org/resource/Alberto_Baeza_Flores |  "Alberto Baeza Flores" |
+--------------------------------------------------+-------------------------+


In most SPARQL endpoints, it is possible to choose the format of the results (XML, JSON,
HTML, etc). For example, the same results in JSON are similar to this::
  
  
  { "head": { "link": [], "vars": ["poet", "poetName"] },
  "results": { "distinct": false, "ordered": true, "bindings": [
    { "poet": { "type": "uri", "value": "http://dbpedia.org/resource/Nicanor_Parra" }	, 
      "poetName": { "type": "literal", "xml:lang": "en", "value": "Nicanor Parra" }
    },
    { "poet": { "type": "uri", "value": "http://dbpedia.org/resource/V%C3%ADctor_Jara" }	, 
      "poetName": { "type": "literal", "xml:lang": "en", "value": "V\u00EDctor Jara" }
    },
    { "poet": { "type": "uri", "value": "http://dbpedia.org/resource/Eduardo_Parra_Pizarro" }	, 
      "poetName": { "type": "literal", "xml:lang": "en", "value": "Eduardo Parra Pizarro" }
    },
    { "poet": { "type": "uri", "value": "http://dbpedia.org/resource/Alberto_Baeza_Flores" }	, 
      "poetName": { "type": "literal", "xml:lang": "en", "value": "Alberto Baeza Flores" }
    }
    ]
   }
  }

  
Using SPARQL results with JavaScript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Thus the results can be used also to fill a webpage. For example, if we want to include a list of Chilean poets in a webpage, we execute the following code (based on jQuery)::

  <html>
  <head>
      <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js" type="text/javascript"></script>
      <script type="text/javascript">
      $(document).ready(function() {
         var q='PREFIX dcterms: <http://purl.org/dc/terms/>\
                PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>\
                PREFIX dbp: <http://dbpedia.org/ontology/>\
                \
                SELECT ?poet ?poetName WHERE{\
                       ?poet dcterms:subject <http://dbpedia.org/resource/Category:Chilean_poets>;\
                              rdfs:label ?poetName .\
                FILTER (LANG(?poetName) = "en")\
                }';
                
         $.ajax({
           dataType: 'jsonp',
           data: {
              query: q,
              format: 'application/sparql-results+json'   // We specify we want the results as a JSON object
              },
           url: 'http://dbpedia.org/sparql',
           success: function(data){
            $(data.results.bindings).each(function(i, item){              
              $("#poetTable").append("<tr><td><a href='"+item.poet.value+"'>"+item.poetName.value+"</a></td>");
             });
           }
         });
      });
    </script>
  </head>
  <body>
    <table id="poetTable">
    <tr><th>Poet name</th></tr>
    </table>
  </body>
  </html>
  

Using SPARQL results with Python
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


`SPARQL Wrapper <http://sparql-wrapper.sourceforge.net/>`_ is a SPARQL client written in Python that can be used to query SPARQL endpoints using Python. The interface is very simple and clean::

  from SPARQLWrapper import SPARQLWrapper, JSON
  
  sparql = SPARQLWrapper("http://dbpedia.org/sparql")
  sparql.setQuery("""
  PREFIX dcterms: <http://purl.org/dc/terms/>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
  SELECT ?poet ?poetName WHERE{
       ?poet dcterms:subject <http://dbpedia.org/resource/Category:Chilean_poets>;
              rdfs:label ?poetName .
       FILTER (LANG(?poetName) = "en")
  }
  """)
  sparql.setReturnFormat(JSON)
  results = sparql.query().convert()
  
  for result in results["results"]["bindings"]:
      print("%s's DBpedia URI is %s" % (result["poetName"]["value"], result["poet"]["value"]))
      
      
References and further reading
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* `RDF Primer <http://www.w3.org/TR/rdf-primer/>`_, is a good introduction to RDF
* `RDF Schema specification <http://www.w3.org/TR/rdf-schema/>`_ provides tools to create new vocabularies
* `A well detailed presentation on Semantic Web and Linked Data <http://www.bbc.co.uk/blogs/radiolabs/s5/linked-data/s5.html>`_
* `RDFa <http://rdfa.info/>`_ is a specification to add RDF embedded in HTML


Tools
-----

* `RDF Validator <http://www.w3.org/RDF/Validator/>`_ check your RDF doesn't have errors
* `SparQled <http://sindicetech.com/sindice-suite/sparqled/>`_ is an interactive SPARQL editor
* `Marbles <http://marbles.sourceforge.net/>`_ is a RDF/Linked Data explorer
* `visualRDF <http://graves.cl/visualRDF/>`_ provides a graphical visualization of RDF graphs

Libraries
---------

* Java
    * `Jena <http://jena.apache.org/>`_
    * `Sesame <http://www.openrdf.org/>`_
* Python
    * `RDFLib <https://github.com/RDFLib/rdflib>`_
    * `SPARQL Wrapper <http://sparql-wrapper.sourceforge.net/>`_
* Ruby
    * `Linked Data for Ruby <http://rdf.rubyforge.org/>`_
* PHP
    * `ARC2 <https://github.com/semsol/arc2/wiki/>`_ parses and serializes RDF, provides a SPARQL endpoint (using MySQL as a backend) and much more
    * `RAP <http://www4.wiwiss.fu-berlin.de/bizer/rdfapi/>`_ an API for RDF
* C
    * `Redland RDF Libraries <http://librdf.org/>`_
* Scala
    * `Scardf <http://code.google.com/p/scardf/>`_
    
