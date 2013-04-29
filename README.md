IXA-EHU-DBpedia-spotligth-EN
============================

Procedure to create a local DBpedia Spotlight web service in English within OpeNER

The general procedure has been described [here](https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Internationalization).

**Summary:**

1. Environment preparation

2. Downloading and setting the required data

3. Setting up the config and server files

4. Building indexes

5. Preparing and running the server

6. Testing the system

7. Setting-up and running the web-based interface
 
Requirements

Java 1.6+ 
Scala 2.9+
Maven (see note on versions below)
RAM of appropriate size for the spotter lexicon you need

Whereas most of DBpedia Spotlight works with Java 1.6, the submodule index requires Java 1.7. => It has been necessary to use 1.7 in the whole process

### 1. Environment preparation

The installation has been performed on a 4 x Xeon E7-4830 8 nucleos 2130MHz , 64 GB RAM

    git clone https://github.com/dbpedia-spotlight/dbpedia-spotlight.git
    cd dbpedia-spotlight/

Edit the file `./dbpedia-spotlight/pom.xml` and comment out the current OpenNLP 1.5.2 repository

    <!-- <repository>
             <id>opennlp.sf.net</id>
             <url>http://opennlp.sourceforge.net/maven2/</url>
         </repository> -->

Then edit the file `./dbpedia-spotlight/core/pom.xml`, comment out the OpenNLP 1.5.2. dependencies:

    <!-- <dependency>
            <groupId>opennlp</groupId>
            <artifactId>maxent</artifactId>
            <version>3.0.0</version>
         </dependency>

         <dependency>
            <groupId>org.apache.opennlp</groupId>
            <artifactId>opennlp-tools</artifactId>
            <version>1.5.1-incubating</version>
         </dependency> -->

and add the OpenNLP 1.5.3 dependencies:

    <dependency>
     <groupId>org.apache.opennlp</groupId>
     <artifactId>opennlp-tools</artifactId>
     <version>1.5.3</version>
    </dependency>



Prepare the environment by executing from `./dbpedia-spotlight/`

    mvn install

In case mvn is not installed, type `sudo apt-get install maven` (version 3). Also, remember to install Java JRE or SDK.

### 2. Downloading and setting the required data
Rename the file `./dbpedia-spotlight/bin/download.sh` into `./dbpedia-spotlight/bin/download-en.sh`.

Edit the file `download-en.sh` by changing the following lines:
    
    export lang_i18n=en
    export language=english
    export dbpedia_workspace=../data/spotlight
    export dbpedia_version=3.8

