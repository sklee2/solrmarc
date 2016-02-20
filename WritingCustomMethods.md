## Introduction ##

Occasionally you need processing that is not already available in SolrMarc.  This page describes how to write your own custom indexing methods in java. For instructions on how to write custom indexing [BeanShell](http://www.beanshell.org) _scripts_ that don't require compiling, see [WritingCustomScripts](WritingCustomScripts.md).

First, you should check if there is a pre-existing "custom" indexing method (see [CustomIndexingRoutines](CustomIndexingRoutines.md)) that can do what you need.

# Writing Custom Indexing Methods #

If you are unable to accomplish what you want to do using any of the other provided indexing techniques:
  * Field Specifications [IndexProperties](IndexProperties.md),
  * Pattern Matching [IndexProperties](IndexProperties.md),
  * pre-defined Custom Indexing Methods [CustomIndexingRoutines](CustomIndexingRoutines.md)
then there is the option to create your own custom indexing functions.

Steps to create a custom indexing function:

  1. Create a java source file defining a class, specifying that the new class extends the class SolrIndexer.
  1. Define a Constructor for that new class that accepts a String naming a properties file, and an array of strings specifying the paths where the property files can be found.
  1. Define a function to correspond to every as-of-yet-undefined custom indexing function that appears in your index specification file.

For example, if you include the following index specification:

> `recordings_and_scores_facet = custom, getRecordingAndScore`

You will need to define a custom function in your java source file named `getRecordingAndScore` as show below:

```
import org.solrmarc.index.SolrIndexer;

public class BlacklightIndexer extends SolrIndexer
{

public BlacklightIndexer(String propertiesMapFile, String propertyPaths[])
    {
        super(propertiesMapFile, propertyPaths);
    }

    public Set<String> getRecordingAndScore(Record record)
    {
        Set<String> result = new LinkedHashSet<String>();
        String leader = record.getLeader().toString();
        String leaderChar = leader.substring(6, 7).toUpperCase();
                
        if(leaderChar.equals("C") || leaderChar.equals("D"))
        {
            result.add("Scores");
            result.add("Recordings and/or Scores");
        }
        
        if(leaderChar.equals("J"))
        {
            result.add("Recordings");
            result.add("Recordings and/or Scores");
        }
        
        return result;
    }

//  other custom indexing functions go here

}
```

This example looks in position 6 of the leader of the MARC record, and if the value there is a _C_ or a _D_ it will return a set containing two values to be added to the index entry named `recordings_and_scores_facet` in the Solr index record, and if the value in that character is _J_ it will return a set containing two other values.  If any other value occurs in that location, this routine will return an empty set, and no values will be added for this index entry in the Solr index record.  Because this routine must return two items based on a single input item, there is no way that it could be handled via the standard index specification syntax.

The next step to make sure that your main properties file lists that name of your new custom class as the value for the `solr.indexer` property, as shown here:

> `solr.indexer = BlacklightIndexer`

Note that if you need to define more than one custom indexing function, they **must** all be defined in the same java file.   Also note that all custom indexing functions must be defined as **public**, must return either a `String` or a `Set<String>`, and must accept a parameter consisting of the `Record` being operated upon, followed by zero or more String parameters that can be used for greater control over the operation of the custom function.

### Example Custom Indexing Routines ###

**_Create an index entry based on the value in field X, but only if a certain value appears in field Y._**

```
    public Set<String> getRecordingFormat(Record record)
    {
        String mapName = loadTranslationMap(null, "recording_format_map.properties");
        Set<String> result = new LinkedHashSet<String>();
        String leader = record.getLeader().toString();
        String leaderChar = leader.substring(6, 7).toUpperCase();
        Set<String> titleH = new LinkedHashSet<String>();
        addSubfieldDataToSet(record, titleH, "245", "h");       
                
        if(leaderChar.equals("J") || leaderChar.equals("I") || 
                (setItemContains(titleH, "videorecording")))
        {
            Set<String> form = new LinkedHashSet<String>();
            addSubfieldDataToSet(record, form, "999", "t");
            Set<String> labels = Utils.remap(form, findMap(mapName"));
            return(labels);
        }
        return(result);
    }
```

Note that the above routine invokes a translation map before it returns its results, you can either handle it this way in a custom indexing routine, or return the values unmapped, and include a third parameter on the field specification entry as described above for the other index specifications.

**_Create an index entry that consists of the first characters from a given field, but only as many characters as are letters._**

```
      public String getCallNumberPrefix(Record record)
    {
        String val = getFirstFieldVal(record, "999a:090a:050a");
        if (val == null || val.length() == 0) return(null);
        String vals[] = val.split("[^A-Za-z]+", 2);
        if (vals.length == 0 || vals[0] == null || vals[0].length() == 0)
     return(null);
        return(vals[0]);
    }
```

This custom function returns the beginning portion of the call number (extracted from either 999a or 090a or 050a  field) but only up until the first character that is not a letter.

**_Create an index entry based on a portion of a given field, but only if the remaining portion of that field contains a certain value._**

```
    public String getOclcNum(Record record)
    {
        Set<String> set = getFieldList(record, "035a");
        if (set.isEmpty())  return(null);
        Iterator iter = set.iterator();
        while (iter.hasNext())
        {
            String value = (String)iter.next();
            if (value.contains("(OCoLC)"))  
            {
                value = value.replaceAll("\\(OCoLC\\)", "");
                return(value);
            }
        }
        return null;
    }
```

This routine gathers all 035a fields in the MARC record, and scans through them looking for one labeled with the string _(OCoLC)_ indicating that it is an OCLC number, the routine strips out the string _(OCoLC)_ and returns the remaining string which is the OCLC number of the item in the MARC record. If no 035a field containing that label is found, no OCLC number is added to the Solr index record for this item.