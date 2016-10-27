# BusinessOS Core Schema
This document lays the schema for the core data entities within BusinessOS.  It is oriented for Firebase clients and does not represent the schema in a traditional relational manner.



### Example JSON
A JSON file containing an example of this schema in action can be [found here](example.json).

### Naming Conventions
All fields start with a lowercase letter and should be camelCased instead of using-dasherization or having a FirstCapitalLetter.

### Default Properties
Unless otherwise noted, all core schema objects have:
- `id` (string, required) - The ID of the object.  _This can be omitted when the type is not a global entity, as noted under "Firebase Path" under the type description.  Additionally, oftentime the type does not have an explicit ID as a child property, but instead the key for a collection is its ID, e.g. the [Appointment][] under `/global/appointments/apptId1` has an `id` of `apptId1`, even though it doesn't have a child property `id`._

Unless otherwise noted, any object that has properties also has:
- `createdAt` ([Timestamp][], required) - When this object was created.  _It is the responsibility of the application creating the object to ensure this is set._
- `lastModifiedAt` ([Timestamp][], required) - When this object was last modified.  This is only required to be updated when a direct-child property of the object (e.g. a sibling property of this property) is modified.  _It is the responsibility of the application creating the object to ensure this is set._

### ObjectReferences
This is a many relationship reference to another object in the system.  It looks like `{entityId: true}`, and is the traditional way to keep a many relationship in Firebase.  For example, if we were referencing parts with IDs of `123` and `456`, the references would look like this:
```
{
  "123": true,
  "456": true
}
```

### Legacy Data and Mappings
Legacy-formatted data from Larson's system, prior to transformation into the BusinessOS schema, is stored in Firebase under the `/legacy` path.  Additionally, mappings between Larson's ID system and that of BusinessOS are stored for each entity under `/legacy-index`.  Each of these paths are listed below for relevant entities.

### Two-Way Data Bindings
🚨 serve to flag one half of a two-way relationship. As we are working within Firebase, we are not dealing with a traditional relational database. Instead, to ease the caching of data within applications and across multiple core schema objects, we have denormalized some of the data. For example, [Appointment][] has a one to many relationship with [Issue][]. Each [Appointment][] contains [ObjectReferences][] of [Issue][] while each [Issue][] has an `appointmentId` child property which contains the ID of the associated [Appointment][].

It is important when manipulating any of the fields involved in such relationships to remember to modify the fields of the associated objects at the other end of the relationship. In our example above, when we modify any of the [ObjectReferences][] on [Appointment][] we must also update the corresponding property, `appointmentId` in this case, on the [Issue][] object. 🚨 are used within this document to flag such fields in an effort to reduce user error.



## Address
This describes a physical location.  It's an address, silly!  _Note: This object does not have `createdAt` or `lastModifiedAt` [Timestamp][]s._

### Firebase Path
Not a global entity.

### Properties
- `street` (string, required) - The street address.
- `street2` (string, optional) - The optional second line of the address.
- `city` (string, required) - The city of the address.
- `state` (string, required) - The 2-letter abbreviation for the state of the address (US only).
- `zip` ([ZipCode][], required) - The zip code of the address.
- `coordinate` ([Coordinate][], optional) - The GPS location of the address.


## Appointment
This describes an appointment that an [Engineer][] has at a [Site][] to fix an [Issue][].

### Firebase Path
`/global/appointments/{id}`

### Legacy Path
`/legacy/appointments/{id}`

### Legacy Mapping
`/legacy-index/appointments/{Service_Call_Id}/{Appointment}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` appointment

