=================
NEXT Architecture
=================



Reference Architecture
======================
A very high-level view of the system

.. figure:: _static/reference-architecture-1.png
    :width: 1000px
    :align: center
    :alt: Reference Architecture
    :figclass: align-center


Technology Choices
==================
To fill in several of the above roles, we have adopted RethinkDB as our data storage engine. In addition to a
document storage interface, RethinkDB is unique in that every modification made to a table creates an entry
in a change stream. By leveraging this power, we can react to events asynchronously with stronger guarantees
than you can get in a traditional database without a two-phase commit between your database and message queue. 

.. figure:: _static/rethinkdb-architecture.png
    :width: 1000px
    :align: center
    :alt: RethinkDB Architecture
    :figclass: align-center

Data Mapping
============
As you may expect, Active Directory and Azure Active Directory are very similar identity directories, but they do have some minor differences that must be accounted for. In addition, there are several fields that come back in responses from one provider and not another. 

To account for these differences and support the synchronization efforts of this project, we have to perform some basic field mapping from each provider into a standard NEXT field, then translate those differences correctly when making changes on the other provider. 

In the future, we expect this mapping to be customizable, but as of now it is a static mapping. 

***Any fields not explicitly listed here are not managed by NEXT, nor synchronized back to any upstream identity provider***.

User Fields

+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|      AD Field       |   AD Field Xform     |     NEXT Field      |   AAD Field Xform    |       AAD Field             |
+=====================+======================+=====================+======================+=============================+
| objectGUID          |                      | user_id             |                      | id                          |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| whenCreated         |                      | created_date        |                      | createdDateTime             |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | deleted_date        |                      | deletedDateTime             |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | account_enabled     |                      | accountEnabled              |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| telephoneNumber     |                      | business_phones     |                      | businessPhones              |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| company             |                      | company_name        |                      | companyName                 |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| countryCode         |                      | country             |                      | country                     |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | city                |                      | city                        |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | office_location     |                      | officeLocation              |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | postal_code         |                      | postalCode                  |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | state               |                      | state                       |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| countryCode         |                      | country             |                      | country                     |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| department          |                      | department          |                      | department                  |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| memberOf            |                      | member_of           |                      |                             |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| displayName         |                      | name                |                      | displayName                 |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| employeeID          |                      | employee_id         |                      | employeeId                  |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| givenName           |                      | given_name          |                      | givenName                   |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| title               |                      | job_title           |                      | jobTitle                    |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| mail                |                      | email               |                      | mail                        |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| cn                  |                      | user_nickname       |                      | mailNickname                |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| manager             |                      | manager             |                      | manager                     |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| mobilePhone         |                      | mobile_phone        |                      | mobilePhone                 |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| distinguishedName   |                      | distinguished_name  |                      | onPremisesDistinguishedName |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| preferredLanguage   |                      | preferred_language  |                      | preferredLanguage           |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| streetAddress       |                      | street_address      |                      | streetAddress               |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | surname             |                      | surname                     |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | usage_location      |                      | usageLocation               |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
| userPrincipalName   |                      | user_principal_name |                      | userPrincipalName           |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+
|                     |                      | user_type           |                      | userType                    |
+---------------------+----------------------+---------------------+----------------------+-----------------------------+


---

Group Fields

+---------------------+----------------------+---------------------+----------------------+----------------------+
|      AD Field       |   AD Field Xform     |     NEXT Field      |   AAD Field Xform    |       AAD Field      |
+=====================+======================+=====================+======================+======================+
| objectGUID          |                      | role_id             |                      | id                   |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| whenChanged         |                      | created_date        |                      | createdDateTime      |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | deleted_date        |                      | deletedDateTime      |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | classification      |                      | classification       |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| description         |                      | description         |                      | description          |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| name                |                      | name                |                      | displayName          |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| groupType           |                      | group_types         |                      | groupTypes           |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | group_nickname      |                      | mailNickname         |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | mail_enabled        |                      | mailEnabled          |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| member              |                      | members             |                      | members              |
+---------------------+----------------------+---------------------+----------------------+----------------------+
| managedBy           |                      | owners              |                      | owners               |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | security_enabled    |                      | securityEnabled      |
+---------------------+----------------------+---------------------+----------------------+----------------------+
|                     |                      | visibility          |                      | visibility           |
+---------------------+----------------------+---------------------+----------------------+----------------------+


Domain Concepts
===============
The workflow of this system is a proposal-based asynchronous messaging
platform. Messages propose changes to the global state, and then are
confirmed or rejected by others with the appropriate permissions.

The core domain objects:

.. csv-table:: Domain Objects
   :header: "Object", "Purpose"
   :widths: auto

    "SysAdmin", "A special NEXT system administrator role"
    "User",     "A User is an entity representing an individual or service account"
    "Proposal", "Encapsulates request to modify permissions"
    "Role",     "A Role maintains a list of Users assigned to that Role, as well as a list of Tasks that members are authorized for"
    "Task",     "A Task is an individual unit of Permission"
    "Email",    "Email-type object"
    "Key",      "Users' Public Key(s)"

Roles and Tasks can have Owners, Admins, and Members. Roles can additionally have Tasks.

.. csv-table:: Role and Task Admin/Owner/Member
   :header: "RoleType", "Purpose"
   :widths: auto

    "Admin",  "Implies Owner; Can add/remove Owners and Admins"
    "Member", "User is a member of role/task for authorization purposes"
    "Owner",  "Implies Member; Can approve/reject/modify membership"
    "Task",   "Role only; A permission granted to members of this group"

Changes to the global state are done through proposals in a message queue. A
few representative message examples (not comprehensive):

.. csv-table:: Messages
   :header: "NEXT Message",        "ActionType", "SubActionType", "AddressType",      "ObjectType", "RelatedType", "RelationshipType", "Description"
   :widths: auto

    "ADD_KEY",                     "Add",        "None",          "Key",              "Key",        "None",        "None",             ""
    "CONFIRM_ADD_ROLE_ADMIN",      "Confirm",    "Add",           "Proposals",        "Role",       "User",        "Admin",            ""
    "CONFIRM_ADD_ROLE_MEMBER",     "Confirm",    "Add",           "Proposals",        "Role",       "User",        "Member",           ""
    "CONFIRM_ADD_ROLE_OWNER",      "Confirm",    "Add",           "Proposals",        "Role",       "User",        "Owner",            ""
    "CONFIRM_ADD_ROLE_TASK",       "Confirm",    "Add",           "Proposals",        "Role",       "Task",        "Member",           ""
    "IMPORTS_ROLE",                "Imports",    "None",          "Roles_Attributes", "Role",       "None",        "Attributes",       ""
    "PROPOSE_ADD_ROLE_ADMIN",      "Propose",    "Add",           "Proposals",        "Role",       "User",        "Admin",            ""
    "PROPOSE_ADD_ROLE_MEMBER",     "Propose",    "Add",           "Proposals",        "Role",       "User",        "Member",           ""
    "PROPOSE_ADD_ROLE_OWNER",      "Propose",    "Add",           "Proposals",        "Role",       "User",        "Owner",            ""
    "PROPOSE_ADD_ROLE_TASK",       "Propose",    "Add",           "Proposals",        "Role",       "Task",        "Member",           ""
    "REJECT_ADD_ROLE_ADMIN",       "Reject",     "Add",           "Proposals",        "Role",       "User",        "Admin",            ""
    "REJECT_ADD_ROLE_MEMBER",      "Reject",     "Add",           "Proposals",        "Role",       "User",        "Member",           ""
    "REJECT_ADD_ROLE_OWNER",       "Reject",     "Add",           "Proposals",        "Role",       "User",        "Owner",            ""
    "REJECT_ADD_ROLE_TASK",        "Reject",     "Add",           "Proposals",        "Role",       "Task",        "Member",           ""
    "CONFIRM_ADD_TASK_ADMIN",      "Confirm",    "Add",           "Proposals",        "Task",       "User",        "Admin",            ""
    "CONFIRM_ADD_TASK_OWNER",      "Confirm",    "Add",           "Proposals",        "Task",       "User",        "Owner",            ""
    "CREATE_TASK",                 "Create",     "None",          "Tasks_Attributes", "Task",       "None",        "Attributes",       ""
    "PROPOSE_ADD_TASK_ADMIN",      "Propose",    "Add",           "Proposals",        "Task",       "User",        "Admin",            ""
    "PROPOSE_ADD_TASK_OWNER",      "Propose",    "Add",           "Proposals",        "Task",       "User",        "Owner",            ""
    "REJECT_ADD_TASK_ADMIN",       "Reject",     "Add",           "Proposals",        "Task",       "User",        "Admin",            ""
    "REJECT_ADD_TASK_OWNER",       "Reject",     "Add",           "Proposals",        "Task",       "User",        "Owner",            ""
    "CONFIRM_UPDATE_USER_MANAGER", "Confirm",    "Update",        "Proposals",        "User",       "User",        "Manager",          ""
    "CREATE_USER",                 "Create",     "None",          "User",             "User",       "None",        "Attributes",       ""
    "PROPOSE_UPDATE_USER_MANAGER", "Propose",    "Update",        "Proposals",        "User",       "User",        "Manager",          ""
    "REJECT_UPDATE_USER_MANAGER",  "Reject",     "Update",        "Proposals",        "User",       "User",        "Manager",          ""

