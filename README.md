Management > Namespace
Think of namespace as a logical database where streams and other objects are stored by account.

Management > Tenants
(Account-level should be hyphenated)

Sequential Data Store >
The Sequential Data Store (SDS) is a cloud-based streaming data storage that is optimized for storing sequential data, usually a time-series, but anything that is indexed by an ordered sequence. You use SDS to store, retrieve, and analyze data. 

An SdsType (used interchangeably with type throughout documentation) defines the shape of a single measured event or object. A type gives structure to your data. For example, if you're measuring three things (longitute, latitude, speed) from a device at the same time, then you want those three properties to be included in your tytpe. An SdsStream (used interchangeably with stream throughout documentation) is an instance of the type you have defined. Each stream is made up of a series of instances (or events). 


===

Types are equivalent to int32, float32, string. Can be defined in any way you like. Things that are measured at the same time (index), at the same place on a same device. In a sensor in an application, you want to give it a structure. You define a type with the way the data should be represented in mind. Do it for the application reason, not for the storage persistence reason. If the sensor is measuring three things at the same time, you should store those in the same type. 

Streams are instance of the type. Each stream will have a series of events. 

Namespace a container for types and streams. 

 [Indexes](xref:sdsIndexes)



# markdownshowdown
## This would be the second heading
### Am I the third heading?
#### Me fourth?
Hello world
===========
Hello world mini
-----------------
Emphasis in *italitcs*  
Emphasis in **bold**  
Combined in **_bold italics_**  
Strikethrough is ~~Scratch this~~  

<table>
  <tr>
    <td>Hello</td>
  </tr>
  <table>
    
>Block Quotes
>> Nested block quotes

unblocked
quotes

* for list items, use asterisk with space
* list
* list

- same with a hyphen for a list
- list
- list

+ or a plus sign
+ another plus sign for a list

1. numbers mark a ordered list
1. any number will do
1. numbers are awesome

* A list with a code block:
    Code block with two tabs or eight spaces