### Properties
- `appointmentNumber` (number, optional) - The sequential number of this appointment within the [Job][], starting at `1`.
- `arrivedAt` ([Timestamp][], optional) - Time that the assigned [Engineer][] arrived on location.
- `assignedAt` ([Timestamp][], optional) - Time that an [Engineer][] was assigned by a [Dispatcher][] or in the case of a reassignment, the time that the [Appointment][] was assigned to a new [Engineer][].
- `assignedEngineer` (string, optional) - ID of the [Engineer][] assigned to complete this [Appointment][].🚨
- `callerName` (string) - Name of the person requesting the [Appointment][].
- `callerPhoneNumber` (string) - [PhoneNumber][] of the person requesting the [Appointment][].
- `completedAt` ([Timestamp][], optional) - Time that the work on this appointment ended.
- `createdBy` (string, optional) - ID of the [Dispatcher][] that created this appointment.
- `departedAt` ([Timestamp][], optional) - Time that the assigned [Engineer][] left the [Site][] of the appointment.
- `engineerRequestsReassignment` (boolean, optional, default = `false`) - Whether or not the `assignedEngineer` requests that this appointment be reassigned to someone else.
- `estimatedArrivalTime` ([Timestamp][], optional) - Approximate time that the `assignedEngineer` is expected to arrive at the [Site][].
- `issues` ([ObjectReferences][] of [Issue][], optional) - The issues related to the reason for this appointment. 🚨
- `jobId` (string, required) - ID of the [Job][] this appointment is associated with. 🚨
- `officeId` (string, required) - ID of the [Office][] this appointment is associated with.
- `priority` (enum, optional) - Priority level for the appointment.  _Note: in Larson's systems, the `priority` is stored on the Service Call, not the Appointment._  Possible values:
- `high`
- `normal`
- `questions` (array[[AppointmentQuestion][]], optional) - Questions that have been submitted by the client regarding a specific appointment.
- `recommendedEngineers` ([ObjectReferences][] of [Engineer][] ID to [Recommendation][], optional) - Engineers that the system recommends to work on this appointment.
- `serviceDescription` (string, optional) - Description of the work to be done during this appointment.
- `startedAt` ([Timestamp][], optional) - Time that the work on this appointment began.
- `status` ([AppointmentStatus][], required) - Current status of the appointment (enum values from Larson - `Appointment_Status`).
- `partRequests` ([ObjectReferences][] of [PartRequest][], optional) - Part Requests raised on the appointment. 🚨
- `type` (enum, required) - Type of the appointment.  Possible values:
- TODO: enumerate values

### Larson-Specific Properties
- `legacyId` (string, optional) - `[Service_Call_ID]-[Appointment]` of the entity in Larson's systems.



## AppointmentQuestion

### Firebase Path
Not a global entity.

### Properties
- `requesterName` (string, required) - Name of the person asking the question.
- `requesterEmail` (string, required) - Email address of the person asking the question.
- `question` (string, required) - Text of the question that was asked.
- `answer` (string, optional) - Text of the answer/reply to the question.
- `answererId` (string, optional) - ID of the [Dispatcher][] that answered the question.



## Client
This is a client who owns or manages multiple [Site][]s.

### Firebase Path
`/global/clients/{id}`

### Legacy Path
`/legacy/clients/{id}`

### Legacy Mapping
`/legacy-index/clients/{CUSTNMBR}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` client

### Properties
- `address` ([Address][], required) - Address of the client.
- `alternatePhoneNumber` - ([PhoneNumber][], optional) - An alternate phone number for the client.
- `billingType` (enum, required) - How this client is billed.  Possible values:
- TODO: enumerate values
- `contactName` (string, optional) - The name of the main contact person at the client.
- `phoneNumber` ([PhoneNumber][], optional) - The phone number of the main contact person at the client.
- `faxNumber` ([PhoneNumber][], optional) - The fax number of the client.
- `name` (string, required) - The name of the client.
- `number` (string, required) - The client's unique customer number.

### Larson-Specific Properties
- `legacyId` (string, optional) - `DEX_ROW_ID` of the entity in Larson's systems.



## Coordinate
This is a GPS location.  _Note: This object does not have `createdAt` or `lastModifiedAt` [Timestamp][]s._

### Firebase Path
Not a global entity.

### Properties
- `lat` (number, required) - Latitude of the coordinate.
- `lon` (number, required) - Longitude of the coordinate.



## Dispatcher
This is a dispatcher who manages [Job][]s and [Appointment][]s.

### Firebase Path
`/global/dispatchers/{id}`

### Properties
- `name` (string, required) - Name of the dispatcher.
- `phoneNumber` ([PhoneNumber][], optional) - Phone number of the dispatcher.



## Document
This is a generic document, e.g. reference manual, how-to guide.  _Note: This object does not have `createdAt` or `lastModifiedAt` [Timestamp][]s._

### Firebase Path
`/global/documents/{id}`

### Elasticsearch Mapping
- `index` firebase-global
- `type` document

