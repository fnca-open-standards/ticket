# Open Ticket Specification

The Open Ticket is a JSON format that can be used to send and receive 811 tickets.  Traditionally, tickets are sent in formats like text, html, xml, and json to recipients, but those are formats defined by each center.  The Open Ticket is an open specification that allows any 811 ticket to be represented in a common JSON format.  This includes any ticket type with any number of custom fields.  

Fields that are common to all tickets (like a Ticket Number, Excavator information, and work site information) are standardized and placed in sections for organization.  These are the "Core" fields for tickets.

Fields that are custom to a center are contained in the custom field groups section after all the core ticket sections.  The custom fields are sent with their names, types, values, and other meta-data to instruct the receiver on what the field represents and where it belongs on the ticket.

Each section below includes a description of what that section contains, followed by a sample in JSON format.  Finally, each field in the section is described along with its data type and maximum length (for strings).


## Data Types

Common and Custom field should conform to the following value types:

**DateTime** - All dates should be transmitted in ISO 8601 format with time zone information.  For greatest compatibility, use millisecond precision (3 decimal places of precision).  Dates for excavation tickets are very close to the present time, so the minimum and maximum values of all receiving systems should never be an issue.

**Integer** – A signed value that can be represented by 32 bits, from -2,147,483,648 through 2,147,483,647.  This is a signed value because most systems default to a signed integer.  A Max value can be specified if appropriate.  For instance, if the custom field should only have a value up to 99, then set this as the Max value.

**Float** – A signed floating point value that can be represented by 32 bits.

**String** – A string value, the value of Max indicates the maximum length of the string.  The Max should be specified for all custom string fields.  This can help the receiving system store as well as display this value appropriately.

**Text** – A string with no maximum length.  The use of these fields should be very limited (for instance, only used for WKT values).  

**Boolean** – The value for these custom fields can be True, False, or Null.

The type for custom fields is specified in its Type value as part of its definition.  Common field types have value types and valid values or lengths defined in their descriptions below.

# Open Ticket Format Sections

**Header** – Information to identify the ticket and process it.

**Work Site** – information about where the excavation is happening.

**Geometry** - all geometric information for the ticket.

**Locate Information** – Locate instructions, remarks, and driving directions.  These are needed by the locator to properly locate the facilities.

**Project** – Additional information about the excavation project.  This is not required for the location, but is useful in damage prevention, such as whether or not explosives will be used, or the expected duration of the excavation.

**Timeline** – Dates commonly associated with tickets.  Dates specific to a ticket type should be in Custom Fields instead.

**Excavator** – information about the excavator.

**Custom Fields** – custom fields with their metadata for this specific ticket type.

**Members** – The utility companies that are being notified to locate facilities.

**Response History** – This section shows the response history for this ticket.  Initially, this may be blank.  Any resends of the ticket would contain the response history as of the resend date and time.

**Attachments** - Section to include binary attachments in the file.

# Header

The first section of the JSON format is the header object.   The purpose of the header is to identify the ticket and other data regarding the transmission.  The inspiration for this section is that of an IP header that contains information about the delivery and routing of an IP packet.  In this case, it is about the delivery and potential routing of a ticket.

```
{
  "header": {
    "formatVersion": "0.5",
    "ticketNumber": "230815-001003",
    "ticketVersion": 0,
    "sequence": 12,
    "source": "Voice",
    "type": "Normal",
    "standardType": "Normal",
    "action": "New",
    "class": null,
    "priority": "Normal",
    "isTest": false,
    "centerName": "GA811",
    "memberIds": [
      "GAUPC"
    ],
    "trace": [
      {
        "dateTime": "2023-08-08T07:32:05.493-04:00",
        "action": "sent",
        "host": "GA811 GeoCall"
      },
      {
        "dateTime": "2023-08-08T07:35:26.157-04:00",
        "action": "received",
        "host": "BigCity Lights EMC BCSTicket"
      },
      {
        "dateTime": "2023-08-08T10:42:22.736-04:00",
        "action": "sent",
        "host": "BigCity Lights EMC BCSTicket"
      },
      {
        "dateTime": "2023-08-08T10:43:01.412-04:00",
        "action": "received",
        "host": "Accurate Locating, Inc. Dispatch"
      }
    ]
  },
```

