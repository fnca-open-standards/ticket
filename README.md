# Open Ticket Standard
This document describes a Open Ticket Format that can be used to send any ticket from an 811 center any receiver.

This is an open standard that is freely available for adoption.  This means that it is not specific to any vendor and can be implemented by anyone wanting to send or receive 811 excavation tickets.  It does require centers and receivers to agree to the format, and to develop and maintain this format for widest adoption.

In turn, every center that adopts this standard eliminates effort on behalf of their members and ticket management vendors that currently must maintain specific formats for every center.  It also allows 3rd party systems (such as business analytics or facility line visualization tools) to integrate much quicker into 811 workflows.  As more of these preprocessing and postprocessing systems are developed to improve excavation safety, a standard open format will eliminate center-specific work and allow system developers to focus on the features of their systems instead of keeping up with ticket format changes.

This in turn significantly reduces the time spent by each center setting up and testing ticket delivery with members and vendors.

## How the Open Ticket Works
The Open Ticket standard is a way to encode 811 tickets similarly to how other data is encoded and transmitted, like web pages.  A web page is marked up with HTML tags that is understandable by any browser (with compatible versioning, which this format uses as well).  
To implement this, we need a method to transmit both the common ticket information that exists between all 811 centers (such as Ticket Number, Excavator Name, work site information, etc.), as well as a method to package custom information for each ticket type and each state in such a way that the receiver can properly decode or parse the information.  We can do this by adding metadata (data about the data) to each custom field we transmit that indicates the name, type, value, grouping, and other attributes of each custom field, and transmitting that as a block of custom fields at the end of the common ticket information.  

Here is a JSON snippet of what this looks like:
```
"customFieldGroups": [    
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
We then use existing best-practice technologies to handle the transmission of this information: HTTP and JSON through WebAPI.  This bypasses the weaknesses of email like anti-spam blacklists and junk mail detection in email clients that can effectively block delivery of requests.

By involving 811 centers, members, and ticket management vendors and doing the work of defining what data is common among all 811 centers, and agreeing on a common definition of terms, we can also create out-of-the-box text and html email formats that would be useful for smaller members that do not need a ticket management system.  These smaller members would benefit from a common format as well.  This would help members like rural EMC’s close to state borders with customers on both sides of the state line.



## The Open Ticket Standard Supports New Ticket Types and Changes to Tickets

With custom formats for every 811 center and ticket type, anytime a change is made (like a new ticket type or adding a new field), this needs to be coordinated with their members and TMS vendors.  The members and TMS vendors need to update their systems to accommodate the new changes, and test with the center before going live.  With the Open Ticket Standard, this is unnecessary.  The format defined here can encode the new ticket type or updated fields and deliver it without any software changes for the sender or receiver.  

This eliminates MOST of the work in ticket changes.  The only changes that may be necessary would be back-end rules that need to be implemented.  For instance, if an 811 center implements Cancel tickets, the TMS vendor would need to know that when they receive a cancel ticket that they would need to change the status of the “cancelled” ticket so that the original ticket would not need to be marked.  These “business rule” changes are not contained in the format and would need to be communicated and coordinated.

## Versioning
The standard is also versioned.  This will allow updates to the specification in the future.  So, when an 811 center updates their system to support a newer version they can start sending tickets in the newer format.  Similarly, receivers can notify the 811 centers when they can receive tickets in the updated format and can switch over to it at that time.  Anyone needing more time can still receive tickets in the older format.

##