### Properties
- `name` (string, required) - Name of the document.
- `type` (string, required) - Type of document.
- `url` (string, required) - URL to the document.



## Engineer
This is an engineer that works in the field and services [Appointment][]s.

### Firebase Path
`/global/engineers/{id}`

### Elasticsearch Mapping
- `index` firebase-global
- `type` engineer

### Properties
- `currentVehicleId` (string, optional) - ID of the [Vehicle][] the engineer is currently utilizing.
- `firstName` (string, optional) - First name of the engineer.
- `lastName` (string, optional) - Last name of the engineer.
- `phoneNumber` ([PhoneNumber][], optional) - This engineer's cell phone number, in case he needs to be reached in the field.
- `email` (string, required) - Email of the engineer.
- `team` (string, optional) - Team the engineer is on.
- `assignedAppointments` ([ObjectReferences][] of [Appointment][] , optional) - a list of appointment which assigned to this engineer. 🚨

### Larson-Specific Properties
- `legacyId` (string, required) - `Technician_ID` of the entity in Larson's systems.


## Equipment
This is a piece of equipment that is installed at a [Site][].

### Firebase Path
`/global/equipment/{id}`

### Elasticsearch Mapping
- `index` firebase-global
- `type` equipment

### Properties
- `shortDescription` (string, required) - The shorthand description for this piece of equipment.  (This is not a quantity.)
- `manufacturer` (string, optional) - The manufacturer of this piece of equipment.
- `modelNumber` (string, optional) - The manufacturer's model number of the equipment.
- `name` (string, optional) - The name or type of the equipment.
- `description` (string, optional) - A description of the equipment.
- `parts` ([ObjectReferences][] of [Part][] ID to array[[PartInstance][]], optional) - A collection of parts that are part of this equipment.
- `siteId` (string, required) - ID of the [Site][] this equipment is located at.

### Larson-Specific Properties
- `legacyId` (string, optional) - `DEX_ROW_ID` of the entity in Larson's systems.



## Issue
This describes a problem which prompted a [Job][] to be requested, and can be resolved via an [Appointment][].

### Firebase Path
`/global/issues/{id}`

### Elasticsearch Mapping
- `index` firebase-global
- `type` issue

### Properties
- `appointmentId` (string) - ID of the [Appointment][] this issue was reported with. 🚨
- `description` (string, optional) - A description of the problem, for the purpose of helping the [Engineer][] on-site during the [Appointment][].
- `equipment` ([ObjectReferences][] of [Equipment][], optional) - The [Equipment][] which is believed to be the cause of the problem.
- `jobId` (string, required) - ID of the [Job][] this issue was reported with. 🚨
- `recommendedParts` ([ObjectReferences][] of [Part][] ID to [Recommendation][], optional) - [Part][]s that are recommended which may resolve the issue.
- `selectedParts` ([ObjectReferences][] of [Part][], optional) - [Part][]s that are selected by the [Dispatcher][] which she suspects may resolve the issue.
- `recommendedDocuments` ([ObjectReferences][] of [Document][] ID to [Recommendation][], optional) - Documents that the system recommends are relevant to fixing the issue
- `type`: ([TypeOfProblem][], required) - Type of Problem (enum values from Larson - `Type_of_Problem`).



## Job
This is created when a customer calls in with a service request.

### Firebase Path
`/global/jobs/{id}`

### Legacy Path
`/legacy/jobs/{id}`

### Legacy Mapping
`/legacy-index/jobs/{Service_Call_Id}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` job

### Properties
- `appointments` ([ObjectReferences][] of [Appointment][]) - The appointments reported for this job. 🚨
- `issues` ([ObjectReferences][] of [Issue][]) - The issues reported for this job. 🚨
- `high`
- `normal`
- `serviceDescription` (string, optional) - Description of the work to be done during this job.
- `siteId` (string, required) - ID of the [Site][] on which the work needs to be performed. 🚨
- `status` ([StatusOfCall][], required) - Status of the current job (enum values from Larson - `Status_of_Call`).
- `type` ([TypeOfCall][], required) - Type of Job (enum values from Larson - `Type_of_Call`).

### Larson-Specific Properties
- `legacyId` (string, optional) - `Service_Call_ID` of the Job within Larson's system.



## Note
This is a general note that an [Engineer][] can leave about a [Site][].  _Note: This object does not have `createdAt` or `lastModifiedAt` [Timestamp][]s._

