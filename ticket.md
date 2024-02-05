# Header
The first section of the JSON format is the header object.   The purpose of the header is to identify the ticket and other data regarding the transmission.  The inspiration for this section is that of an IP header that contains information about the delivery and routing of an IP packet.  In this case, it is about the delivery and potential routing of a ticket.

```
"header": {
    "formatVersion": "0.3",
    "ticketNumber": "230815-001003",
    "sequence": 12,
    "source": "Voice",
    "type": "Normal",
    "standardType": "Normal",
    "priority": "Normal ",
    "status": "New",
    "isTest": false,
    "centerName": "GA811",
    "wktSrid": 4326,
    "memberIds": [
      "GAUPC"
    ]
  },
```

**Format Version (formatVersion)** – This is the version of the Open Ticket Format specification this JSON document represents.  When new versions of the specification are developed, this will allow receivers to know what version of the specification the transmission follows.  For centers not yet implementing newer versions, they can continue to send, and receivers can continue to receive for that version of the spec.  Once a center or receiver has upgraded their system to the latest spec, then they can coordinate the changeover to the new format.

It is possible that ticketing systems need to add a ticket version to this header.  If so, naming this FormatVersion allows us to separate a version of a ticket from the delivery format version.

**Ticket Number (ticketNumber)** – this is the ticket number.

**Sequence (sequence)** – this is a serial number of this transmission for this receiver.   This allows the receiver to know what order tickets were sent to them in a day.  Since this format can be used for transmissions of tickets as well as retrieval of tickets by API call, this will only have a value if this is a transmission.
[This is very much like a session number for networking.  This identifies the order the ticket was transmitted each day, as if each ticket were a packet sent in a daily session]

**Source (source)** – this is the channel (Call Center, Web Portal, etc.) the ticket was submitted through.

**Type (type)** – the legal ticket type for this ticket.

**Standard Type (standardType)** – the standard ticket type for this ticket.  This should be mapped to a national standard.  For instance, “Design Request” might be the legal ticket type defined by a state law, but “Design” would be an appropriate “standard” type.  Each center should define their ticket type mappings to a national standard.

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

**Priority (priority)** – This helps the receiver determine how fast this ticket needs to be processed.  This can be one of these values:
	Normal – This should be processed after Rush or Emergency tickets
  Rush – This should be processed before Normal priority tickets, but after emergency priority tickets
	Emergency – This should be processed before other priority types.

**Status (status)** – The current status of this ticket, New, Resend, Cancel, or other status supported by the center.
- New – this is the first send of this ticket
- Resend – this is a resend of a previous ticket
- Cancel – this is a cancellation of a ticket

**IsTest (isTest)** – If implemented by the center, this would allow them to send test tickets to members and have them identify those tickets as test tickets.  Although most testing is done using test systems, there is still a need for test tickets to be sent through production systems.  This would allow the identification of test tickets that do not require an actual locate to be performed.

If this is not implemented by the center’s system, then this flag should indicate Null.  This will tell the receiver that the sending system does not implement this feature, rather than False which would indicate the center positively identifying this as a production ticket.

**Center Name (centerName)** – an identifier for the center transmitting the ticket.  This is most often the state abbreviation plus “811”.

**WKT SRID (wktSrid)** – This is the Spatial Reference Identifier of the coordinate system used for WKT values.  The best practice is for this to always be 4326 (WGS84). 

**Member Ids (memberIds)** – this is an array of string values containing the alphanumeric member identifiers for the facilities included in this transmission.  This allows a single ticket to be delivered for multiple facilities for a single operator.  For instance, a member who operates water and sewer facilities may receive a single ticket that includes the codes for the water and sewer facilities they operate, so this would contain both member ids corresponding to those facilities.

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
    "intersection": "Highway 25",
    "longitude": -84.5482687468752,
    "latitude": 33.8551704136117,
    "secondaryLongitude": -84.545451253125,
    "secondaryLatitude": 33.8575195862275,
    "boundaryWkt": null,
    "workSiteWkt": "POLYGON((-84.54761 33.85697,-84.54761 33.85572,…))",
    "bufferedWkt": "POLYGON((-84.547642 33.855171,…))”    
  },
