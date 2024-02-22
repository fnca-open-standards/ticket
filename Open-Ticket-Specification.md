# Open Ticket Format

The open ticket format is described here.  

Fields that are common to all tickets are standardized and placed in sections for organization.  These are the "Core" fields for tickets.

Fields that are custom to a center are contained in the custom field groups section after all the core ticket sections.

Each section includes a description of what that section contains, followed by a sample in JSON format.  Finally, each field in the section is described along with its data type and maximum length (for strings).


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

**Geography** - all geographic information for the ticket.

**Locate Information** – Locate instructions, remarks, and driving directions.  These are needed by the locator to properly locate the facilities.

**Project** – Additional information about the excavation project.  This is not required for the location, but is useful in damage prevention, such as whether or not explosives will be used, or the expected duration of the excavation.

**Timeline** – Dates commonly associated with tickets.  Dates specific to a ticket type should be in Custom Fields instead.

**Excavator** – information about the excavator.

**Custom Fields** – custom fields with their metadata for this specific ticket type.

**Members** – The utility companies that are being notified to locate facilities.

**Response History** – This section shows the response history for this ticket.  Initially, this may be blank.  Any resends of the ticket would contain the response history as of the resend date and time.


# Header

The first section of the JSON format is the header object.   The purpose of the header is to identify the ticket and other data regarding the transmission.  The inspiration for this section is that of an IP header that contains information about the delivery and routing of an IP packet.  In this case, it is about the delivery and potential routing of a ticket.

```
"header": {
    "formatVersion": "0.5",
    "ticketNumber": "230815-001003",
    "sequence": 12,
    "source": "Voice",
    "type": "Normal",
    "standardType": "Normal",
    "priority": "Normal ",
    "status": "New",
    "isTest": false,
    "centerName": "GA811",
    "memberIds": [
      "GAUPC"
    ]
  },
```

**Format Version (formatVersion)** string (10) – This is the version of the Open Ticket Format specification this JSON document represents.  When new versions of the specification are developed, this will allow receivers to know what version of the specification the transmission follows.  For centers not yet implementing newer versions, they can continue to send, and receivers can continue to receive for that version of the spec.  Once a center or receiver has upgraded their system to the latest spec, then they can coordinate the changeover to the new format.

It is possible that ticketing systems need to add a ticket version to this header.  If so, naming this FormatVersion allows us to separate a version of a ticket from the delivery format version.

**Ticket Number (ticketNumber)** string (25) – this is the ticket number.

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

**Priority (priority)** string (10) – This helps the receiver determine how fast this ticket needs to be processed.  This can be one of these values:
	Normal – This should be processed after Rush or Emergency tickets
  Rush – This should be processed before Normal priority tickets, but after emergency priority tickets
	Emergency – This should be processed before other priority types.

**Status (status)** string (10) - The current status of this ticket, New, Resend, Cancel, or other status supported by the center.
- New – this is the first send of this ticket
- Resend – this is a resend of a previous ticket
- Cancel – this is a cancellation of a ticket

**IsTest (isTest)** boolean – If implemented by the center, this would allow them to send test tickets to members and have them identify those tickets as test tickets.  Although most testing is done using test systems, there is still a need for test tickets to be sent through production systems.  This would allow the identification of test tickets that do not require an actual locate to be performed.

If this is not implemented by the center’s system, then this flag should indicate Null.  This will tell the receiver that the sending system does not implement this feature, rather than False which would indicate the center positively identifying this as a production ticket.

**Center Name (centerName)** string (25) – an identifier for the center transmitting the ticket.  This is most often the state abbreviation plus “811”.

**Member Ids (memberIds)** string (10) array – this is an array of string values containing the alphanumeric member identifiers for the facilities included in this transmission.  This allows a single ticket to be delivered for multiple facilities for a single operator.  For instance, a member who operates water and sewer facilities may receive a single ticket that includes the codes for the water and sewer facilities they operate, so this would contain both member ids corresponding to those facilities.

Since this format can be used for transmissions of tickets as well as retrieval of tickets by API call, this will only have a value if this is a transmission.

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

# Geography

All geographic information is available in this section

```
"geography":{
    "srid": 4326,
    "longitude": -84.5482687468752,
    "latitude": 33.8551704136117,
    "secondaryLongitude": -84.545451253125,
    "secondaryLatitude": 33.8575195862275,
    "boundaryArea": null,
    "workSiteArea": ["POLYGON((-84.54761 33.85697,-84.54761 33.85572,…))",
                    "POLYGON((…))"],
    "bufferedArea": ["POLYGON((-84.547642 33.855171,…))"]    
},
```

**SRID (srid)** integer – This is the Spatial Reference Identifier of the coordinate system used for WKT values.  The best practice is for this to always be 4326 (WGS84). 