**Format Version (formatVersion)** string (10) – This is the version of the Open Ticket Format specification this JSON document represents.  When new versions of the specification are developed, this will allow receivers to know what version of the specification the transmission follows.  For centers not yet implementing newer versions, they can continue to send, and receivers can continue to receive for that version of the spec.  Once a center or receiver has upgraded their system to the latest spec, then they can coordinate the changeover to the new format.

**Ticket Number (ticketNumber)** string (25) – this is the ticket number.

**Ticket Version (ticketVersion)** integer - the version number for the ticket, if the source system uses ticket versioning.

**Sequence (sequence)** integer – this is a serial number of this transmission for this receiver.   This allows the receiver to know what order tickets were sent to them in a day.  Since this format can be used for transmissions of tickets as well as retrieval of tickets by API call, this will only have a value if this is a transmission.
[This is very much like a session number for networking.  This identifies the order the ticket was transmitted each day, as if each ticket were a packet sent in a daily session]

**Source (source)** string (30) – this is the channel (Call Center, Web Portal, etc.) the ticket was submitted through.

**Type (type)** string (60) – the legal ticket type for this ticket.

**Standard Type (standardType)** string (60) – the standard ticket type for this ticket.  This should be mapped to a national standard.  For instance, “Design Request” might be the legal ticket type defined by a state law, but “Design” would be an appropriate “standard” type.  Each center should define their ticket type mappings to a national standard.

- Normal - standard new production ticket for normal excavation and notice periods
- Emergency - immediate work that must be performed to eliminate danger to life, health, property, etc.
- Short Notice - work that must begin before the normal notice period, but not an emergency ticket
- Design - non-excavation work used for planning purposes
- No response - notification to a member that they have not marked in the time provided
- Cancel - a notification of a cancellation of another ticket
- Correction - a replacement ticket with updated information
- Remark - a ticket sent to request updated markings on an existing ticket
- Update - a ticket to renew the excavation for additional time
- Damage - a notification of a damage
- Project - a longer-term excavation than a Normal ticket
- Other - any ticket that does not fit another category

**Action (action)** - The action that caused this ticket to be created, such as New (for a new request), Resend, Cancel, 2nd Notice, Relocate, Late, etc.

**Class (class)** - Classification for the ticket, if used by the center.  Centers that use qualifiers in their ticket types can use class to modify the base Type (above).  Centers that do not use Class should send a NULL value.

**Priority (priority)** string (10) – This helps the receiver determine how fast this ticket needs to be processed.  Values may include:
	Normal – This should be processed after Rush or Emergency tickets
  Rush – This should be processed before Normal priority tickets, but after emergency priority tickets
	Emergency – This should be processed before other priority types.

**IsTest (isTest)** boolean – If implemented by the center, this would allow them to send test tickets to members and have them identify those tickets as test tickets.  Although most testing is done using test systems, there is still a need for test tickets to be sent through production systems.  This would allow the identification of test tickets that do not require an actual locate to be performed. If this is not implemented by the center, a NULL value should be sent.

If this is not implemented by the center’s system, then this flag should indicate Null.  This will tell the receiver that the sending system does not implement this feature, rather than False which would indicate the center positively identifying this as a production ticket.

**Center Name (centerName)** string (25) – an identifier for the center transmitting the ticket.  This is most often the state abbreviation plus “811”.

**Member Ids (memberIds)** string (10) array – this is an array of string values containing the alphanumeric member identifiers for the facilities included in this transmission.  This allows a single ticket to be delivered for multiple facilities for a single operator.  For instance, a member who operates water and sewer facilities may receive a single ticket that includes the codes for the water and sewer facilities they operate, so this would contain both member ids corresponding to those facilities.

This is helpful for routing tickets to the appropriate internal group for the recipient.  For instance, contract locator companies with many customers can use this to route the ticket to the appropriate locator.

Since this format can be used for transmissions of tickets as well as retrieval of tickets by API call, this will only have a value if this is a transmission.

