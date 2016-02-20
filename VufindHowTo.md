# Important Note #

Unless you have a very old copy of VuFind (earlier than 1.0RC1), chances are that some version of SolrMarc is already bundled with your installation.  Please see the <a href='http://vufind.org/wiki/importing_records#marc_records'>VuFind documentation</a> for all of the necessary information on using the bundled copy of SolrMarc.

If you want to customize or upgrade your copy of SolrMarc, some of the following documentation may be helpful.  However, some details may be out of date.  For most users, the simplest course of action is to upgrade to the latest version of VuFind and use the provided version of SolrMarc.

# Introduction #

SolrMarc makes creating your index for VuFind extremely simple. First, you need to get the latest version of either the Subversion repository, or latest binary. If you're not comfortable with building source code, don't worry, the binary will work on all platforms with Java 1.5 or higher.

To get the Source distribution you can either do a subversion checkout as described on the page http://code.google.com/p/solrmarc/source/checkout or download the souce code distribution named SolrMarc\_Distribution.tar.gz from the downloads page http://code.google.com/p/solrmarc/downloads/list.

To get the binary distribution, download either the  Binary\_Generic\_VuFind\_SolrMarc\_Unix.tar.gz  or  Binary\_Generic\_VuFind\_SolrMarc\_PC.tar.gz from the downloads page http://code.google.com/p/solrmarc/downloads/list.  The binary distribution consists of a single large jar file containing all of the code, libraries and data files to configure and run Solrmarc.

The only difference between the two binary distributions is that the Unix version contains a number of bash shell scripts for running the SolrMarc indexer or for running the the other utility programs associated with running  SolrMarc, whereas the PC version of the binary distribution contain batch files to perform the same tasks.

If you choose to use a Source distribution, the ant build process walks you through the process, asking questions about certain item that need to be configured or customized for your site.  See the wiki page http://code.google.com/p/solrmarc/wiki/GettingStarted for more information about the process of building SolrMarc from source.


# Using the Binary Distribution #
Solrmarc uses a series of Java properties files for its configuration, these are stored inside the single large jar file that is included in the binary distribution.   Some of the values in these properties files need to be set before you will be able to run SolrMarc to produce an index for your VuFind installation.  The main configuration file is named vufind\_config.properties, an example of it is shown below:

```
 # Path to your solr instance
 solr.path = /usr/local/vufind/solr
 solr.core.name = biblio
 solr.indexer = VufindIndexer
 solr.indexer.properties = vufind.properties

 #optional URL of running solr search engine to cause updates to be recognized.
 solr.hosturl = http://localhost:8983/solr/update

 marc.to_utf_8 = true
 marc.permissive = true
 marc.default_encoding = MARC8
 marc.include_errors = false

```

If you unpack the binary distribution into a directory named import inside the VuFind distribution, all you need to do is run two shell scripts to configure the SolrMarc indexer.

First run:
> import/setsolrwar  ./solr/jetty/webapps/solr.war
then run:
> import/setsolrhome ./solr

You will then be ready index MARC record into the solr index that will be used by VuFind via either of the following two commands:

import/import /path/to/marcrecords.mrc
> or
import/indexfile /path/to/marcrecords.mrc

The first of these will merely print "Now Importing /path/to/marcrecords.mrc ..." and quietly import all of the record in that file.   The second will be somewhat more verbose as it processes records, displaying informational messages and warnings while it is running.