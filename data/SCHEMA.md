# Data Schema Information

This describes the schema as I am working on it.  I think part of this document should probably end up in the final product information:

I am trying to make things as flexible as possible, while maintaining things fairly simply.  So rather than making sure that anyone can create any kind of schema, I have decided that there are certain things which will be common to how things are handled.

Every bit of data has a bit of metadata which points to it.  This really is about data that the database itself is using to point to stuff.  This is how I want to look at things.  As far as the metadata, there are potential problems with the way I have set things up, but I will probably get it to work out.

## Tables

Tables that I feel will probably be needed to start off with are:

* Metadata
* Users
* Status Data
* Content Types
* Permissions

Those tables look like:

### Metadata: 

Field | Field Type | Reference Table | Note
--- | --- | --- | ---
ID | Int (unique) (not null) (unsigned) | NULL | The ID of the metadata. 
Owner | ID | User | [Information about Ownership][1]
Creator | ID | User | The Original Owner [See Also][1]
Version Number | Integer | NULL | This is incremented whenever data is changed. [See Also][1]
Previous Version | ID | Metadata | Points to the [previous version][2] of data.
Date | DateTime | NULL | This is the date of this version.
Original Cretion Date | DateTime | Null | This is the date of the original version. [See Also][1] [Further See][2]
Status | ID | Status | This will contain the status of the data which is related to this bit of metadata.  
Content | ID | Various | [Content][3] will be a bit tricky.
Content Type | ID | Content Types | Points to the type of data which is being referenced.  

#### Metadata notes

I do not really see that this information is likely to be actually displayed to the end user.  There *may* be cases where it would be displayed to an administrator, but I am not sure that it really would actually do that.  

The Content Type table would hold information about content types which are displayed.  

### Content Type

Field | Field Type | Reference Table | Note
--- | --- | --- | ---
ID | 

#### Data Types Notes
#### Some of this information came out of thinking about how the metadata was being handled

Anything which the metadata points to would have to have a content type defined.  At least, anything which the metadata points to through the content ID needs to have a content type defined.  

The metadata is intended to track changes to data with any kind of information.

The data itself is *not* supposed to change.  (this will be relevant when talking about users, and possibly information about the other tables this points to).

With stuff other than *content* being pointed here.  Those tables *probably* will need some kind of *change* information, or those tables will refer to the content which will be refered to through an ID which points to them in a seperate data table.

I am thinking about the users.  How do we handle the data which exists for a user.  One option which I have thought of has been the maintaining of "current data" within the user table itself, and then historical data in other tables which would be pointed to quite possibly via this metadata table.

One of the things which I have wanted to make sure has historical information is passwords.  We will *never* store actual passwords.  But I want to maintain historical passwords.  I am not sure if I want to have some way of looking at historical passwords in terms of being able to test against them, and also in terms of being able to see if there are users which are using the same password.

Generally, I want the password information to be something which would be very difficult to get very much, if any information about the content of those passwords without at the very least, access to the code which is on the server which is setting the passwords.  
 
OK, so:

1) The Data Types table needs to refer to any table which the Metadata points tothrough the content field.
2) Any data which can be changed, will have metadata attached to it.
3) Certain data may be changed directly, as the metadata table refers to that data directly.
4) Data which can be changed directly, may have historical records of that data, but will not be required to.
5) If the data will be maintained historically (this should be an option which can be turned on or off, either at the admin level, or the user level, preferablly both, with the admin being able to set whether something is overridable or not), of tables which the metadata refers directly to, the historical data would be in a seperate table, with a seperate data type.
6) Data types will not be needed for any data which is not refered to in the metadata content field.  
    1) If something is directly refered to by the metadata then it would not nescesarilly be refered to in the content field:
         * These tables are, 
         * metadata itself
         * User
         * Status
         * Content Type
    2) If historical data is maintained, it would be refered to by the metadata table
    3) Historical data would be maintained in a serarate table from the main fields.
    4) Those main tables in the metadata table do *not* need to have Content Types defined, but the historical data tables will.

## Field Types

* ID
** An ID field
** Integer
** Not Null
*** I will try to not create the need to have any reference IDs which can be null, that is that there is no reference being made.  
*** Deleted data will not null a reference ID, but it will delete the actual data, and it *may* delete the actual record.  
*** If actual records are deleted that ID is no longer used in that table, so it *may* allow for the re-use of IDs. 
*** Without deleting the record, the data will be gone, but the ID will not be freed. 
*** This is easier to handle, but does not allow for the reuse of IDs, which may with large datasets, lead to the need to increase the size of Integer. 
*** At this point, we will proabably *not* be deleting actual records.  
*** When the size of the ID data type becomes an issue (most likely first with the metadata ID) we will decide if re-using IDs, or increasing the size of the ID is preferable.
*** It will proabbly be decided that the size of the ID is likely preferable. 
** Unsigned
* Integer
** 16 bits
** Unsigned
*** If signed is needed will define new field type.
** Not Null
*** If Null needed will define new field type.
* Date Time
** 32 bit Integer
** Seconds after Unix Epoch (1970 January 1)
*** Need to create a different version for dates prior to this, or after 2039

## Extended Notes

### Ownership

Ownership refers to the current owner.  In most cases it will probably be the same as the Creator.  There are cases where the owner of data would be different:

1) When creating data, the creator defines a different owner.
2) When the creator or owner of data transfers ownership to another owner.
3) When someone "clones" data somewhat compeplex things happen:
    1) The Creator remains the same with the new data
    2) The Pointer to the data remains the same
    3) New Metadata record is created
    4) Current owner remains the same
    5) Status of the data which is being cloned remains the same
    6) Person cloning data becomes new owner
    7) Changes to permissions will likely be made
    8) Version Number is set to 0
    9) Pointer to previous version points to the version which is being cloned
    10) The Original Creation Date remains the same


I think that may be it.

### Previous Version

The Previous Version is used for a few reasons:

1) It allows us to track changes.
2) It allows us to pay attention to forks, and the changes of a fork.
3) With notes to previous versions, we can either archive, maintain, or delete data.  The deletion of data does actually mean that the data is deleted, but the metadata is not.  
    1) Metadata does not contain any information about content beyond the content type, and a pointer to the actual content itself.  
    2) We do not even write a note about changes in the metadata.  If a plugin wishes to maintain change information they will need to do so through actual data, and that may end up being a bit tricky.  

### Content

There are a number of issues which I see here.  I do not actually want to change any actual content that is in the database (thus we keep old content, even when something is edited) the issue here is we will be maintaining a single ID which will point to multiple tables (content types).  This brings up a bit of an issue.

With multiple content types, which will be in different tables, it means that content will be sharing diferent IDs.  So, this field cannot be unique.  But there may be a way to create a unique index for each pair of a "content type" and "content".

When cloning data the pair of "content type" and "content ID" will not actually change.  At least the "cloning" process will not change it.  And since cloning can be done by the owner of the data, the cloning process may not change anything significant about the metadata which can mean we can generate a unique ID for the content.  

This will be something which needs to be worked out over time.

[1] <#ownership> "Information about Ownership of data, in relation to Metadata"
[2] <#previous version> "Information about what the purpose of the previous verision field is used for.
[3] <#content> "Iformation about how the content ID will be handled.