**Trace (trace)** array of trace objects - this section contains the trace information for this ticket.  The optional trace information is appended to the ticket by each system that processes a ticket.  This allows for inspecting the path the ticket has taken through various systems.  This information will be more critical as more systems are used for processing and forwarding tickets.

> **DateTime (dateTime)** DateTime - The date and time of the event (ticket send or receive).
>  
> **Action (action)** string (20) - typically "send" or "receive".  What action is being taken on the ticket.  Other actions can be specified, such as routing to internal processing or groups.
> 
> **Host (host)** string (100) - an identifier to show the organization and host name of the system processing the ticket for this action.

# Work Site

The work site information is all the information regarding where the excavation is occurring.  The information in this section is used by the receiver to identify the location of the work site.

```
"workSite": {
    "countryCode": "US",
    "stateProvince": "GA",
    "county": "Cobb",
    "place": "Smyrna",
    "fullAddress": "123 Another LN SE",
    "addressNumber": "123",
    "addressPrefix": "",
    "addressStreetName": "Another",
    "addressStreetType": "LN",
    "addressSuffix": "SE",
    "intersection": "Highway 25"  
  },
```

**Country Code (countryCode)** string (2) – specifying a country code allows this specification to be used in countries other than the United States.  This should be the ISO Alpha-2 code for the country, such as US for the United States, or CA for Canada.

**State/Province (stateProvince)** string (2) – a two-character abbreviation for the state or province.

**County (county)** string (75) – the name of the county for the work site.

**Place (place)** string (75) – the city, town, or unincorporated place name.

**Full Address (fullAddress)** string (370) – the full street address for the work site.

**Address Number (addressNumber)** string (20) – number part of the address.  This is a string, not a number, to accommodate alphabetic characters (‘34B’ for instance).

**Address Prefix (addressPrefix)** string (10) – A prefix to the street name, normally directional (North, East, West, etc.)

**Address Street Name (addressStreetName)** string (300) – The street name.

**Address Street Type (addressStreetType)** string (30) – The type of street (Rd, St, Blvd, Hwy, etc.)

**Address Suffix (addressSuffix)** string (10) – A suffix, normally directional (North, East, West, etc.)

**Intersection (intersection)** string (370) – the name of the nearest intersecting street.

# Geometry

All geometric information is available in this section

```
"geometry":{
    "srid": 4326,
    "longitude": -84.548269,
    "latitude": 33.855170,
    "secondaryLongitude": -84.545451,
    "secondaryLatitude": 33.857520,
    "boundaryArea": ["MULTIPOINT((-83.782915, 32.630501),(-83.780738, 32.632213))"],
    "workSiteArea": ["POLYGON((-84.54761 33.85697,-84.54761 33.85572,…))",
                    "POLYGON((…))"],
    "bufferedArea": ["POLYGON((-84.547642 33.855171,…))"],
    "geoJson":{
      "type": "FeatureCollection",
      "bbox": [-83.782915, 32.630501, -83.780738, 32.632213],
      "features": [
        {
          "type": "Feature",
          "id": 0,
          "properties": {        
	    "objectType": "excavationSite"
          },
          "geometry": {
    	     "type": "Polygon",
	    "coordinates": [
	      [
	        [-83.782266, 32.631388],
	        [-83.781601, 32.631049],
	        [-83.781387, 32.631366],
	        [-83.781961, 32.631664],
	        [-83.782266, 32.631388]
	      ]
	    ]
          }
        },
        {
          "type": "Feature",
          "id": 1,      
          "properties": {
    	    "objectType": "bufferedSite"        
              },
          "geometry": {
	    "type": "Polygon",
	    "coordinates": [
	      [
	        [-83.781694, 32.630506],
	        [-83.781566, 32.630501],
	        [-83.781438, 32.630517],
	        [-83.781317, 32.630555],
	        [-83.781207, 32.630612],
	        [-83.781113, 32.630687],
	        [-83.781038, 32.630776],
	        [-83.780824, 32.631092],
	        [-83.780772, 32.631188],
	        [-83.780743, 32.631291],
	        [-83.780738, 32.631397],
	        [-83.780758, 32.631501],
	        [-83.7808, 32.631601],
	        [-83.780864, 32.631691],
	        [-83.780947, 32.63177],
	        [-83.781047, 32.631834],
	        [-83.781621, 32.632132],
	        [-83.781735, 32.632179],
	        [-83.781857, 32.632206],
	        [-83.781984, 32.632213],
	        [-83.782109, 32.632199],
	        [-83.782229, 32.632164],
	        [-83.782339, 32.63211],
	        [-83.782434, 32.63204],
	        [-83.78274, 32.631764],
	        [-83.782818, 32.631678],
	        [-83.782875, 32.631581],
	        [-83.782908, 32.631475],
	        [-83.782915, 32.631367],
	        [-83.782898, 32.631259],
	        [-83.782855, 32.631157],
	        [-83.78279, 32.631063],
	        [-83.782704, 32.630982],
	        [-83.782601, 32.630917],
	        [-83.781936, 32.630579],
	        [-83.78182, 32.630532],
	        [-83.781694, 32.630506]
	      ]
	    ]
          }
        }
      ]
    }
},
```