The following 4 fields provide the upper-left and lower-right points of a rectangle that encloses the work site extent.  This rectangle could be created as a polygon in Well-Known-Text (WKT) and sent as the boundaryArea below instead.  This provides two methods of sending the extent information.

**Longitude (longitude)** float – The longitude of the left-most point of the work site.

**Latitude (latitude)** float – The latitude of the upper-most point of the work site.

**Secondary Longitude (secondaryLongitude)** float – the longitude of the right-most point of the work site.

**Secondary Latitude (secondaryLatitude)** float – the latitude of the bottom-most point of the work site.

*NOTE: Well Known Text Fields - The following fields contain Well Known Text (WKT) values.  All WKT values are expressed in the WGS84 (SRID 4326) and have a precision of 6 digits (11.1 cm resolution at the equator).*

*Additionally, any buffered geometry should minimize the number of points used in rounding endcaps.  Reducing the number of points and keeping the precision to 6 digits can speed up processing significantly, which is particularly important for high-volume receivers or pre-screeners.*

**Boundary Area (boundaryArea)** text array – a string array of the well-known-text areas of the work site extent.  

**Work Site Area (workSiteArea)** text array – a string array of the the well-known-text of the work site areas.

**Buffered Area (bufferedArea)** text array – a string array of the well-known-text of the buffered work site areas.

# Locate Information

This section contains the information the locator needs to mark facilities at the work site.

```
"locateInformation": {
    "isWhiteLined": false,
    "locateInstructions": "FRONT RIGHT - DAF",
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
"timeLine": {
    "createdOn": "2023-08-08T07:32:05.493-04:00",
    "legalOn": "2023-08-12T07:00:00.000-04:00",
    "workOn": "2023-08-12T07:00:00.000-04:00",
    "updateBy": "2023-09-05T16:30:00.000-04:00",
    "expiresOn": "2023-09-11T23:59:59.000-04:00"
  },
```
**Created On (createdOn)** datetime – The date and time the ticket was created.

**Legal On (legalOn)** datetime – the date and time the ticket is legal to dig.

**Work On (workOn)** datetime – the date the excavator states that work will begin.

**Update By (updateBy)** datetime – the date and time the ticket needs to be updated.  After this date and time, a new ticket would be needed.

**Expires On (expiresOn)** datetime – the date and time the ticket will expire.

# Project

The project section contains information about the excavation project itself.  What type of work, who the work is being performed for, and whether directional boring or explosives will be used.

```
"project": {
    "workType": "demolition of a building",
    "duration": "6 days",
    "workDoneFor": "Someone Else",
    "isDirectionalBoring": false,
    "isExplosives": false
  },
```

**Work Type (workType)** string (300) – the description for the type of work being performed.

**Duration (duration)** string (50) – The duration of the project.

**Work Done For (workDoneFor)** string (100) – The name of the person or company that this excavation is being done for. 

**Is Directional Boring (isDirectionalBoring)** boolean – Will directional boring be used?

**Is Explosives (isExplosives)** boolean – Will explosives be used?

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
    "callerName": "Jake Jones",
    "callerPhone": "7705559999",
    "callerPhoneExtension": "",
    "callerEmail": "",
    "contactName": "Jake Jones",
    "contactPhone": "7705559999",
    "contactPhoneExtension": ""
  },
