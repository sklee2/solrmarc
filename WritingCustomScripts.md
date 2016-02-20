## Introduction ##

Occasionally you need processing that is not already available in SolrMarc.  This page describes how to write custom indexing scripts in [BeanShell](http://www.beanshell.org).

Custom indexing scripts are a way to extend SolrMarc even if you started from a pre-built binary release, and are unable to compile custom indexing routines, or if you just don't want to write java code.  The scripts are interpreted at run-time, therefore no compiling is necessary.

You might want to check if there is a pre-existing "custom" indexing method (see [CustomIndexingRoutines](CustomIndexingRoutines.md)) that does what you need.

If you want to write in java, see [WritingCustomMethods](WritingCustomMethods.md).

# Writing Custom Indexing Scripts #

If you are unable to accomplish what you want to do using any of the other provided indexing techniques:
  * Field Specifications [IndexProperties](IndexProperties.md),
  * Pattern Matching [IndexProperties](IndexProperties.md),
  * pre-defined Custom Indexing Methods [CustomIndexingRoutines](CustomIndexingRoutines.md)
then there is the option to create your own custom indexing with scripts.

The syntax of the language [BeanShell](http://www.beanshell.org) for creating a custom indexing script is _very_ similar to java.  If you obtain the java source code for a custom indexing routine from someone, and want to implement that routine in a BeanShell script, it usually takes only about 5 minutes to manually convert.

The syntax that you need to use in your index.properties file to reference a custom indexing script is very similar to was described above for referencing a custom indexing routine.  You can reference a custom-created indexing script, by specifying `script` as the second value in the field specification entry as shown in the example below:

> `callnumber = script(callnumber.bsh), getFullCallNumber`

In this case the value in parentheses after _script_ specifies the name the BeanShell script that contains the custom scripted indexing method that you want to use, and the value following the comma is the name of the scripted method to invoke to handle this particular bit of indexing.

## Creating Custom Indexing Scripts ##

Note before you go to the work to create a new custom indexing script, see the above section on [Pre-Defined Custom Indexing Routines](#Pre-Defined_Custom_Indexing_Routines.md), because there may already be a custom routine included in SolrMarc to accomplish what you want.  If you are still unable to accomplish what you want to do using any of the provided indexing techniques, there is the option to create your own custom indexing functions.

Steps to create a custom indexing function:

  1. Create a BeanShell script file in the index\_scripts directory, defining one or more scripted custom indexing methods.
  1. Add `import` statements to the top of the BeanShell script file referencing all of the non-builtin-java classes your scripts reference, such as classes from SolrMarc or from marc4j.
  1. Add a script-wide variable declaration of a variable named indexer: `org.solrmarc.index.SolrIndexer indexer = null;`  Note that the SolrIndexer code will set this value before script methods are called.

For example, if you include the following index specification:

> `callnumber = script(callnumber.bsh), getFullCallNumber`

You will need to define a script named callnumber.bsh in the index\_scripts directory as show below:

```
import org.marc4j.marc.Record;

// define the base level indexer so that its methods can be called from the script.
// note that the SolrIndexer code will set this value before the script methods are called.
org.solrmarc.index.SolrIndexer indexer = null;

/**
 * Extract the call number label from a record
 * @param record
 * @return Call number label
 */
public String getFullCallNumber(Record record) {

    String val = indexer.getFirstFieldVal(record, "099ab:090ab:050ab");

    if (val != null) 
    {
        return val.toUpperCase().replaceAll(" ", "");
    } 
    else 
    {
        return val;
    }
}

```

This example gets the first occurance of field 099, 090 or 050 and extract the a and b subfields from that field.  It then shifts the returned string to all uppercase letters and strips out spaces.

Note that if you want to define more than one scripted custom indexing method, they **can** all be defined in the same BeanShell custom indexing script file, or they can be separated into several different BeanShell files.  Also note that all scripted custom indexing methods must return a `String`, a `List` or a `Set`, and must accept a parameter consisting of the `Record` being operated upon, followed by zero or more `String` parameters that can be used for greater control over the operation of the custom function.

### Example Scripted Custom Indexing Methods ###

**_Create a facet based on the Dewey Decimal numbers found in the record_**

```
import org.marc4j.marc.Record;
import org.solrmarc.tools.Utils;
import org.solrmarc.tools.CallNumUtils;

// define the base level indexer so that its methods can be called from the script.
// note that the SolrIndexer code will set this value before the script methods are called.
org.solrmarc.index.SolrIndexer indexer = null;

/**
 * returns the facet value for dewey hundreds digits, and dewey tens digits
 * @param record
 * @return Set of Strings containing facet value for dewey hundreds digits, and dewey tens digits
 */
Set getDeweyFacet(Record record, String propertiesMapName)
{
    LinkedHashSet resultSet = new LinkedHashSet();
    Set values = indexer.getFieldList(record, "082a");
    String mapName = indexer.loadTranslationMap(propertiesMapName);

    for (String dewey : values)
    {
        if (! CallNumUtils.isValidDewey(dewey))  continue;
        String hundreds = dewey.substring(0, 1) + "00";
        String tens = dewey.substring(0,2) + "0";
        String hundredsMapped = Utils.remap(hundreds, indexer.findMap(mapName), true);
        String tensMapped = Utils.remap(tens, indexer.findMap(mapName), true);
        if (hundredsMapped != null) resultSet.add(hundredsMapped);
        if (tensMapped != null) resultSet.add(tensMapped);
    }

    return resultSet;
}

```

Note that the above routine invokes a translation map before it returns its results (the name of the translation map to use is passed in as a parameter), you can either handle it this way in a scripted custom indexing routine, or return the values unmapped, and include a third parameter on the field specification entry as described above for the other index specifications.  In this case though since both the description of the Dewey decimal number at the hundreds level and at the tens level is desired, the mapping must be handled in this way.   Also note that if this BeanShell scrip was written out to a file named dewey.bsh, the indexing specification to invoke this method would be:

` dewey_facet = script(dewey.bsh), getDeweyFacet(callnumber_map.properties) `