**SRID (srid)** integer – This is the Spatial Reference Identifier of the coordinate system used for WKT values.  This must always be 4326 (WGS84) to ensure the same coordinate system is used as any GeoJSON transmitted with the ticket.  This field is here to be explicit and ensure the same SRID is used.  If a ticket uses a different value, it should be considered invalid.

The following 4 fields provide the upper-left and lower-right points of a rectangle that encloses the work site extent.  This rectangle could be created as a polygon in Well-Known-Text (WKT) and sent as the boundaryArea below instead.  This provides two methods of sending the extent information.

**Longitude (longitude)** float – The longitude of the left-most point of the work site.

**Latitude (latitude)** float – The latitude of the upper-most point of the work site.

**Secondary Longitude (secondaryLongitude)** float – the longitude of the right-most point of the work site.

**Secondary Latitude (secondaryLatitude)** float – the latitude of the bottom-most point of the work site.

*NOTE: Well Known Text Fields - The following fields contain Well Known Text (WKT) values.  All WKT values are expressed in the WGS84 (SRID 4326) and have a precision of 6 digits (11.1 cm resolution at the equator).*

*Additionally, any buffered geometry should minimize the number of points used in rounding endcaps.  Reducing the number of points and keeping the precision to 6 digits can speed up processing significantly, which is particularly important for high-volume receivers or pre-screeners.*

**Boundary Area (boundaryArea)** text array – a string array of the well-known-text of the work site extent.  The extent is a rectangle that encompasses the excavation sites on the ticket.  If provided, it is represented by a MULTIPOINT WKT string that contains two points: the lower-left (Southwestern) corner of the rectangle, and the upper-right (Northeastern) corner of the rectangle.  This is similar to the bbox property in GeoJson.

**Work Site Area (workSiteArea)** text array – a string array of the the well-known-text of the work site areas.

**Buffered Area (bufferedArea)** text array – a string array of the well-known-text of the buffered work site areas.

## GeoJson

The geoJson property of the ticket contains the geospatial information for the ticket in a GeoJson compatible form ([RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946)).  This is an optional section, which can be null.  The purpose of this section is to allow senders to transmit the geospatial information for a ticket in an open standard format that can be used in receiver's GIS systems.

The geographic information for each ticket is a FeatureCollection containing at least two features.  One feature will be the Excavation Site, and the second would be the Buffered Excavation Site.  These features are differentiated by using a "objectType" property for the feature. Each feature will be identified as an "excavationSite" or a "bufferedSite".

```
"properties": {
    "objectType": "excavationSite"
}
```
OR
```
"properties": {
    "objectType": "bufferedSite"
}
```

If a ticket contains multiple excavation areas, then the ticket would have multiple features identified as "excavationSite"s and "bufferedSite"s.

The features also include an **id** which is a (zero-based) index value to indicate the number of the feature that can be used to uniquely identify that feature on the ticket.  This allows senders and receivers to have a way to have a common unique identifier for features when troubleshooting (for instance).

**Multiple Excavation Site Example**
If there are multiple shapes that define the excavation site such as a polygon for a power station and a point for a nearby power pole, then the features might look like this (focussing on the Id and Properties of the GeoJson, excluding the actual coordinates for clarity):