```

**Name (name)** string (300) – the name of the excavator, or excavator company.

**Excavator Type (excavatorType)** string (100) – The type of excavator, such as Homeowner, Contractor, or Utility Owner.

**Street Address (streetAddress)** string (300)

**City (city)** string (300)

**State (state)** string (2)

**Postal Code (postalCode)** string (5)

**Caller Name (callerName)** string (150)

**Caller Phone (callerPhone)** string (10)

**Caller Phone Extension (callerPhoneExtension)** string (10)

**Caller Email (callerEmail)** string (255)

**Contact Name (contactName)** string (150)

**Contact Phone (contactPhone)** string (10)

**Contact Phone Extension (contactPhoneExtension)** string (10)

**Contact Email (contactEmail)** string (255)

# Custom Field Groups

Custom fields are listed in this section.  Custom fields are contained within a group which defines a group name, display name, and then the list of custom fields.

If the group name matches an existing ticket section (such as timeLine or workSite) then that custom field is intended to be displayed in the same section as the other fields in that section by the receiving system.  If it does not match, then the fields in that group are intended to be displayed in a new, supplemental section.  It is intended that the name of that new group would be what is in the Display Name property for that group.

```
"customFieldGroups": [
    {
      "groupName": "timeLine",
      "displayName": "Time Line",
      "fields": [
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
      "fields": [
        {
          "fieldName": "previousTicketNumber",
          "displayName": "Previous Ticket Number",
          "type": "String",
          "value": null,
          "max": 50
        },
        {
          "fieldName": "originalTicketNumber",
          "displayName": "Original Ticket Number",
          "type": "String",
          "value": "230815-001002",
          "max": 50
        }
      ]
    }
  ]
```

Each group within this section has these properties:

**Group Name (groupName)** string (100) – this is a key value that can be used to identify the group.  This is in camel case to be compatible with automated systems.

**Display Name (displayName)** string (100) – This is a human friendly name for the group that can also be used in the receiving system’s user interface.

**Fields (fields)** – this is an array of custom fields, each with the following properties:

   **Field Name (fieldName)** string (100) – This is a computer-friendly name that can be used as a key value for the receiving system.  This should not contain special characters or spaces.

   **Display Name (displayName)** string (100) – The display name of the custom field as it should appear to anyone viewing the information.

   **Type (type)** string (20) – The type of information this field contains, such as String for text, DateTime for date/time values, Boolean for true/false values, Integer for a whole number value, or Number for a numeric value containing a decimal portion.  See Appendix A for value types.

   **Value (value)** string (4000) – This is a textual representation of the value.  This can be used in combination with the Type above to translate the value into an appropriate native representation of that value on the receiving system.  So, an integer can be translated from the text value here into a native integer data type variable so that it can be used appropriately.  If that custom field does not have a value, this attribute will be “null”.

   **Max (max)** integer – The maximum number of characters the value can have, or the maximum value for a numeric field.

# Members

The members that have facilities in the work site area will be listed in this section, including their names and phone numbers.  This section is a list of member items.  The sample section contains two members:

```
"members": [
    {
      "memberId": "ABC120",
      "memberName": "DEMO CITY GAS - ABC120",
      "facilityTypes": [
        "Gas"
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
      "facilityTypes": [
        "Water"
      ],
      "phoneNumbers": [
        {
          "phone": "8885554587",
          "extension": "",
          "type": "Main"
        }
      ]
    }
  ]
```

**Member Id (memberId)** string (10) – The alphanumeric value representing the service area for the member.

**Member Name (memberName)** string (300) – The member’s name.

**Facility Types (facilityTypes)** string (40) array – This is an array of strings containing the facility types this member operates in the service area represented by the member id (above).

**Phone Numbers (phoneNumbers)** array – this is an array of phone number objects containing phone information for the member.  These objects contain the following information:

   **Phone (phone)** string (10) – The phone number.

   **Extension (extension)** string (10) – The extension.

   **Type (type)** string (20) – The type of phone number.  For instance, “Main” would be the phone number for most contact types, but many members have a “Damage” number that should be called to report any damages.

# Response History

This section contains the response history for all the members on the ticket.  This is useful if the ticketing system automatically adds responses in cases (for instance, corrected tickets that add a new member but wants to retain the response history of all others members), or if the ticket is re-sent.

```
"responseHistory": [
    {
      "memberId": "DEMO123",
      "memberName": "DEMO GAS – DEMO123",
      "facilities": [
        "Gas"
      ],
      "responseEntries": [
        {
          "responseDate": "2023-08-19T00:10:01.620-04:00",
          "responseCode": "LATE",
          "responseDescription": "Response is late",
          "respondent": "System",
          "comment": "Added by System"
        }
      ]
    },
    {
      "memberId": "ABC999",
      "memberName": "CITY OF DEMO WATER – DEMO999",
      "facilities": [
        "Water"
      ],
      "responseEntries": [
        {
          "responseDate": "2023-08-19T00:10:02.167-04:00",
          "responseCode": "LATE",
          "responseDescription": "Response is late",
          "respondent": "System",
          "comment": "Added by System"
        }
      ]
    }
  ]
```

**Member Id (memberId)** string (10) – the alphanumeric value that identifies the members facilities.

**Member Name (memberName)** string (300) – the name of the utility company that this response is for.

**Facilities (facilities)** string (40) array – a string array of the facility types for this member’s response

**Response Entries (responseEntries)** – an array of responses, each containing the following fields:

   **Response Date (responseDate)** datetime – the date/time the response was recorded

   **Response Code (responseCode)** string (10) – the name or code of the response.

   **Response Description (responseDescription)** string (300) – the full meaning of the response.

   **Respondent (respondent)** string (255) – the username or other identifier for the responder.

   **Comment (comment)** string (1000) – any comments added by the respondent.

# Attachments

Attachments to tickets are Base64 encoded binary values contained in a simple object collection where the object has a name and a value.

```
"attachments":[
  {
      "name":"Satellite",
      "value":"MIIHNjCCBh6gAwIBAgIQCVe4E0h49mzI0NcSqMy1+jANBgkqhkiG9w0BAQsFADB1"
  },
  {
      "name":"Project Plan",
      "value":"NTk1OVowgcoxHTAbBgNVBA8MFFByaXZhdGUgT3JnYW5pemF0aW9uMRMwEQYLKwYB"
  }
]
```

**Name (name)** string (200) - a descriptive name describing the attachment.

**Value (value)** text - Base64 encoded binary value.