### Firebase Path
Not a global entity.

### Properties
- `authorId` (string, optional) - ID of the [Engineer][] that authored the note.
- `content` (string, required) - The text content of the note.
- `writtenAt` ([Timestamp][], optional) - When this note was written.



## Office
This is an office of the company.

### Firebase Path
`/global/offices/{id}`

### Properties
- `address` ([Address][], optional) - Address of the office.
- `dispatchers` ([ObjectReferences][] of [Dispatcher][], optional) - The dispatchers associated with this office.
- `engineers` ([ObjectReferences][] of [Engineer][], optional) - The engineers associated with this office.
- `name` (string, required) - Name of the office.



## Part
This is the definition of a specific kind of part.  _It does not reflect an instance of a part--for that, see [PartInstance][]._

### Firebase Path
`/global/parts/{id}`

### Legacy Path
`/legacy/parts/{id}`

### Legacy Mapping
`/legacy-index/parts/{ITEMNMBR}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` part

### Properties
- `alternateNumber` (string, optional) - An alternate part number.
- `alternateParts` ([ObjectReferences][] of [Part][], optional) - Other parts that could be used in place of this part.
- `description` (string, optional) - A description of the part.
- `name` (string, required) - The name of the part.
- `number` (string, required) - The part number, used for searching or referencing a part by something other than its name.
- `inventoryItemType` ([InventoryItemType][], optional) - A client field indicating a part type.
- `vendorId` (string, optional) - vendor identifier for this part.
- `vendorName` (string, optional) - vendor name for this part.
- `price` (number, optional) - The current price for a single instance of this part, in US Dollars.



## PartInstance
This is an instance of a group of one or more parts.  Despite the insinuation of singularity from the word "Instance" in the title, this can reflect more than one actual part.

### Firebase Path
Not a global entity.  The owner of the part instance is implied by its location in Firebase.  For example, any parts under `/global/vehicles/{id}/inventory` are on a [Vehicle][].

### Properties
- `description` (string, optional) - A description of how the parts are used.
- `installDate` ([Timestamp][], optional) - The date these parts were installed at a [Site][].
- `partId` (string, required) - ID of the [Part][] this is an instance of.
- `quantity` (number, required) - The number of parts in this instance.
- `storageLocation` (string, optional) - If being stored somewhere, an additional description of where the [Part][] is (e.g. `Bin 32` in a [Warehouse][]).
- `warrantyExpiration` ([Timestamp][], optional) - The date the warranty expires for these parts.
- `warrantyProvider` (enum, optional) - The provider of the warranty for these parts.  Possible values:
  - `manufacturer`
  - `Larson`
  - `refurb`



## PartRequest
This is the request for an instance of parts.

### Firebase Path
`/global/partRequests/{id}`

### Properties
- `appointmentId` (string, required) - ID of the [Appointment][] which the part request is raised for. 🚨
- `partId` (string, required) - ID of the [Part][] being requested.
- `quantity` (number, required) - The number of parts being requested.
- `engineerId` (string, optional) - ID of the [Engineer][] requesting the parts.
- `status` (string, required) - Status of the part request.  
  - TODO: change it to enumerate values. Right now we haven't implement the whole flow yet so keep it string for now
- `priority` (string, required) - Priority of the part request.
  - TODO: change it to enumerate values. Right now we haven't implement the whole flow yet so keep it string for now
- `deliveryMethod` (string, required) - method of delivery of the part request
  - TODO: change it to enumerate values. Right now we haven't implement the whole flow yet so keep it string for now
- `notes` ([ObjectReferences][] of [Note][], optional) - A list of notes about this part




## Payment
Payments collected by [Engineer][]. _Note: This object does not have `lastModified` [Timestamp][]._

### Firebase Path
Not a global entity.

### Properties
- `paymentMethod` (enum of [PaymentMethod][], required) - payment method (enum values from Larson - `Payment_Method`).
- `amount` (number, required) - amount paid.



## PhoneNumber
US phone numbers should be stored as a string, without the leading "1" and without any formatting (i.e. only digits).  This allows the UI to easily format the phone number however it chooses.



