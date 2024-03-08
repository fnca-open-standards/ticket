# Open Ticket Format - PA811 Initial Samples

A number of PA811 tickets were mapped and permutations of PA811 ticket types are included. The initial goal was to simply determine inital custom field groups for additional fields. 

# Standard Section Extensions

Custom field groups were created for each standard section of the Open Ticket Format, with the exception of Timeline for now. All are prefixed with "pa811."

# Header/pa811Header

PA811 tickets were mapped into the standard section Header. The custom field group also includes PA811 primary fields used for ticket classification.

** version ** - PA811 tickets are versioned. Versions apply to specific tickets types, such as CANCEL and RENOTIFY

** actionType ** - CANCEL, NEW, RENOTIFY, UPDATE.

** requestType ** - CROSS BORE, DAMAGE, DEMOLITION, EXCAVATION, NO ONE CALL, ODOR OF GAS.

** requstClass ** - COMPLEX PROJECT, EMERGENCY, FINAL DESIGN, INSUFFICIENT, PRELIMINARY DESIGN, ROUTINE.

** outputFormatVersion ** - Our output formats are versioned, and we would like to inlcude our parsable/human readable output (pa811AdditionalTicketInformation.).

# Additional Custom Field Groups

Some additional custom field groups were created and are prefixed with "pa811."

** pa811AddtitionalTicketInformation** - Fields applying to the entire ticket, or for data relevant to specific types. This could change depending on how granular the usage of custom field groups becomes.

** pa811NoOneCall ** - Data specific to a NO ONE CALL ticket type.

** pa811Reference ** - From the suggested reference custom field group.

** pa811Renotify ** - Data specific to a RENOTIFY.

