# FHISO Legacy Serialisation (FHISO-L): Generic Syntax

*This document is a draft by Richard Smith <`rsmith@fhiso.org`> for
consideration by FHISO.  It is not endorsed by FHISO.*

## Introduction

This is a module of the [FHISO-L standard](core.md) and defines the 
generic syntax of the FHISO-L serialisation without reference to the 
specific FHISO-L data model.  The generic data representation language
defined in this chapter may be used to represent any form of structured
information in a sequential stream of characters, not just genealogical
data.

## Concepts

A FHISO-L document represents a database in the form of a sequential
stream of related records.  A record is represented as a sequence of
tagged, variable-length lines, arranged in a hierarchy.  A line always
contains a hierarchical level number, a tag, and an optional value.  A
line may also contain a cross-reference identifier or a pointer.  The
FHISO-L line is terminated by a carriage return, a line feed character,
or any combination of these.

The tag in the FHISO-L line, taken in its hierarchial context, identifies
the information contained in the line, in the same sense that a
field-name identifies a field in a database record.  This means that the
data is self-defining.  Tags allow a field to occur any number of times
within a record, including zero times.  They also allow the use of
different or new fields to be included in the FHISO-L data without
introducing incompatibility, because the receiving system will ignore
data which it does not understand and process only the data that it does
understand.

The hierarchical relationships are indicated by a level number.
Subordinate lines have a higher level number.  The hierarchy allows a
line to have sub-lines, which in turn may have their own sub-lines, and
so forth.  A line and its sub-lines constitute a context or enclosure,
that is, a cluster of information pertaining directly to the same thing.
This hierarchical arrangement corresponds with the natural hierarchy
found in most structured information.  A series of one or more lines
constitutes a record.  The beginning of a new record is indicated by a
line whose level number is 0 (zero).  In addition to hierarchical
relationships, FHISO-L defines the inter-record relationships that allow
a record to be logically related to other records, without introducing
redundancy.  These relationships are represented by two additional, but
optional, parts of a line: a cross-reference pointer and a cross-
reference identifier.  The cross-reference pointer "points at" a related
record, which is identified by a required, matching unique
cross-reference identifier.  The cross-reference identifier is analogous
to a primary key in relational database terminology.

## Grammar

This chapter defines the grammar for the FHISO-L format.  The grammar is
a set of rules that specify the character sequences that are valid for
creating the FHISO-L line.  The character sequences are described in
terms of various combinations of elements (variables and/or constants).
Elements may be described in terms of a set of other elements, some of
which are selected from a set of alternative elements.  Each element in
the definition is separated by a plus sign (`+`) signifying that both
elements are required.  When there is a choice of different elements that
can be used, the set of alternatives are listed between opening and
closing square brackets (`[]`), with each choice separated by a vertical
bar (`[alternative_1 | alternative_2]`).  When there is only one
alternative shown then the choice is optional, that is, it is the same
as `[alternative_1 | <NULL>]`. The user can read the grammar components
of the selected element by substituting any sub-elements until all
sub-elements have been resolved.

A FHISO-L transmission consists of a sequence of logical records, each of
which consists of a sequence of `gedcom_lines`, all contained in a
sequential file or stream of characters.  The following rules pertain to
the `gedcom_line`:

### Grammar Rules