```

**Country Code (countryCode)** – specifying a country code allows this specification to be used in countries other than the United States.  This should be the ISO Alpha-2 code for the country, such as US for the United States, or CA for Canada.

**State/Province (stateProvince)** – a two-character abbreviation for the state or province.

**County (county)** – the name of the county for the work site.

**Place (place)** – the city, town, or unincorporated place name.

**Full Address (fullAddress)** – the full street address for the work site.

**Address Number (addressNumber)** – number part of the address.  This is a string, not a number, to accommodate alphabetic characters (‘34B’ for instance).

**Address Prefix (addressPrefix)** – A prefix to the street name, normally directional (North, East, West, etc.)

**Address Street Name (addressStreetName)** – The street name.

**Address Street Type (addressStreetType)** – The type of street (Rd, St, Blvd, Hwy, etc.)

**Address Suffix (addressSuffix)** – A suffix, normally directional (North, East, West, etc.)

**Intersection (intersection)** – the name of the nearest intersecting street.

The following 4 fields provide the upper-left and lower-right points of a rectangle that encloses the work site extent.  This rectangle could be created as a polygon in Well-Known-Text (WKT) and sent as the boundaryWkt below instead.  This provides two methods of sending the extent information.

**Longitude (longitude)** – The longitude of the left-most point of the work site.

**Latitude (latitude)** – The latitude of the upper-most point of the work site.

**SecondaryLongitude (secondaryLongitude)** – the longitude of the right-most point of the work site.

**SecondaryLatitude (secondaryLatitude)** – the latitude of the bottom-most point of the work site.

*NOTE: Well Known Text Fields - The following fields contain Well Known Text (WKT) values.  All WKT values are expressed in the WGS84 (SRID 4326) and have a precision of 6 digits (11.1 cm resolution at the equator).*

*Additionally, any buffered geometry should minimize the number of points used in rounding endcaps.  Reducing the number of points and keeping the precision to 6 digits can speed up processing significantly, which is particularly important for high-volume receivers or pre-screeners.*

**Boundary WKT (boundaryWkt)** – the well-known-text of the work site extent.  

**Work Site WKT (workSiteWkt)** – the well-known-text of the work site area.

**Buffered WKT (bufferedWkt)** – the well-known-text of the buffered work site area.

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
**Is White Lined (isWhiteLined)** – is the area outlined in white paint or flags? True or False

**Locate Instructions (locateInstructions)** – Instructions to the locators explaining what to locate.

**Remarks (remarks)** – Additional remarks about the ticket.

**Driving Directions (drivingDirections)** – Any driving directions for the locators

# Timeline

This section should contain all the dates for the ticket.

*All dates should be transmitted in ISO 8601 format with time zone information.  For greatest compatibility, use millisecond precision (3 decimal places of precision).*

https://nickb.dev/blog/iso8601-and-nanosecond-precision-across-languages/

``
"timeLine": {
    "createdOn": "2023-08-08T07:32:05.493-04:00",
    "legalOn": "2023-08-12T07:00:00.000-04:00",
    "workOn": "2023-08-12T07:00:00.000-04:00",
    "updateBy": "2023-09-05T16:30:00.000-04:00",
    "expiresOn": "2023-09-11T23:59:59.000-04:00"
  },

``
**Created On (createdOn)** – The date and time the ticket was created.

**Legal On (legalOn)** – the date and time the ticket is legal to dig.

**Work On (workOn)** – the date the excavator states that work will begin.

**Update By (updateBy)** – the date and time the ticket needs to be updated.  After this date and time, a new ticket would be needed.

**Expires On (expiresOn)** – the date and time the ticket will expire.