Where `../data/spotlight` is the place where you want to install the DBpedia Spotlight. Download the complete version of [`download-en.sh`] (https://github.com/ialdabe/IXA-EHU-DBpedia-spotlight-EN/master/bin/download-en.sh).

Create the DBpedia workspace directory:

    mkdir -p /data/spotlight

Execute the `./dbpedia-spotlight/bin/download-en.sh`.

Get a stopword file from [here](http://svn.tartarus.org/snowball/trunk/website/algorithms/english):

    wget http://svn.tartarus.org/*checkout*/snowball/trunk/website/algorithms/english/stop.txt?revision=431 --output-file=stopwords.en.list

and copy `stopwords.en.list` to `../data/spotlight/dbpedia_data/data`:

    cp stopwords.en.list ../data/spotlight/dbpedia_data/data

Create a new file called `blackListedURIPattens.en.list` to put instructions (one regex pattern per line) that will guide DBpedia Spotlight to remove pages and articles that should not be considered valid resources for annotation.

    ^List_of_
    .+([Dd]isambiguation)$
    ^[0-9]+$

Then copy the file to `../data/spotlight/dbpedia_data/data`:

    cp blacklistedURIPatterns.en.list ../data/spotlight/dbpedia_data/data

Be careful with the encoding of the generated files. It should be UTF-8

The latest Wikipedia version does not work. It has been used the wikipedia from March 7, 2012. Downloaded from: http://dumps.wikimedia.org/enwiki/20120307/ . It has been renamed to enwiki-lates-pages-articles.xml


### 3. Setting up the config files

Consider now [`./dbpedia-spotlight/conf/indexing.properties`]() and set it as the example in this repository.

Lucene information has been taken from [here](http://lucene.apache.org/core/3_6_2/index.html).

These are the variables changed: (From the Italian version; In the English version all are relative paths

    org.dbpedia.spotlight.data.wikipediaDump = ../data/spotlight/dbpedia_data/original/wikipedia/en/enwiki-latest-pages-articles.xml 

    org.dbpedia.spotlight.index.dir =../data/spotlight/dbpedia_data/data/output/index

    org.dbpedia.spotlight.data.labels =../data/spotlight/dbpedia_data/original/dbpedia/en/labels_en.nt.bz2
    org.dbpedia.spotlight.data.redirects = ../data/spotlight/dbpedia_data/original/dbpedia/en/redirects_en.nt.bz2
    org.dbpedia.spotlight.data.disambiguations = ../data/spotlight/dbpedia_data/original/dbpedia/en/disambiguations_en.nt.bz2
    org.dbpedia.spotlight.data.instanceTypes = ../data/spotlight/dbpedia_data/original/dbpedia/en/instance_types_en.nt.bz2

    org.dbpedia.spotlight.data.conceptURIs = ../data/spotlight/dbpedia_data/data/output/conceptURIs.list
    org.dbpedia.spotlight.data.redirectsTC = ../data/spotlight/dbpedia_data/data/output/redirects_tc.tsv
    org.dbpedia.spotlight.data.surfaceForms = ../data/spotlight/dbpedia_data/data/output/surfaceForms.tsv

    org.dbpedia.spotlight.language = English
    org.dbpedia.spotlight.language_i18n_code = en
    org.dbpedia.spotlight.lucene.analyzer = org.apache.lucene.analysis.en.EnglishAnalyzer

    org.dbpedia.spotlight.default_namespace = http://dbpedia.org/resource/
    org.dbpedia.spotlight.default_ontology= http://dbpedia.org/ontology/

    org.dbpedia.spotlight.data.stopWords.english = ../data/spotlight/dbpedia_data/data/stopwords.en.list
    org.dbpedia.spotlight.data.badURIs.english= ../data/spotlight/dbpedia_data/data/blacklistedURIPatterns.en.list

    org.dbpedia.spotlight.yahoo.language = en
    org.dbpedia.spotlight.yahoo.region = en

Consider then [`./dbpedia-spotlight/conf/server.properties`](), set the variables to English and select the Spotter (now only SpotXmlParser)

These are the variables changed: (From the Italian version; In the English version all variables have relative paths)

    org.dbpedia.spotlight.web.rest.uri = http://localhost:2020/rest
    org.dbpedia.spotlight.default_namespace = http://dbpedia.org/resource/
    org.dbpedia.spotlight.default_ontology= http://dbpedia.org/ontology/
    
    org.dbpedia.spotlight.language = Englsih
    org.dbpedia.spotlight.language_i18n_code = en
    
    org.dbpedia.spotlight.data.stopWords.english = ../data/spotlight/dbpedia_data/data/stopwords.it.list
    org.dbpedia.spotlight.spot.spotters = SpotXmlParser
    org.dbpedia.spotlight.spot.dictionary = ../data/spotlight/dbpedia_data/data/spotter.dict
    
    org.dbpedia.spotlight.spot.cooccurrence.database.connector = jdbc:hsqldb:file:../data/spotlight/dbpedia_data/data/spotsel/ukwac_candidate;shutdown=true&readonly=true
    org.dbpedia.spotlight.spot.cooccurrence.classifier.unigram = ../data/spotlight/dbpedia_data/data/spotsel/ukwac_unigram.model
    org.dbpedia.spotlight.spot.cooccurrence.classifier.ngram = ../data/spotlight/dbpedia_data/data/spotsel/ukwac_ngram.model
pedia.spotlight.core.database.connector = jdbc:hsqldb:file:/data/spotlight/database/spotlight-db;shutdown=true&readonly=true
    org.dbpedia.spotlight.candidateMap.dir = ../data/spotlight/dbpedia_data/data/output/index-withSF-withTypes-compressed
    org.dbpedia.spotlight.index.dir =/data/spotlight/dbpedia_data/data/output/index-withSF-withTypes-compressed
    org.dbpedia.spotlight.lucene.analyzer = org.apache.lucene.analysis.en.EnglishAnalyzer
    org.dbpedia.spotlight.sparql.endpoint = http://dbpedia.org/sparql
    org.dbpedia.spotlight.sparql.graph = http://dbpedia.org

The server properties file does not need all the variables. In order to obtain a faster response time, the [`./dbpedia-spotlight/conf/server_jar.properties`]() is provided. It contains just the necessary variables to run the server. Thus, the necessary variable are the ones related with SPOTTING, DISAMBIGUATION, and LINKING

The SpotXmlParser option considers as input a text and a list of already detected named entities. This is the option to be used.

For more info:https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Spotting

If the NER has to be done again, the server.properties file needs to be used and change the org.dbpedia.spotlight.spot.spotters variable to "NESpotter WikiMarkupSpotter" 

In this case, it is necessary to compile/install the spotlight with the file versions provided with this distribution:
core/src/main/java/org/dbpedia/spotlight/spot/NESpotter.java
core/src/main/java/org/dbpedia/spotlight/model/SpotterConfiguration.java
core/src/main/java/org/dbpedia/spotlight/spot/OpenNLPUtil.java

This change in the code allows the use of one single model (the one from OpeNER) for NER. This new option is represented by means of:
org.dbepdia.spotlight.spot.opennlp.ner = http://dbpedia.org/ontology in the server.properties file


### 4. Building indexes

Copy `./dbpedia-spotlight/bin/index.sh` into `./dbpedia-spotlight/bin/index-en.sh`. Edit the new file and set the following parameter:

    export DBPEDIA_WORKSPACE=../data/spotlight/dbpedia_data

and set the variable according to the available memory in your system (e.g. 2GB):

    JAVA_XMX=2g - For the English version the original 14g value has been kept

Take a copy of the file available in this repository [here] (https://github.com/ialdabe/IXA-EHU-DBpedia-spotlight-EN/master/bin/index-en.sh) and change the above mentioned parameters.

When running the indexing of the English version, the system sometimes goes into a infinite loop. This is due to the getEndChainUri function in the ExtractCandidateMap.scala. If this happens, change the source file from the one contained in 
index/src/main/scala/org/dbpedia/spotlight/util/ExtractCandidateMap.scala

The function has a new variable to control the number of loops that has been done. It has been set at 1000. This is useful when the given uri contains '. Altough being the values of s and k equal, if there is a ', the system doesn't recognise it as equal and a new recursive loop is done.

Then unzip the file `/data/spotlight/dbpedia_data/original/wikipedia/it/itwiki-latest-pages-articles.xml.bz2` once for all, otherwise specify it in the `index-en.sh` and it will unzip it everytime you run the script.

If the one retrieved from March 7, 2012 has not this name, unzip it and change the name. 

Execute `./bin/index-en.sh`

Copy the file

    ../data/spotlight/dbpedia_data/data/output/surfaceForms.tsv.spotterDictionary

to

    ../data/spotlight/dbpedia_data/data/spotter.dict

This file is not necessary anymore. The spotting is not done with the spotlight. 

<!-- *Note*: If you need to re-run this script, be sure to delete everything *but* `index` folder in `../data/spotlight/dbpedia_data/data/output/` -->

### 5. Preparing and running the server

Before running the server, some necessary files are needed. Copy the `similarity-thresholds.txt` file just generated in `../data/spotlight/dbpedia_data/data/output/index` to `/data/spotlight/dbpedia_data/data/output/index-withSF-withTypes-compressed/`

In the file `./bin/server.sh`  change the following parameters, according to the system memory (here 2G is specified):

    JAVA_OPTS="-Xmx2G  -XX:+UseConcMarkSweepGC -XX:MaxPermSize=2G"
    MAVEN_OPTS="-Xmx2G -XX:+UseConcMarkSweepGC -XX:MaxPermSize=2G"
    SCALA_OPTS="-Xmx2G -XX:+UseConcMarkSweepGC -XX:MaxPermSize=2G"
 
Execute the server:

    ./bin/server.sh

### 6. Testing the system

It is possible to use the DBpedia Spotlight Web Service based on the WADL (Web Application Description Language), described at [http://spotlight.dbpedia.org/rest/application.wadl]([http://spotlight.dbpedia.org/rest/application.wadl). Here the system has been configured only to spot Named Entities. Therefore it is advisible to rely on the following example:

**Example 1: Simple request**

* text= "Enrico Fermi fisico."
* confidence = 0.2; support=20

    curl http://localhost:2020/rest/disambiguate?spotter=SpotXmlParser + 
					"&confidence=" + CONFIDENCE
					+ "&support=" + SUPPORT
		                        + "&text" + text

The new disambiguation type requires other type of input:
<annotation text="Brazilian oil giant Petrobras and U.S. oilfield service company Halliburton have signed a technological cooperation agreement, Petrobras announced Monday. The two companies agreed on three projects: studies on contamination of fluids in oil wells, laboratory simulation of well production, and research on solidification of salt and carbon dioxide formations, said Petrobras. Twelve other projects are still under negotiation.">
<surfaceForm name="oil" offset="10"/>
<surfaceForm name="company" offset="56"/>
<surfaceForm name="Halliburton" offset="64"/>
<surfaceForm name="oil" offset="237"/>
<surfaceForm name="other" offset="383"/>
</annotation>

### 7. Setting-up and running the web-based interface

THIS OPTION HAS NOT BEEN DONE FOR THE ENGLISH VERSION

Copy the directory `demo` to your web server root (e.g. `/var/www`) and then in the following files:

    index.html

change the parameter `server`:

    var server = 'http://your-server-ip/rest'

according to your server name or IP.

Then, in the file `dbpedia-spotlight-0.3.js`  change the following parameter:

    var li = "<li class='"+className+" "+className+"-" + i + "'><a href='http://it.dbpedia.org/resource/" + r["@uri"] + "' about='" + r["@uri"] + "'>" + r["@label"] + "</a>";

Finally, load the page http://your-server-ip/demo and test the system.

### 8. Required  information to run the server
The index step generates lots of information that is not necessary to run the server. Once all the indexing steps are done, it is possible to keep just the following information:
- pos-en-general-brown.HiddenMarkovModel (it is not used, but it is necessary because of the configuration)
- index-withSF-withTypes-compressed: this directory contains the necessary information to disambiguate the entities
It is a good idea to create a jar file with all the dependencies once the server is ready to be used. To do so, go to the dist directory and type:
mvn package

This will create:
- dbpedia-spotlight-0.6.deb
- target/dbpedia-spotlight-0.6-jar-with-dependencies.jar

The debian package does not contain the necessary information to install the modified version of the spotlight. But, the jar file contains all the dependencies the system needs to run the spotlight. As the paths used in the server are relative paths, it is necessary to move the jar to the bin directory. 

To run the server with this jar just:
java -cp dbpedia-spotlight-0.6.jar-with-dependecies.jar org.dbpedia.spotlight.web.rest.Server server_jar.properties