## Recommendation
This encapsulates the recommendation of a various entity.  Since this is not a global object, the type of the entity and its ID is inferred from how it is reference.  For example, for recommended [Part][]s, it may look like this:
```
{
  "recommendedParts": {
    "partId1": {
      "score": 0.75,
      "feedback": {
        "0": {
          "authorId": "engineerId1",
          "type": "confirm",
          "submittedAt": "[timestamp]"
        }
      }
    },
    "partId2": {
      "score": 0.65
    }
  }
}
```
From the context of `recommendedParts` we know that the IDs stored are [Part][] IDs.

### Firebase Path
Not a global entity.

### Properties
- `feedback` (array[[RecommendationFeedback][]], optional) - Feedback about the recommendation.
- `score` (number, optional) - Numeric score from `[0-1]` (inclusive) of how relevant the recommendation is, with the higher score being a more relevant recommendation.



## RecommendationFeedback
This encapsulates feedback about a [Recommendation][].  _Note: This object does not have `createdAt` or `lastModifiedAt` [Timestamp][]s._

### Firebase Path
Not a global entity.

### Properties
- `authorId` (string, required) - ID of the user who provided the feedback.
- `authorType` (enum, required) - The type of user who provided the feedback.  Possible values:
  - `dispatcher`
  - `engineer`
- `submittedAt` ([Timestamp][], required) - When this feedback was submitted.
- `type` (enum, required) - The kind of feedback.  Possible values:
  - `add`
  - `confirm`
  - `remove`


## SearchRequest
This is a search request that will be made to Elasticsearch.

### Firebase Path
`/search/request/{id}`