```
"id": 0,
"properties": {
    "objectType": "excavationSite"
},
  "geometry": {
    "type": "Polygon",
.
.
.
"id": 1,
"properties": {
    "layer": "excavationSite"
},
  "geometry": {
    "objectType": "Point",
.
.
.
"id": 2,
"properties": {
    "layer": "bufferedSite"
},
  "geometry": {
    "objectType": "Polygon",
.
.
.
"id": 3,
"properties": {
    "layer": "bufferedSite"
},
  "geometry": {
    "objectType": "Polygon",
```

Each feature can be referenced using the id, and each feature identifies with the Layer property if it is either the excavation site, or the buffered excavation site.

**Bounding Box**

In addition, the FeatureCollection itself has a bounding box property (**bbox**).  This can be used by the receiver to set a display window that will encompass the entire geometry included in the ticket.

**Notes**
1. GeoJson assumes the coordinate reference system to be WGS84.

2. Coordinate values should be limited to 6 digits (11.1 cm resolution at the equator).  This should handle the resolution that is needed in the 811 industry, and keeping this limited to 6 digits speeds up processing time for recievers.  This is the recommendation from the GeoJson spec (section 11.2 Coordinate Precision).

3. Any buffered geometry should minimize the number of points used in rounding endcaps.  Reducing the number of points simplifies the calculations needed to be performed, so reduced processing time.


# Marking Information

This section contains the information the locator needs to mark facilities at the work site.

```
"markingInformation": {
    "isWhiteLined": false,
    "markingInstructions": "FRONT RIGHT - DAF",
    "remarks": "fdgaewsfa",
    "drivingDirections": ""
  },
```
**Is White Lined (isWhiteLined)** boolean – is the area outlined in white paint or flags? True or False

**Locate Instructions (locateInstructions)** text – Instructions to the locators explaining what to locate.

**Remarks (remarks)** text – Additional remarks about the ticket.

**Driving Directions (drivingDirections)** text – Any driving directions for the locators

# Timeline

This section should contain all the dates for the ticket.