Notice that every Proposal has related Confirm and Reject messages.

Blockchain Storage
==================

The underlying distributed ledger (aka Blockchain) implementation we're using
is Sawtooth. The generic model of this ledger is a key-value store
with an address key indexing to an opaque string. This string is a serialized
and compressed protobuf. 

Applications that work with the Sawtooth platform must determine how to index
their data and keys. Sawtooth gives the following specification:

.. figure:: _static/hyperledger_addressing.png
    :width: 958px
    :align: center
    :alt: Hyperledger Addressing
    :figclass: align-center

Given this, we have chosen to address our data like so:

+---------------------+----------------------+---------------------+-------------------------+
|  Bytes              |  Purpose             |  Example            |  Extra                  |
+=====================+======================+=====================+=========================+
| 0-2 (3 Bytes)       | Namespace            | 'bac001'            | Always 'bac001', UTF-8  |
+---------------------+----------------------+---------------------+-------------------------+
| 3-4 (2 Bytes)       | Reserved             | 0x0000              | Always 0x0000; Reserved |
+---------------------+----------------------+---------------------+-------------------------+
| 5-6 (2 Bytes)       | Object Type          | 0x0028 (Proposal)   | See "Domain Objects"    |
+---------------------+----------------------+---------------------+-------------------------+
| 7-18 (12 Bytes)     | Object ID Hash       | 0x....abcd123456789 | We generate this hash   |
+---------------------+----------------------+---------------------+-------------------------+
| 19-20 (2 Bytes)     | Related Object Type  | 0x0032 (Role)       | See "Domain Objects"    |
+---------------------+----------------------+---------------------+-------------------------+
| 21 (1 Byte)         | Relationship Type    | 0x88 (Manager)      |                         |
+---------------------+----------------------+---------------------+-------------------------+
| 22-33 (12 Bytes)    | Related Obj ID Hash  | 0x....abcd123456789 | All 0-byte for 'None'   |
+---------------------+----------------------+---------------------+-------------------------+
| 34 (1 Byte)         | Reserved             | 0x00                | Always 0x00; Reserved   |
+---------------------+----------------------+---------------------+-------------------------+

Given this scheme, we can refer to entries as a tuple:

.. code::

   Address ~= (ObjectType, ObjectId, RelatedType, RelationshipType, RelatedId)

This reads a bit like a sentence: User<X> is <a member of> Role<Y>. Given that
we are storing this in (essentially) a key-value store, this data structure
gives us something akin to an adjacency list.

There are two parts to storage: the address and the payload. Some facts about
the state of the system can be checked by the mere presence of data at an
address. For example, a user's membership in a role can be validated by
requesting the tuple (Role, :code:`0x123...`, User, Member, :code:`0x456...`).

Messages are not persisted on the blockchain, but proposals are.
The primary structures stored on the blockchain include:

.. csv-table:: Addressing Types
    :header: "ObjectType", "RelatedType", "RelationshipType", "Purpose"
    :widths: auto

    "SysAdmin", "None", "Attributes", "A System Maintainer Role record"
    "User",     "None", "Attributes", "A User record"
    "Proposal", "None", "Attributes", "A Proposal record"
    "Role",     "None", "Attributes", "A Role record"
    "Task",     "None", "Attributes", "A Task record"
    "Email",    "None", "None",       "An Email record"
    "Key",      "None", "None",       "A User's Public/Private Key Record"

Notice the "RelatedType" is "None" for all of those. Relationships:

.. csv-table:: Addressing Relationships
   :header: "ObjectType", "RelatedObjectType", "RelationshipType", "Purpose"
   :widths: auto

    "SysAdmin", "User",  "Admin",        "User is an admin of SysAdmin Role"
    "SysAdmin", "User",  "Member",       "User is a member of SysAdmin role"
    "SysAdmin", "User",  "Owner",        "User is an owner of SysAdmin role"

    "User",     "Email", "Owner",        "User's Email Address"
    "User",     "Key",   "Owner",        "User's Keypair Information"
    "User",     "User",  "Manager",      "User2 is manager of User1"
    "User",     "User",  "DirectReport", "User2 is direct report of User1"

    "Role",     "User",  "Admin",        "User is an admin of Role"
    "Role",     "User",  "Member",       "User is a member of Role"
    "Role",     "User",  "Owner",        "User is an owner of Role"
    "Role",     "Task",  "Member",       "Task is a member of Role"

    "Task",     "User",  "Admin",        "User is an admin of Task"
    "Task",     "User",  "Owner",        "User is an owner of Task"

