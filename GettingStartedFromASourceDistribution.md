

### About This Guide ###

This getting started guide describes SolrMarc, defines its software dependencies, steps the user through configuring and building the importer program, and gives basic
Currently, SolrMarc is configured to work with:
  * Blacklight (http://projectblacklight.org/) -- see SolrMarc BlacklightHowTo
  * VuFind (http://vufind.org/) -- see SolrMarc VufindHowTo

There are three different ways in which you can obtain/create a working SolrMarc:
  1. Start from a Full Source distribution from the googlecode download page or a full SVN checkout
  1. Start from a Pre-built source distribution from the googlecode download page.
  1. Start from a Pre-built binary distribution from the googlecode download page.

The process you will then take for each of the cases to create a SolrMarc installation which is customized for your needs is similar.  The details of where edits and additions should be made in some cases are different for each of these different methods, but the steps that you have to take are largely the same.

If you do not anticipate a need for writing your own custom indexing functions, the third of the options, the binary distribution of SolrMarc, is a simpler option. The binary distribution is delivered as a single large .zip or .tar file.  When unpacked this distribution will create a directory containing a large SolrMarc.jar file, a number of .properties files, and a couple of sub-directories.

For information on using binary distributions, see SolrMarcBinaryDistribution

To get the binary distribution, go to [this project's downloads page](http://code.google.com/p/solrmarc/downloads/list) and download one of:
  * `SolrMarc_GenericBlacklight_Binary_PC.zip`
  * `SolrMarc_GenericBlacklight_Binary_Unix.tar.gz`
  * `SolrMarc_GenericVuFind_Binary_PC.zip`
  * `SolrMarc_GenericVuFind_Binary_Unix.tar.gz`
  * `SolrMarc_Generic_Binary_PC.zip`
  * `SolrMarc_Generic_Binary_Unix.tar.gz`

The rest of this guide will assume that you have opted for one of the source distributions, or a checkout of source code from SVN, and will guide you through the steps necessary to create a working installation of SolrMarc.

<br />
### Software Dependencies ###

Building SolrMarc from source requires the java development kit (JDK) version 1.5 or newer and ant version 1.7 or newer.

#### Java ####

To check the version of the java development kit (JDK), at the command prompt type

> _javac -version_

To download the proper version of the java development kit, go to http://java.sun.com/javase/downloads/index.jsp

#### Ant ####

To check the ant version, at the command prompt type

> _ant -version_

To download the proper version of ant, go to http://ant.apache.org/

#### Solr ####

SolrMarc includes a runnable copy of jetty and a solr.war file taken from the recent Solr 1.4 release.  These are used internally, by default, for testing, and _can_ be used to deploy and run the solr index, or you can point SolrMarc at an already installed copy of Solr.

If you decide you want to download and install a separate installation of Solr, go to http://lucene.apache.org/solr/

<br />

### What Should my Marc Data Look Like ###
normal marc data;  data that follows the marc bibliographic record standard.  It can be in marcxml or marc21 (these are the file structures, not the encoding).  The SolrMarc utility programs all assume that the mac file will have a suffix of .mrc or .marc for marc21 records, and a suffix of .xml for marcxml records.  If this expectation is a major obstacle, you can also pipe your data to the utility program over standard input

> _cat marcfile.withdifferentsuffix | indexfile_

### Getting Started from a Full Source distribution, or a SVN checkout ###

First either download the software via a subversion checkout:

`svn checkout http://solrmarc.googlecode.com/svn/branches/solrmarc-2.1 solrmarc`

or by downloading the distribution (SolrMarc\_Distribution.tar.gz) from the downloads page:
http://code.google.com/p/solrmarc/downloads/list

Next, switch into the solrmarc directory you just created, and at the command prompt, run

`ant init`

The initialization process will ask which of the examples should be used to base your local site on:
GenericBlacklight,  GenericVuFind,  stanfordBlacklight,  UvaBlacklight,   or  none.

If you select one of the provided examples, all of the necessary files will be copied to the local\_build directory, and you will then be ready to build a working copy of SolrMarc.   You can skip forward to the section on [Customizing SolrMarc build from a Full Source distribution, or a SVN checkout](http://code.google.com/p/solrmarc/wiki/GettingStartedFromASourceDistribution#Customizing_build_from_a_Full_Source_distribution,_or_a_SVN_chec)
#### Installation Questions ####

If you specify ‘none’ then you will have to answer a few more questions before you will be able to build SolrMarc, and even then you will have to extensively modify the index.properties files before your SolrMarc will be able to produce a useful solr index.   (Since the default index specification file is simply a placeholder, which specifies the record id and the title as the only fields that are added to the solr index.)

The questions it will ask are as follows:  (Note that for all of them you can accept the default responses by hitting return)

##### Enter prefix for site-specific config and index properties file names: #####
> (ie. for  myprefix\_config.properties  and   myprefix\_index.properties  enter  myprefix)
> default response: myprefix

##### Enter java heap size to use in generated scripts for running site-specific SolrMarc Indexer: #####
> default response: -Xmx256m

> While 256MB is sufficient to load the demo, a real-world indexing job requires much more. The developer uses 1024MB on his desktop and 2048MB on the server where the indexing job will include millions of records.

##### Enter encoding of MARC records: #####
> MARC8 - Longstanding standard encoding scheme used by U.S. libraries
> UNIMARC - Encoding scheme used in many places in Europe
> UTF8 - Unicode character encoding scheme, used in some newer systems
> BESTGUESS - You want the program to try to determine the encoding
> default response: MARC8

> Note that if you have a pretty good guess as to the encoding that is used in the records you will be processing, SolrMarc can run faster.  If some records are in MARC8 while others will be in UTF8, SolrMarc will be able to handle them, since SolrMarc translates all records to UTF-8 and for the records that are already in UTF-8, the translation step will be skipped.   Also note that the BESTGUESS option will try to make a determination as to the character encoding used on a record-by-record basis, however since many records do not have large numbers of "special characters" (ie. ones outside the basic 7-bit ascii range) the information available for deciding the correct encoding can be minimal.)

##### Do you want to use the 'builtin' solr configuration, jetty configuration and solr.war file or do you want to use a 'custom' one you have already installed. #####
> Note: You can start from the builtin version and modify to suit your needs.
> Or you can change the one used later by modifying the build.properties file.
> default response: builtin

> If you answer "custom", you will be asked for the URL of where your Solr server will be running, the full path location of your solr home directory, and the full path of the solr.war file that the Solr server uses for running solr.

> Note one option that you can use if the Solr server will be running on a different machine than where you intend to run SolrMarc, and you want to remotely access that running Solr server, enter the URL of the Solr server for the first response, and then enter "REMOTE" for the question: "Enter full path of Solr home directory"  and enter an empty string for the question: "Enter full path of where solr.war file is located (include solr.war at end)"

These custom settings are now written out to a build.properties file, in the local\_build directory, and they will be used when you run the "ant dist" target to correctly fill in values in the configuration files.

### Customizing SolrMarc build from a Full Source distribution, or a SVN checkout ###
### Customizing SolrMarc based on a pre-built Source distribution. ###

If you look in the "local\_build" directory after you complete the "ant init" step, you will see a number of directories, a couple of properties files, and a build.xml file.  These files and directories are explained below.   At this point you will be at virtually the same point that you would be if you had merely obtained a pre-built source distribution.  So both of these section are described together here.

At this point you can modify the files in the local\_build directory to customize your installation.

- You can modify the xxxx\_config.properties file that specifies how the overall SolrMarc program will run, how MARC files should be processed as they are read, what file (or files) defines the indexing specification to use, whether Marc processing errors should be added to the Solr index.

- You can modify the xxxx\_index.properties files to change what fields are added to the index and where and how the data for those fields is extracted. (Note that any fields added to this file _must_ match either a field declaration in the schema.xml file that is a part of the solr configuration or match a dynamic field declaration in that file.)

- You can add custom indexing java routines in the src directory, and reference them in the xxxx\_index.properties file.

- You can add custom java-like indexing scripts in the index\_scripts directory,  and reference them in the xxxx\_index.properties file.

- You can add or modify translation maps that map values found in the MARC records to more-readable values.

- You can change entries found in the build.properties file that are copied into the xxxx\_config.properties file or into the bash script or batch files from the script\_templates directory.

- You can modify the solr configuration, which can be done by editing files or adding new files in the solrConf directory, or by going directly to the test/solr directory and adding or editing files there.

- You can add tests to the test/data/indextest.txt file so that you can help ensure that as you make any of the above changes you don’t accidently break some other aspect of your indexing configuration.

#### Building Runnable SolrMarc after Changing Configuration ####

After you have made the desired changes, you should run "ant dist" which will copy all the necessary files to the distribution directory.  You can change to that directory, or merely add dist/bin to your executable path, and run the indexing utilities (described in a later section) from any command prompt.
Note: you should refrain from editing files in the distribution directory directly when you are working from a source distribution, since those changes could be overwritten the next time you run "ant dist".

#### Files/directories in local\_build directory, or Prebuilt Source Distribution ####
<table border='1'>
<tr valign='top'><td>build.xml</td><td>Generated Ant build file to compile custom source code, bundle classes and additional data into jar, and generate working distribution directory by copying and/or modifying files found in the local_build directory.</td></tr>
<tr valign='top'><td>build.properties</td><td>Used by the Ant build process to define properties used by the build process.  Some of the values defined in this file are inserted into the xxxxx_config.properties file as it is copied into the distribution directory.  Other values are inserted into the bash scripts or batch files that are copied from the script_templates directory to the dist/bin directory.</td></tr>
<tr valign='top'><td>xxxxx_config.properties</td><td>Specifies values used to configure a particular run of SolrMarc.  Note that some of the values in this file can be expressed in terms of named properties (such as  solr.path = ${solr.path} ) and the value of that property will be substituted in (usually) as the ant build process copies this file to the distribution directory.</td></tr>
<tr valign='top'><td>xxxxx_index.properties</td><td>Specifies all of the indexing rules for mapping fields and subfields in the marc records that will be processed to the solr index fields that will be added to the solr index by SolrMarc. Note that any fields added to this file <i>must</i> match either a field declaration in the schema.xml file that is a part of the solr configuration or match a dynamic field declaration in that file.  The schema that will be used by default can be found in the solrConf directory listed below.</td></tr>
<tr valign='top'><td>bin</td><td>Directory where class files from custom indexing methods are placed. Also where the .jar file containing all of those custom class files are placed.<br>
<tr valign='top'><td>buildtools</td><td>Contains files that add functionality used by the ant build process, or are used by the smoketest target in the ant build process.</td></tr>
<tr valign='top'><td>extra_data</td><td>Directory to contain resources that are to be placed in the custom .jar file, these might be text files that are used by custom indexing methods. Most sites will not need any of these, and therefore can safely ignore this directory.<br>
<tr valign='top'><td>index_scripts</td><td>Directory to contain custom java-like beanshell script files to handle custom indexing tasks.  This is an alternative to creating custom java indexing routines, it is slightly easier to create and modify these scripts, but slightly slower to run them as compared to compiled java versions implementing the same code. </td></tr>
<tr valign='top'><td>lib</td><td>Contains the already-compiled SolrMarc.jar file that is virtually identical for all users of SolrMarc.   The only difference being that the name of the "default configuration" to use will be recorded in the Manifest within the Jar file as the file is copied to the distribution directory by the "ant dist" target.</td></tr>
<tr valign='top'><td>script_templates</td><td>Contains a number of bash scripts and batch files that invoke SolrMarc in a number of useful ways.  The Ant build process will copy the platform-appropriate versions of these files to the bin subdirectory of the distribution directory.  In the process of copying them the Ant build process will take some values defined in the build.properties file, and substitute them into this script files.</td></tr>
<tr valign='top'><td>solrConf</td><td>Contains modifications to the configuration of solr. Usually this would consist merely of a copy of solrconfig.xml and schema.xml which would then be placed in dist/jetty/solr/conf directory.  However it can also include a different version of solr.war or even a multicore solr.xml file and subdirectories containing the configurations for each of those multiple cores.</td></tr>
<tr><td>src</td><td>Contains java source code defining custom indexing routines.  Note that the main custom class must extend the SolrMarc class org.solrmarc.index.SolrIndexer  .  The Ant build process should detect the fact that a custom indexing class is included here, and correctly set the ant properties  custom.jar.name.jar   and  custom.indexer.class  that will be used when copying the xxxxx_config.properties file to the distribution directory.</td></tr>
<tr valign='top'><td>translation_maps</td><td>Contains properties files defining translations maps that are used and referenced by indexing specification entries in the xxxxx_index.properties file, or by custom indexing routines, or custom indexing scripts.  These translation maps usually map short mnemonic strings found in MARC records to longer strings that are more meaningful to the user.  ( For instance mapping the three letter language value found in a MARC record to the full value of that language: eng=English<br>
<tr valign='top'><td>test</td><td>Contains several subdirectories associated with testing a SolrMarc installation.</td></tr>
<tr><td>test/bin</td><td>Contains compiled versions of site specific junit testing code, and a .jar file that is built contain all such compiled junit testing code.  Many sites will not have any such specialized tests.</td></tr>
<tr valign='top'><td>test/data</td><td>Contains files used as data by various tests that are performed by the "test" target in the Ant build process.  Mostly this consists of sample MARC  records that are indexed by one or another of the ant tests. <br> One notable file in this directory is indextest.txt in which each line specifies a specific test that ought to be performed.  A sample line appears below:<br><br>blacklight_config.properties, u11.mrc, barcode_facet, 11-1001<br><br>This says use the configuration: blacklight_config.properties, and run the indexer on the file u11.mrc,  and then in the index field named "barcode_facet" produced by the indexing process, the value ought to be the string "11-1001"<br>
<br>
Unknown end tag for </td><br>
<br>
<br>
<br>
Unknown end tag for </tr><br>
<br>
<br>
<tr valign='top'><td>test/jetty</td><td>Contains an installation of jetty that for the most part will be copied directly to the distribution directory</td></tr>
<tr valign='top'><td>test/solr</td><td>Contains a solr configuration that will be used as the basis for what is copied to the dist/jetty directory.  Files that are in the solrConf directory (described above) will be used to override the default versions of them that are found within this directory.</td></tr>
<tr valign='top'><td>test/src</td><td>Contains the source code for site specific junit testing code.  Many sites will not have any such specialized tests and will rely on the higher level ant test targets: "smoketest" and "indextest"</td></tr>
<br>
<br>
Unknown end tag for </table><br>
<br>