* Long values can be broken into shorter FHISO-L lines by using a
subordinate `CONC` or `CONT` tag.  The `CONC` tag assumes that the
accompanying subordinate value is concatenated to the previous line
value without saving the carriage return prior to the line terminator.
If a concatenated line is broken at a space, then the space must be
carried over to the next line.  The `CONT` assumes that the subordinate
line value is concatenated to the previous line, after inserting a
carriage return.
* The beginning of a new logical record is designated by a line whose
`level` number is 0 (zero).
* Level numbers must be between 0 to 99 and must not contain leading
zeroes, for example, level one must be 1, not 01.
* Each new level number must be no higher than the previous line plus 1.
* All FHISO-L lines have either a `value` or a `pointer` unless the line
contains subordinate FHISO-L lines.  The presence of a level number and
a tag alone should not be used to assert data (i.e. `1 FLAG Y` not just
`1 FLAG` to imply that the flag is set).
* Logical FHISO-L record sizes should be constrained so that they will
fit in a memory buffer of less than 32K.  FHISO-L files with records
sizes greater than 32K run the risk of not being able to be loaded in
some programs.  Use of pointers to records, particularly `NOTE` records,
should ensure that this limit will be sufficient.
* Any length constraints are given in characters, not bytes. When wide
characters (characters wider than 8 bits) are used, byte buffer lengths
should be adjusted accordingly.
* The cross-reference ID has a maximum of 22 characters, including the
enclosing 'at' signs (`@`), and it must be unique within the FHISO-L
transmission.
* Pointers to records imply that the record pointed to does actually
exists within the transmission.  Future pointer structures may allow
pointing to records within a public accessible database as an
alternative.
* The length of the FHISO-L TAG is a maximum of 31 characters, with the
first 15 characters being unique.
* The total length of a FHISO-L line, including level number,
cross-reference number, tag, value, delimiters, and terminator, must not
exceed 255 (wide) characters.
* Leading white space (tabs, spaces, and extra line terminators)
preceding a FHISO-L line should be ignored by the reading system.
Systems generating FHISO-L should not place any white space in front of
the FHISO-L line.

### Grammar Syntax

A `gedcom_line` has the following syntax:
```
gedcom_line :=
  level + delim + [optional_xref_ID] + tag + [optional_line_value] + terminator
```
for example:
```
1 NAME Will /Rogers/
```

The components used in the pattern above are defined below in
alphabetical order.  Some of the components are defined in terms of
other primitive patterns.  The spaces used in the patterns below are
only to set them apart and are not a part of the resulting pattern.
Character constants are specified in the hex form (0x20) which is the
ASCII hex value of a space character.  Character constants that are
separated by a (-) dash represent any character with in that range
from the first constant shown to and including the second constant
shown.

```
alpha :=
  [(0x41)-(0x5A) | (0x61)-(0x7A) | (0x5F)]

alphanum :=
  [alpha | digit]

any_char :=
  [alpha | digit | otherchar | (0x23) | (0x20) | (0x40)+(0x40)]

delim :=
  [(0x20)]

digit :=
  [(0x30)-(0x39) ]

escape :=
  [(0x40) + (0x23) + escape_text + (0x40) + non_at ]

escape_text :=
  [any_char | escape_text + any_char ]

level :=
  [digit | digit + digit ]

line_item :=
  [any_char | escape | line_item + any_char | line_item + escape]

line_value :=
  [pointer | line_item]

non_at :=
  [alpha | digit | otherchar | (0x23) | (0x20 ) ]

null := nothing

optional_line_value := delim + line_value

otherchar :=
  [(0x21)-(0x22) | (0x24)-(0x2F) | (0x3A)-(0x3F) | (0x5B)-(0x5E) | (0x60)
    | (0x7B)-(0x7E) | (0x80)-(0xFE)]

pointer :=
  [(0x40) + alphanum + pointer_string + (0x40) ]

pointer_char :=
  [non_at ]

pointer_string :=
  [null | pointer_char | pointer_string + pointer_char ]

tag :=
  [alphanum | tag + alphanum ]

terminator :=
  [carriage_return | line_feed | carriage_return + line_feed
    | line_feed + carriage_return ]

xref_ID :=
  [pointer]
```

## Description of Grammar Components

alpha :=
  ~ The alpha characters include the underscore, which is used to link
    word pieces together in forming tag names or tag labels.

any_char :=
  ~ Any 8-bit ASCII character except the control characters found in the
    range of 0x00–0x1F and 0x7F.

delim :=
  ~ The `delim` (delimiter), a single space character, terminates both
    the variable-length `level` number and the variable-length `tag`.  
    Note that espace characters may also be present in a value.

