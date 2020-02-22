(This document will likely move to a vignette in the R package in the future.)

# Data Contracts

Data management is the initial step in almost any project analyzing data, and it
can also be the most time-consuming part of the analysis.

The objective of data contracts is to simplify the data management process by
defining in a more human-readable form the data to be managed. The concepts will
be that from the simple-to-specify data contract, the minimum amount of code
should be required to convert the raw data to being analysis-ready.

For data contracts, the data are assumed to be two-dimensional. No restriction
is placed on the initial, intermediate, or final data formats other than the
general idea that they are two-dimensional. (And, the concepts may allow for
higher dimensional data.)

Overall, data contracts are built-up in blocks where more specific blocks (to
the right in data contract language) may define blocks to the left, and each
block may be defined by one or more sub-contracts. Data within a sub-contract is
assumed to have fully-fulfilled its subcontract. An example diagram of how data
contract and sub-contract blocks may be arranged to make a dataset is below
(where each rectangular region would be considered to be a subcontract).

```
+-----------------------+
|                       |
|                       |
+-----+-----+-----------+
|     |     |           |
|     |     |           |
|     +-----+-----+-----+
|     |     |     |     |
|     |     |     |     |
|     |     |     +-----+
|     |     |     |     |
|     |     |     |     |
|     |     +-----+-----+
|     |     |     |     |
|     |     |     |     |
+-----+-----+-----+-----+
```

# Layers of Data Management

Data management often may happen in stages where multiple datasets are combined
together. For a data contract, contracts are made for one block (set of
2-dimensional) data, and these blocks can be combined by binding rows or columns
together.

The initial layer is considered as a single column, and the final layer is
considered as the completed dataset. To maximize simplicity of data managemet
code, data management should start from the simplest level and progress to more
complex levels only when required. Also, any data management that can happen
later in the code should be delayed as much as possible to prevent code
duplication for performing the same operation more than once.

## Column Data Contracts

Columns typically have limits on the information that they may contain within
data management.

The initial restriction on a column is its data type. In R, typical data types
are logical, integer, numeric, character, factor, Date, and date-time (POSIXct),
and more complex types are possible.

**Restriction**: All columns must have a data type assigned.

For all columns regardless of the type of data, they may have constraints on
missingness of data. (`NULL` in SQL terminology, or less precisely `NA` in R
terminology.) Typically, missingness is either allowed or not within a column.

**Restriction**: All columns must have an allowed missingness definition.

For columns containing continuous or discrete numeric data (e.g. floating point
numbers, integers, dates, time durations, boolean/logical, etc.), range
qualifiers may be placed on the data. Range qualifiers may include information
like the fact that a value must be non-negative (>=0) or that a date must occur
in a given year.

**Suggestion**: Continuous and discrete columns should have range limits,
whenever possible. (Even if those range limits are that the value must not be
infinite; infinity should always be considered if it is allowed in a range.)

For columns containing character data types (including characters with limited
vocabularies like factors in R), value qualifiers may be placed on the data.
Value qualifiers may include the fact that a value must be `"A"`, `"B"`, or
`"C"`, but `"D"` (or any other letter) is not allowed. Additionally, the
character data may come in with a mixture of upper and lower case, and the
column may or may not be case-sensitive.

**Suggestion**: Character columns should have limits on the value of the column,
whenever possible.

Any column may also have restrictions on duplication such that values within the
column must unique or not. The default is that a column may have duplicated
values.

## Multi-Column Data Contracts

Some columns may depend on other columns. In a simple example, a character value
of `"A"` may need to be mapped to a numeric value of `1`. In a more compelex
example, a birth date must be before a death date for an individual.

One of the simplest multi-column data contracts is that when one column has a
value, that fully defines the value of one or more other columns. The 1:1
mapping is a merge (or SQL join). At the end of applying a merge, rules for
missingness at the column-level should be re-examined.

# Combining Blocks

At its core, any set of multi-column data contracts defines the data contract
for a block of data (any rectangular region in a data set). For simplicity, once
a data contract is verified for a block, no additional tests are required. With
the rule that when a contract is verified no additional tests are required,
combining data contracts becomes simpler because tests are only required outside
of the block and not within the block.

**Restriction**: To combine two data contracts by row, they must have the same
names.

To ensure that no additional tests are required, combination of two data
contracts by row (`rbind` in R) requires that the data contracts have the same
column names. While the requirement of having the same names may seem
restrictive, it prevents the error of arbitrary values being added to a missing
column name when they are combined (e.g. with `dplyr::bind_rows()` creating
values of `NA`; those `NA` values may violate the contract).

**Restriction**: No modifications may occur to verified column data after
verified contract blocks have been combined.

Once two verified blocks have been combined, they automatically are a verified
larger block. New columns may be added to the verified block, but no
modifications may occur within the verified block as that could violate the
contract. And, due to the complexity of managing contract verification when
different blocks may have different rules for verification.

Additional columns may be added outside of the verified columns in a combined
block. And, it is generally preferred to add columns as late as possible after
combining blocks so that verification can be as broad as possible.

Row permutations (e.g. sorting rows of a dataset) are allowed after
verification. Row permutation may cause multiple sub-contracts to become mixed,
but because a contract must be verified before it can be combined, the mixture
does not cause any verification issues inherently. In cases where mixture or
permutation may cause verification issues, the contract that contains the
sub-contracts must have a rule to allow the permutation. (Essentially, the outer
contract must have all rules that make it valid even when they may affect the
inner contracts.)