*All dates should be transmitted in ISO 8601 format with time zone information.  For greatest compatibility, use millisecond precision (3 decimal places of precision).  For a discussion on time precision between programming languages, visit (https://nickb.dev/blog/iso8601-and-nanosecond-precision-across-languages/)*.

```
"timeline": {
    "createdOn": "2023-08-08T07:32:05.493-04:00",
    "responseDue": "2023-08-11T00:00:00.000-04:00",
    "legalOn": "2023-08-12T07:00:00.000-04:00",
    "workOn": "2023-08-12T07:00:00.000-04:00",
    "updateBy": "2023-09-05T16:30:00.000-04:00",
    "expiresOn": "2023-09-11T23:59:59.000-04:00"
  },
```
**Created On (createdOn)** datetime – The date and time the ticket was created.

**Response Due (responseDue)** datetime - The date and time responses are due for this ticket.  If no time is included, this is assumed to be be inclusive of the entire date (i.e. a due date of 4/1/2024 has all day 4/1/2024 to respond).

**Legal On (legalOn)** datetime – the date and time the ticket is legal to dig.

**Work On (workOn)** datetime – the date the excavator states that work will begin.

**Update By (updateBy)** datetime – the date and time the ticket needs to be updated.  After this date and time, a new ticket would be needed.

**Expires On (expiresOn)** datetime – the date and time the ticket will expire.

# Project

The project section contains information about the excavation project itself.  What type of work, who the work is being performed for, and whether directional boring or explosives will be used.

```
"project": {
    "workType": "demolition of a building",
    "workDoneFor": "Someone Else",
    "isDirectionalBoring": false,
    "isExplosives": false,
    "isRoadWork": false,
    "projectReference": "Demo of Dollar Central # 1435"
  },
```

**Work Type (workType)** string (300) – the description for the type of work being performed.

**Work Done For (workDoneFor)** string (100) – The name of the person or company that this excavation is being done for. 

**Is Directional Boring (isDirectionalBoring)** boolean – Will directional boring be used?

**Is Explosives (isExplosives)** boolean – Will explosives be used?

**Is Road Work (isRoadWork)** boolean - Does the excavation involve any work on roads?

**Project Reference (projectReference)** string (100) - This can be used  for a project number, job number, permit number, or project name by the excavator.

# Excavator

Information about the excavator, including contact information, is contained in this section.

```
"excavator": {
    "name": "Acme Construction, LLC",
    "excavatorType": "Business",
    "streetAddress": "999 New Street Rd",
    "city": "Duluth",
    "state": "GA",
    "postalCode": "30096",
    "phoneNumber":"",
    "contactList":[
	{
	   "contactType":"Caller",
	   "Name":"Jake Jones",	   
           "Phone": "7705559999",
           "PhoneExtension": "",
	   "EmailAddress":""
	},
	{
	   "contactType":"Field Contact",
	   "Name":"Jake Jones",	   
           "Phone": "7705559999",
           "PhoneExtension": "",
	   "EmailAddress":""
	}
    ]
  },
```

**Name (name)** string (300) – the name of the excavator, or excavator company.

**Excavator Type (excavatorType)** string (100) – The type of excavator, such as Homeowner, Contractor, or Utility Owner.

**Street Address (streetAddress)** string (300)

**City (city)** string (300)

**State (state)** string (2)

**Postal Code (postalCode)** string (5)

**ContactList (contactList)** array of Contacts

**Contact**

- **Contact Type (contactType)** string (40) - Type of excavator contact, such as Caller or Field Contact.

- **Name (name)** string (150) - the full name of the excavator contact.

- **Phone (phone)** string (15) - phone number, digits only - no formatting characters.  15 digits to allow E.164 standard phone numbers.

- **Phone Extension (phoneExtension)** string (10) - Phone extension for the contact.

- **Email (email)** string (255) - Email address for the contact.


# Custom Field Group List

Custom fields are listed in this section.  Custom fields are contained within a group which defines a group name, display name, and then the list of custom fields.

If the group name matches an existing ticket section (such as timeLine or workSite) then that custom field is intended to be displayed in the same section as the other fields in that section by the receiving system.  If it does not match, then the fields in that group are intended to be displayed in a new, supplemental section.  It is intended that the name of that new group would be what is in the Display Name property for that group.

Members can add their own field groups after receiving an Open Ticket.  For instance, if a member has a system that calculates risk on a ticket, that risk information can be added to an open ticket by using a new custom field group that includes the risk score and other information before passing that information along to a traditional ticket management system.  To identify what field groups were sent by the center, there is a "isCenterCreated" flag on each field group. This will be set to true for all field groups sent by the 811 center.  All other field groups should have this set to False.

```
"customFieldGroupList": [
    {
      "groupName": "timeLine",
      "displayName": "Time Line",
      "isCenterCreated": true,
      "fieldList": [
        {
          "fieldName": "responseBy",
          "displayName": "Response By",
          "type": "DateTime",
          "value": "2023-08-19T23:59:59.000-04:00",
          "max": null
        },
        {
          "fieldName": "updateableOn",
          "displayName": "Updateable On",
          "type": "DateTime",
          "value": "2023-09-08T00:00:00.000-04:00",
          "max": null
        }
      ]
    },
    {
      "groupName": "reference",
      "displayName": "Reference",
      "isCenterCreated": true,
      "fieldList": [
        {
          "fieldName": "updateNumber",
          "displayName": "Update Number",
          "type": "integer",
          "value": 2,
          "max": 99
        },
        {
          "fieldName": "riskScore",
          "displayName": "Risk Score",
          "type": "Float",
          "value": 77.6,
          "max": 150
        },
	{
          "fieldName": "HasGasFacilities",
          "displayName": "Has Gas Facilities",
          "type": "Boolean",
          "value": false,
          "max": null
        },
	{
          "fieldName": "projectDescription",
          "displayName": "Project Description",
          "type": "String",
          "value": null,
          "max": 500
        }
      ]
    }
  ],
```

Each group within this section has these properties:

**Group Name (groupName)** string (100) – this is a key value that can be used to identify the group.  This is in camel case to be compatible with automated systems.

**Display Name (displayName)** string (100) – This is a human friendly name for the group that can also be used in the receiving system’s user interface.

**Is Center Created (isCenterCreated)** boolean - True for all field groups that were added by the center.  Any field groups added by other systems should set this to False.

**Field List (fieldList)** – this is an array of custom fields, each with the following properties:

- **Field Name (fieldName)** string (100) – This is a computer-friendly name that can be used as a key value for the receiving system.  This should not contain special characters or spaces.

- **Display Name (displayName)** string (100) – The display name of the custom field as it should appear to anyone viewing the information.

- **Type (type)** string (20) – The type of information this field contains, such as String for text, DateTime for date/time values, Boolean for true/false values, Integer for a whole number value, or Number for a numeric value containing a decimal portion.  See Appendix A for value types.

- **Value (value)** (various types, see Custom Field Data Types below) – This value is used in combination with the Type above to translate the value into an appropriate native representation of that value on the receiving system.  So, an integer can be translated from the value here into a native integer data type variable so that it can be used appropriately.  

- **Max (max)** integer – The maximum number of characters the value can have, or the maximum value for a numeric field.

## Custom Field Data Types

Custom fields use native JSON data types for maximum compatibility with standard parsers:

**String** – JSON string value. Max indicates maximum character length.
**Text** – JSON string value with no length restriction (use sparingly).
**Integer** – JSON number value representing a whole number. Max indicates maximum allowed value.
**Float** – JSON number value that may contain decimal places. Max indicates maximum allowed value.
**Boolean** – JSON boolean value: true, false, or null.
**DateTime** – JSON string value in ISO 8601 format with timezone information.

Null values are represented as JSON null (not the string "null").

# Member List

The members that have facilities in the work site area will be listed in this section, including their names and phone numbers.  This section is a list of member items.  The sample section contains two members:

```
"memberList": [
    {
      "memberId": "ABC120",
      "memberName": "DEMO CITY GAS - ABC120",
      "facilityTypeList": [
	{
            "typeName": "Gas",
            "attributes": {
	    	"color": "yellow",
		"priority": "standard",
		"lineType": "3 inch"
	    }
	}
      ],
      "phoneNumbers": [
        {
          "phone": "9995551234",
          "extension": "",
          "type": "Main"
        }
      ]
    },
    {
      "memberId": "DEMO230",
      "memberName": "CITY OF DEMO WATER – DEMO230",
      "facilityTypeList": [
        {
            "typeName": "Water",
            "attributes": {
	    	"color": "blue",		
		"lineType": "1.5 inch"
	    }
	}
      ],
      "phoneNumbers": [
        {
          "phone": "8885554587",
          "extension": "",
          "type": "Main"
        }
      ]
    }
  ],
```

**Member Id (memberId)** string (10) – The alphanumeric value representing the service area for the member.

**Member Name (memberName)** string (300) – The member’s name.

**Facility Type List (facilityTypeList)** array – This is an array of facility objects containing the facility types this member operates in the service area represented by the member id (above).
  
-  **Type Name (typeName)** string (40) - the name of the facility type, such as Electric, Water, Sewer, Telecom, etc.
  
-  **Attributes (attributes)** name/value pairs - these are name/value pairs allowing the center to add additional information about the facility that may be useful for the excavator.  This is similar to the Foreign Members feature of GeoJSON.  These can be used to describe the APWA color for that facility, or the types of line (diameter, material, etc.), or any other information.

**Phone Numbers (phoneNumbers)** array – this is an array of phone number objects containing phone information for the member.  These objects contain the following information:

- **Phone (phone)** string (15) – The phone number.  15 digits allow E.164 standard phone numbers to be used.

- **Extension (extension)** string (10) – The extension.

- **Type (type)** string (20) – The type of phone number.  For instance, “Main” would be the phone number for most contact types, but many members have a “Damage” number that should be called to report any damages.

# Response History

This section contains the response history for all the members on the ticket, and this will only appear on tickets sent from the center.  This is useful if the ticketing system automatically adds responses in cases (for instance, corrected tickets that add a new member but wants to retain the response history of all others members), or if the ticket is re-sent.

```
"responseHistory": [
    {
      "memberId": "DEMO123",
      "memberName": "DEMO GAS – DEMO123",
      "facilityTypeList": [
        "Gas"
      ],
      "responseList": [
        {
          "responseDate": "2023-08-19T00:10:01.620-04:00",
          "responseCode": "LATE",
          "responseDescription": "Response is late",
          "respondent": "System",
          "note": "Added by System",
	  "attachmentList":[
		  {
		      "name":"Satellite",
		      "mimeType":"image/png",		      
		      "uri":"https://media.tickethost.com/cSqMy1+jAN"
		  },
		  {
		      "name":"Project Plan",
		      "mimeType":"application/pdf",
		      "value":"NTk1OVowgcoxHTAbBgNVBA8MFFByaXZhdGUgT3JnYW5pemF0aW9uMRMwEQYLKwYB"
		  }
            ]
        }
      ]
    },
    {
      "memberId": "ABC999",
      "memberName": "CITY OF DEMO WATER – DEMO999",
      "facilityTypeList": [
        "Water"
      ],
      "responseList": [
        {
          "responseDate": "2023-08-19T00:10:02.167-04:00",
          "responseCode": "LATE",
          "responseDescription": "Response is late",
          "respondent": "System",
          "Note": "Added by System"
        }
      ]
    }
  ],
```

**Member Id (memberId)** string (10) – the alphanumeric value that identifies the members facilities.

**Member Name (memberName)** string (300) – the name of the utility company that this response is for.

**Facility Type List (facilityTypeList)** string (40) array – a string array of the facility types for this member’s response

**Response List (responseList)** – an array of responses, each containing the following fields:

- **Response Date (responseDate)** datetime – the date/time the response was recorded

- **Response Code (responseCode)** string (10) – the name or code of the response.

- **Response Description (responseDescription)** string (300) – the full meaning of the response.

- **Respondent (respondent)** string (255) – the username or other identifier for the responder.

- **Note (note)** string (1000) – any comments or notes added by the respondent.
   
- **AttachmentList (attachmentList)** - array of attachment objects that are attached to this response.  Attachment objects have the following properties:
   
- - **Name (name)** string (200) - a descriptive name describing the attachment.

- - **Mime Type (mimeType)** string (255) - the mime type for the binary information stored in the value field (below).  Restrictions on mime types should be consistent with the center's positive response attachment policy.
   
- - **Value (value)** text - Base64 encoded binary value.  Restrictions on size should be consistent with the center's positive response attachment policy.
 
- - **URI (uri)** text - URL to the "attached" document.  The referenced resource should be available for the life of the ticket (until the expiresOn date).  The reference should be accessible to all receivers of the ticket.

*Note: The attachment can either have a binary value for transmissions to include the actual document OR the URL for a link to the resource.  Each attachment should have either Value or URI, but not both.*

# AttachmentList (Attachments to tickets)

Attachments to tickets can be URI links or Base64 encoded binary values contained in a simple object collection.  If a document is directly attached, then the base64 encoded binary information should be in the "value" property.  If the document is a URI reference, the value should be in the "uri" property instead.

```
"attachmentList":[
  {
      "name":"Satellite",
      "mimeType":"image/png",
      "uri":"https://media.tickethost.com/cSqMy1+jAN"
  },
  {
      "name":"Project Plan",
      "mimeType":"application/pdf",
      "value":"NTk1OVowgcoxHTAbBgNVBA8MFFByaXZhdGUgT3JnYW5pemF0aW9uMRMwEQYLKwYB"
  }
]
```

**Name (name)** string (200) - a descriptive name describing the attachment.

**Mime Type (mimeType)** string (255) - the mime type for the binary information stored in the value field (below).  This is needed so the receiver can correctly interpret the binary data.

**Value (value)** text - Base64 encoded binary value.  Limits should be set by the center on the maximum size that should be sent, and receivers should be prepared to receive attachments of that size.  For centers that receive tickets with attachments, size limits should be communicated and enforced to ensure acceptable service levels for all senders and receivers.

**URI (uri)** text - URL to the "attached" document.  The referenced resource should be available for the life of the ticket (until the expiresOn date).  The reference should be accessible to all receivers of the ticket.
