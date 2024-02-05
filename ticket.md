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
[We need to insert a reference to some standardized ticket type names, ideally maintained by the FNCA.  It would be up to individual centers to then map their ticket types to the standard types.  And, there should always be an “Other” ticket type for those that cannot be mapped to a standard type.]

**Priority (priority)** – This helps the receiver determine how fast this ticket needs to be processed.  This can be one of these values:
	Normal – This should be processed after Rush or Emergency tickets
  Rush – This should be processed before Normal priority tickets, but after emergency priority tickets
	Emergency – This should be processed before other priority types.

**Status (status)** – The current status of this ticket, New, Resend, Cancel, or other status supported by the center.
	New – this is the first send of this ticket
	Resend – this is a resend of a previous ticket
	Cancel – this is a cancellation of a ticket

**IsTest (isTest)** – If implemented by the center, this would allow them to send test tickets to members and have them identify those tickets as test tickets.  Although most testing is done using test systems, there is still a need for test tickets to be sent through production systems.  This would allow the identification of test tickets that do not require an actual locate to be performed.

If this is not implemented by the center’s system, then this flag should indicate Null.  This will tell the receiver that the sending system does not implement this feature, rather than False which would indicate the center positively identifying this as a production ticket.

**Center Name (centerName)** – an identifier for the center transmitting the ticket.  This is most often the state abbreviation plus “811”.

**WKT SRID (wktSrid)** – This is the Spatial Reference Identifier of the coordinate system used for WKT values.  The best practice is for this to always be 4326 (WGS84). 

**Member Ids (memberIds)** – this is an array of string values containing the alphanumeric member identifiers for the facilities included in this transmission.  This allows a single ticket to be delivered for multiple facilities for a single operator.  For instance, a member who operates water and sewer facilities may receive a single ticket that includes the codes for the water and sewer facilities they operate, so this would contain both member ids corresponding to those facilities.

Since this format can be used for transmissions of tickets as well as retrieval of tickets by API call, this will only have a value if this is a transmission.