escape :=
  ~ The `escape` is a character sequence in the grammar used to specify
    special processing, such as for switching character sets or for
    indicating an inclusion of a non-FHISO-L data form into the FHISO-L
    structure.  The form of the escape sequence is: 

    ```
    @ + # + escape_text + @ + non_at
    ```

    Receiving systems should discard any space character which follows
    the escape sequence's closing at-sign (`@`).  If the character
    following the escape sequence's closing at-sign (`@`) is not a
    space character then it should be kept as a part of the text
    following the escape.  Systems writing escape sequences should
    always output a space character following the escape sequence.  

    The specific format of the escape sequence is defined for the
    specific FHISO-L form being defined.

escape_text :=
  ~ The `escape_text` is defined to meet the requirements of a
    particular `FHISO-L` form.

level :=
  ~ The level number works the same way as the level of indentation in
    an indented outline, where indented lines provide detail about the
    item under which they are indented.  A line at any level L is enclosed
    by and pertains directly to the nearest preceding line at
    level L-1.  The Level L may increase by 1 at most. Level numbers
    must not contain leading zeroes, for example level one must
    be (1), not (01).

    The enclosed subordinate lines at level L are said to be in the
    context of the enclosing superior line at level L-1.  The
    interpretation of a tag must be in the context of the tags of the
    enclosing line(s) rather than just the tag by itself. Take the
    following record about an individual's birth and death dates, for
    example:

    ```
    0 INDI
    1 BIRT
    2 DATE 12 MAY 1920
    1 DEAT
    2 DATE 1960
    ```
    
    In this example, the expression `DATE 12 MAY 1920` is interpreted
    within the `INDI` (individual) `BIRT` (birth) context, representing
    the individual's birth date.  The second `DATE` is in the
    `INDI.DEAT` (individual's death) context.  The complete meaning of
    `DATE` depends on the context.

    *Note: The above example is indented according to the level numbers
    to make the concept more obvious.  In the actual FHISO-L data, the
    level numbers are lined up vertically, meaning they are the first
    character(s) of the FHISO-L line.*

    Some systems output indented FHISO-L data for better readability by
    putting space or tab characters between the terminator and the level
    number of the next line to visibly show the hierarchy.  Also, some
    people have suggested allowing extra blank lines to visibly separate
    physical records. FHISO-L files produced with these features are not
    to be used when transmitting FHISO-L to other systems.

line_value :=
  ~ The `line_value` identifies an object within the domain of possible
    values allowed in the context of the `tag`.  The combination of the
    `tag`, the line_value, and the hierarchical context of the
    supporting `gedcom_lines` provides the understanding of the enclosed
    values.  This domain is defined by a specific grammar for
    representing a given FHISO-L form.

    Values whose source information contains illegible parts of the
    value should be indicated by replacing the illegible part with an
    ellipsis (`...`).  

    Values are generally not encoded in binary or other abbreviation
    schemes for reducing space requirements, and they are generally
    constrained to be understandable by a typical user without decoding.
    This is intended to reduce the decoding burden on the receiving
    software.  A FHISO-L-optimized data compression standard will be
    defined in the future to reduce space requirements.  Meanwhile,
    users may agree to compress and decompress FHISO-L files using
    any compression system available to both sender and receiver.

    The `line_value` within the context of a tag hierarchy of
    `gedcom_line`s represents one piece of information and corresponds
    to one field in traditional database or file terminology.

otherchar :=
  ~ Any 8-bit ASCII character except control characters (0x00–0x1F),
    `alphanum`, space (` `), number-sign (`#`), the at sign (`@`), and
    the DEL character (0x7F).

pointer :=
  ~ A `pointer` stands in the place of the record or context identified
    by the matching `xref_ID`.  Theoretically, a receiving system should
    be prepared to follow a `pointer` to find *any( needed value in a
    manner that is transparent to the logic of the subsystem that is
    looking for specific tags.  This highly flexible facility will
    probably be used more in the future.   For the time being, however,
    the use of pointers is explicitly defined within the FHISO-L form.

    The pointer represents the association between two objects that
    usually reside in different records.  Objects within a logical
    record can be associated.  If this need exists, the pointer record
    composition contains an exclamation point (`!`) that separates the
    parent record's cross-reference ID from the specific substructure's
    cross-reference ID, which is at some subordinate level to the
    logical record at level zero. The cross-reference ID of the
    substructure subordinate to a zero level record, for inter-record
    associations is always composed of the Record ID number and the
    Substructure ID number, such as `@I132!1@`. Including the Record ID
    number in the pointer that associates objects within a record will
    allow the FHISO-L processors to build the index only at the record
    level and then search sequentially for the appropriate substructure
    cross-reference ID.  The parent record ID is assumed when the
    cross-reference ID begins with a exclamation point (`!`) signifying
    an intra-record association.

    Complex logical record structures are divided into small physical
    records to accommodate memory constraints, many-to-many
    relationships, and independent record creation and deletion.

    The `pointer` must match a corresponding unique `xref_ID` within the
    transmission, unless the colon (`:`) character is present (which will
    be used in the future as a network reference to a permanent file
    record).  A `pointer` is given instead of duplicating an object,
    though the logical result is equivalent.  An expanded traversal of a
    record tree includes following the `pointer` to related records to
    some depth, and splicing those records (logically) into the
    resultant expanded tree.  *Pointers may refer to either records
    which have not yet appeared in the transmission (forward reference)
    or to records that have already appeared earlier in the transmission
    (backward reference).*  This arrangement usually requires a
    preliminary pass to construct a look up table to support random
    access by `xref_ID` during subsequent passes.

tag :=
  ~ A `tag` consists of a variable length sequence of `alphanum`
    characters.  All user-defined tags that have not been defined in the
    FHISO-L standard, must begin with an underscore character (0x95).

    The `tag` represents the meaning of the `line_value` within the
    context of the enclosing tags, and contributes to the meaning of the
    enclosed subordinate lines.  Specific tags are defined in various
    FHISO-L standard modules.  The presence of a tag together with a
    value represents an assertion which the submitter wishes to
    communicate to a receiver.  A tag without a value does not represent
    an assertion.  If a tag is absent, no assertion is made.
    Information of a negative nature (such as knowing positively an
    event did not occur) is handled through the semantic definition of a
    different tag and its accompanying value that assert the information
    explicitly.

    Although formally defined tags are only three or four characters
    long, systems should prepare to handle user tags of greater length.
    Tags will be unique within the first 15 characters.

    Valid combinations of specific `tag`, `line_value`, `xref_ID`, and
    `pointer` constructs are constrained by the FHISO-L form defined in
    the relevant FHISO-L standard module.

terminator :=
  ~ The `terminator` delimits the variable-length `line_value` and
    signals the end of the `gedcom_line`.  The valid terminator
    characters are:

    ```
    [carriage_return | line_feed | carriage_return + line_feed
      | line_feed + carriage_return ]
    ```

xref_ID :=
  ~ (See also `pointer`.)

    The `xref_ID` is formed by any arbitrary combination of characters
    from the `pointer_char` set.  The first character must be an `alpha`
    or a `digit`.  The `xref_ID` is not retained in the receiving
    system, and it may therefore be formed from any convenient combination
    of identifiers from the sending system.  No meaning is attributed by
    the receiver to any part of the xref_ID, other than its unique
    association with the associated record.  The use of the colon (`:`)
    character is also reserved.


### Examples

The following are examples of valid but unrelated GEDCOM lines:

```
0 @1234@ INDI
1 AGE 13y
1 CHIL @1234@
1 NOTE This is a note field that is
2 CONT continued on the next line.
```

The first line has a level number 0, an `xref_ID` of `@1234@`, an `INDI`
tag, and no value.

The second line has a level number 1, no `xref_ID`, an `AGE` tag, and a
value of 13y.

The third line has a level number 1, no `xref_ID', a `CHIL' tag, and a
value of a pointer to a `xref_ID` named @1234@.
