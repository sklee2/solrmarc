## Introduction ##

It is sometimes necessary to do custom processing of values during indexing.  This page lists some of the pre-defined indexing routines included in SolrMarc to help solve common problems along with a short description of what each of them does.  For more details on how to call these functions, see the [IndexProperties](IndexProperties.md) page.

If the type of processing you need is not available, you may write your own custom indexing method ([WritingCustomMethods](WritingCustomMethods.md)) or a custom indexing script ([WritingCustomScripts](WritingCustomScripts.md)).

This page may get out of date - the most complete information can be found in the javadocs (once we have them all written) or in the source code.

# Pre-Defined Custom Indexing Methods #

**getAllAlphaExcept(fieldSpec)**:
> For _each_ occurrence of a MARC field in the !fieldSpec list, concatenate the alphabetic subfield contents _except_ the ones specified, using a space separator.

> `geographic_search = custom, getAllAlphaExcept(651vxyz:691vxyz)`

> For each MARC 651 and 691 field, concatenate all the subfields from a to u, in order of occurrence, and add a field to the Solr document named geographic\_search with the concatenated value.
> Currently, the !fieldSpec may be one or more letters: it will not recognize regular expressions and it will include all alpha if there are no subfields specified.

**getAllAlphaSubfields(fieldSpec)**:
> For _each_ occurrence of a MARC field in the !fieldSpec list, concatenate all alphabetic subfield contents, using a space separator.

> `title_full_display = custom, getAllAlphaSubfields(245)`

> For each MARC 245 field, concatenate all alphabetic subfield values, then add a field named full\_title\_display to the Solr document with the concatenated value.

**getAllAlphaSubfields(fieldSpec, first|join|all)**:
> As above, but using "first" "join" or "all" to indicate handling of multiple occurrences of field values

> `title_uniform_display = custom, getAllAlphaSubfields(130:240, first)`

> If the MARC record has a 130 field, concatenate all alphabetic subfield values, then add a field named title\_uniform\_display to the Solr document with the concatenated value.  If there is no 130 field in the MARC record, look for a 240 field, concatenate all alphabetic subfield values, then add a field named title\_uniform\_display to the Solr document with the concatenated value.
> "join" - all the values indicated by the !fieldSpec will be concatenated into a single value.
> "all" - each value indicated by the fieldSpec will be a separate field in the Solr document.

**getAllSubfields(fieldSpec, separator)**:
> operates similarly to the standard indexing specification, but it provides a little more power. If your author field specification was as follows:

> `author_person_search = 100abcdefghijklmnopqrstuvwxyz:700abcdefghijklmnopqrstuvwxyz`

> You could shorten it and make it easier to read thusly:

> `author_person_search = custom, getAllSubfields(100:700,  " ")`

> Or, if you needed to be sure that only alphabetic subfields would be included:

> ` author_person_search = custom, getAllSubfields(100[a-z]:700[a-z],  " ") `

**getAllSearchableFields(lowerBound, upperBound)**:
> retrieves the contents of all fields/subfields where the field tag is greater or equal to the lower bound parameter, and less than the upper bound parameter. This is useful for grabbing all data from all fields to put into a default, catch-all search field.  Usually you would want to specify 100 as the lower bound and 900 as the upper bound thusly:

> `marc_text = custom, getAllSearchableFields(100, 900)`