### Properties
- `index` (string or array, required) - A comma-separated list of index names to search; use _all or empty string to perform the operation on all indices
- `type` (string or array, required) - A comma-separated list of document types to search; leave empty to perform the operation on all types
- `q` (string, required) - Query in the Lucene query string syntax
- `size ` - (number) The number of search results to return (defaults to 10 if not specified)
- `explain` - (boolean) - Specify whether to return detailed information about score computation as part of a hit
- (Full list of properties that can be specified for requests to Elasticsearch are documented [here](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#api-search) )

## SearchResponse
This is a search response from Elasticsearch.

### Firebase Path
`/search/response/{id}`

### Properties
- `max_score` -
- `total` - total number of documents matching our search criteria
- `hits` – search results
- `hits.hits` – actual array of search results (defaults to first 10 documents)
- `hits.explain` - further detailed info about the result. Provided when explain=true was specified on the search request.
- (More Elasticsearch response documentation can be found [here ](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/_the_search_api.html))

## Site
This is the site of equipment that needs to be maintained/fixed at an [Appointment][].

### Firebase Path
`/global/sites/{id}`

### Legacy Path
`/legacy/sites/{id}`

### Legacy Mapping
`/legacy-index/sites/{CUSTNMBR}/{ADRSCODE}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` site

### Properties
- `address` ([Address][], required) - Address of the site.
- `alternatePhoneNumber` - ([PhoneNumber][], optional) - An alternate phone number for the site.
- `clientId` (string, required) - ID of the [Client][] that manages or owns this site.
- `contactName` (string, optional) - The name of the contact person for this site.
- `equipment` ([ObjectReferences][] of [Equipment][] ID, optional) - The [Equipment][] that is installed at this site. 🚨
- `faxNumber` ([PhoneNumber][], optional) - The fax number of the site.
- `installedParts` ([ObjectReferences][] of [Part][] ID to array[[PartInstance][]], optional) - The parts that have been installed at this site during past [Appointment][]s.
- `jobs` ([ObjectReferences][] of [Job][]) - The jobs reported for this site. 🚨
- `name` (string, required) - The name of the site.
- `notes` (array[[Note][]], optional) - The notes that [Engineer][]s have taken regarding this site.
- `phoneNumber` ([PhoneNumber][], optional) - The primary phone number for contacting this site.

### Larson-Specific Properties
- `legacyId` (string, optional) - `[CUSTNMBR]-[ADRSCODE]` of the entity in Larson's systems.



## Timestamp
A UNIX timestamp in milliseconds.



## Vehicle
This is a vehicle which [Engineer][]s drive to [Appointment][]s and house an inventory of [Part][]s.

### Firebase Path
`/global/vehicles/{id}`

### Legacy Path
`/legacy/vehicles/{id}`

### Legacy Mapping
`/legacy-index/vehicles/{LOCNCODE}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` vehicle

### Properties
- `inventory` ([ObjectReferences][] of [Part][] ID to [PartInstance][]) - The inventory of parts for this vehicle.
- `location` ([Coordinate][], optional) - Most recently known location of the vehicle.
- `locationUpdatedAt` ([Timestamp][], optional) - When the `location` of this vehicle was last updated.
- `name` (string, required) - Name of the vehicle.

### Larson-Specific Properties
- `legacyId` (string, optional) - `DEX_ROW_ID` of the entity in Larson's systems.

## Warehouse
This is a warehouse which houses [Part][]s.

### Firebase Path
`/global/warehouses/{id}`

### Legacy Path
`/legacy/warehouses/{id}`

### Legacy Mapping
`/legacy-index/warehouses/{LOCNCODE}` = BusinessOS ID

### Elasticsearch Mapping
- `index` firebase-global
- `type` warehouse

### Properties
- `address` ([Address][], required) - Address of the warehouse.
- `description` (string, optional) - Description of the warehouse.
- `inventory` ([ObjectReferences][] of [Part][] ID to [PartInstance][]) - The inventory of parts for this warehouse.
- `name` (string, required) - Name of the warehouse.

### Larson-Specific Properties
- `legacyId` (string, optional) - `DEX_ROW_ID` of the entity in Larson's systems.



## WorkOrder
This is a write-up of the work that was done by an [Engineer][] during an [Appointment][].

### Firebase Path
`/global/workOrders/{id}`

### Elasticsearch Mapping
- `index` firebase-global
- `type` workOrder

### Properties
- `appointmentId` (string, required) - ID of the [Appointment][] for which the work was done.
- `description` (string, optional) - A description of the work that was completed.
- `engineerId` (string, required) - ID of the [Engineer][] that submitted the work order.
- `jobCOD` (boolean, default = `false`) - Whether or not the job is COD (cash on deposit).
- `paymentCollected` (boolean, optional, default = `false`) - Whether or not [Engineer][] has collected the payment for COD job
- `payment` (array[[Payment][]], optional) - Customer's payments for COD job.
- `partsUsed` ([ObjectReferences][] of [Part][] ID to [PartInstance][], optional) - The parts that were used to complete this work order.
- `partsMissing` ([ObjectReferences][] of [PartRequest][] ID, optional) - Additional parts that were needed in order to complete the requested service.
- `returnTripNeeded` (boolean, default = `false`) - Whether or not a return trip is needed to complete more work.
- `returnTripReason` (string, optional) - The reason a return trip will be needed.
- `workCompleted` (boolean, default = `true`) - Whether or not all of the work was completed that this appointment was created for.



## ZipCode
US zip codes should be stored as a string without any formatting.  Two formats for zip codes are acceptable:
* 12345 (the standard 5 digit zipcode)
* 123456789 (5 digit zipcode + 4 additional digits)
It is up to the UI application to properly format 9-digit zip codes (as either 12345-6789 or 12345 6789).

## DispatcherApp

[Dispatcher App Schema](dispatcherApp)


## Something Missing?
See the [Contribution Guide](CONTRIBUTING.md) for more information on how to suggest changes.



[Address]: #address
[Appointment]: #appointment
[AppointmentQuestion]: #appointmentquestion
[Client]: #client
[Coordinate]: #coordinate
[Dispatcher]: #dispatcher
[Document]: #document
[Engineer]: #engineer
[Equipment]: #equipment
[Issue]: #issue
[Job]: #job
[Note]: #note
[ObjectReferences]: #objectreferences
[Office]: #office
[Part]: #part
[PartInstance]: #partinstance
[PartRequest]: #partrequest
[Payment]: #payment
[PhoneNumber]: #phonenumber
[Recommendation]: #recommendation
[RecommendationFeedback]: #recommendationfeedback
[Site]: #site
[Timestamp]: #timestamp
[Vehicle]: #vehicle
[Warehouse]: #warehouse
[WorkOrder]: #workorder
[ZipCode]: #zipcode

[AppointmentStatus]: enums/appointment-status.md
[InventoryItemType]: enums/inventory-item-type.md
[PaymentMethod]: enums/payment-method.md
[StatusOfCall]: enums/status-of-call.md
[TypeOfCall]: enums/type-of-call.md
[TypeOfProblem]: enums/type-of-problem.md