**getEra**:
> gets the era field values from 045a, which contains a coded time period (see http://www.loc.gov/marc/bibliographic/bd045.html ).  When combined with a translation map provided by the SolrMarc code, values are end-user friendly.  For example:
> > `composition_era_facet = custom, getEra, composition_era_map.properties`

**getFullTextUrls**:

> gets the URLs for the full text of a resource described by the MARC record.
> For each MARC 856 field,
> > if the second indicator is 0, _each_ subfield u is added to the Solr document.
> > if the second indicator is 2, no value is added to the Solr document.
> > otherwise, subfields 3 and z are examined.  If _none_ of the following text is present, then subfield u is added to the Solr document
      * abstract
      * description
      * sample text
      * table of contents
> > `url_fulltext = custom, getFullTextUrls`

**getLinkedField(fieldSpec)**:

> allows you to retrieve the linked original-language version of a given field, if such an original language version exists, a 880 field. For example, items that are originally in Chinese, Japanese, Korean, Arabic, Hebrew, or Russian, the cataloger usually transliterates the title and author fields from their original language into a latin-alphabet, phonic representation of the original title or author string, placing the original title or author in a 880 field with a tag indicating that that entry is linked to the transliterated title or author string in the main body of the MARC record.  For the example record below, if you wanted to extract the original version of the title, you could use the following specification:

> `linked_title_display = custom, getLinkedField(245abc)`

> Which would extract  `"紫禁城宮殿 /于倬雲主編 ; [攝影高志強, 胡錘]."` for the field `linked_title_display`.

```
LEADER 01584cam a2200397 a 4500
001 u2103
003 SIRSI
005 20030106152914.0
008 840523s1982    cc af    b    000 0 chi d
010   $a   83111383
020   $a9620750012
035   $a(Sirsi) o10765514
040   $aViBlbV$cViBlbV$dOrU$dCLASIA$dMH-HY$dNcU$dOCoLC
043   $aa-cc-pe
066   $c$1
082 00$a725/.17/0951156
245 00$6880-01$aZi jin cheng gong dian /$cYu Zhuoyun zhu bian ; 
       [she ying Gao Zhiqiang, Hu Chui].
260   $6880-02$aXianggang :$bShang wu yin shu guan Xianggang fen guan, $c1982.
300   $a331 p. :$bchiefly ill. (some col.) ;$c37 cm.
500   $6880-03$aIncludes 1 portfolio (2 leaves of plates): Zi jin cheng gong
       dian yu xi yuan qian.
500   $aErrata slip inserted.
504   $aBibliography: p. 328.
596   $a4
610 20$aForbidden City (Beijing, China)
650  0$aPalaces$zChina$zBeijing.
700 1 $6880-05$aYu, Zhuoyun.
700 1 $6880-06$aGao, Zhiqiang.
700 1 $6880-07$aHu, Chui.
880 00$6245-01/$1$a紫禁城宮殿 /$c于倬雲主編 ; [攝影高志強, 胡錘].
880   $6260-02/$1$a香港 :$b商務印書館香港分館,$c1982.
880   $6500-03/$1$aIncludes 1 portfolio (2 leaves of plates): 紫禁城宮殿玉璽原鈐.
880 1 $6700-05/$1$a于倬雲.
880 1 $6700-06/$1$a高志強.
880 1 $6700-07/$1$a胡錘.
987   $c20010620$dc$e06.12.2001$f500-a /
```

> The !fieldSpec will be treated in the standard fashion, except _any_ !fieldSpec containing square brackets, such as `245[a-c7]` will be treated as a regular expression of desired subfields.

**getLinkedFieldCombined(fieldSpec)**:
> operates similarly to `getLinkedField`, including not only the original language version of the specified field, but also the cataloger created transliterated version. This would be done for fields used for searching so that a user could search by title by specifying either the transliterated version or the original language version. For the example record, if you wanted to extract the original and the transliterated versions of the title, you could use the following specification:

> `title_text = custom, getLinkedFieldCombined(245a)`

> Which would extract `“紫禁城宮殿”` and “Zi jin cheng gong dian” for the field `title_text`.

> The !fieldSpec will be treated in the standard fashion, except _any_ !fieldSpec containing square brackets, such as `245[a-c7]` will be treated as a regular expression of desired subfields.

**getSortableAuthor**:
> A sortable author value is added to the Solr document.  This value is a string containing the concatenated values of:
    1. the main entry author, if there is one (fields 100, 110 or 111)
> > 2. the main entry uniform title (240), if there is one - not including non-filing chars as noted in 2nd indicator of the 240
> > followed by
> > 3. the 245 title, not including non-filing chars as noted in ind 2 of the 245


> NOTE: in your Solr schema.xml file, you'll want to ensure this field as the desired properties.  Do you want sorting to be case sensitive?  Do you want it to normalize diacritics?  Do you want it to remove preceding or trailing whitespace?  You may want to use the alphaOnlySort field type provided in the standard Solr schema.xml file.

**getSortableTitle**:
> retrieves the value from the 245a subfield, and then strips off any "non-sorting characters" at the beginning of the string, according to the MARC records indicators at the head of the field.

> NOTE: in your Solr schema.xml file, you'll want to ensure this field as the desired properties.  Do you want sorting to be case sensitive?  Do you want it to normalize diacritics?  Do you want it to remove preceding or trailing whitespace?  You may want to use the alphaOnlySort field type provided in the standard Solr schema.xml file.

**getSupplUrls**:
> gets the URLs for the full text of a resource described by the MARC record.
> For each MARC 856 field,
> > if the second indicator is 0, no value is added to the Solr document.
> > if the second indicator is 2, _each_ subfield u is added to the Solr document.
> > otherwise, subfields 3 and z are examined.  If _any_ of the following text is present, then subfield u is added to the Solr document
      * abstract
      * description
      * sample text
      * table of contents
> > `url_suppl = custom, getSupplUrls`

**getTitle**:

> Create a Solr field containing the 245a (and 245b, if it exists, concatenated with a space between the two subfield values), with trailing punctuation removed.  Removed punctuation includes space, comma, slash, semicolon, colon, trailing period if it is preceded by at least three letters, and single square bracket characters if they are the start and/or end chars of the cleaned string

**removeTrailingPunct(fieldSpec)**:
> creates a list of entries extracted from the fields/subfields specified by the `fieldSpec` parameter, and then post-processes all of the entries in the list to remove any trailing punctuation from the entries.  So if you were defining a index entry like this

> `title_display = 245a`

> but you were unhappy with the colons, commas, and slashes which appear on the entries that end up being stored in the index, you could change the index specification to be:

> `title_display = custom, removeTrailingPunct(245a)`

> The specific characters that are removed are: space, comma, slash, semicolon, colon, trailing period if it is preceded by at least three letters, and single square bracket characters if they are the start and/or end chars of the cleaned string.

> The !fieldSpec will be treated in the standard fashion, except _any_ !fieldSpec containing square brackets, such as `245[a-c7]` will be treated as a regular expression of desired subfields.