# API Technical Description

## Authentication and User Context Resolution

The application uses Entra ID for authentication.
Users authenticate using their organizational identity, ensuring secure access
and alignment with existing identity and access management policies.

After successful authentication, the application performs a user lookup in
Public 360 via the SIF API to establish the correct user context.

This step is required to:
- Identify the corresponding Public 360 user
- Determine the user's organizational unit
- Ensure that subsequent SIF requests are executed with the correct permissions

### Resolve Public 360 User Information

After login, the authenticated Entra ID user is mapped to a Public 360 user by
using the email address retrieved from the Entra authentication token.

**SIF API Method:**  
`UserService/GetUsers`

**Request Example:**
```json
{
  "UserId": "anna.saethre@tietoevry.com"
}
```

**Notes:**
- The `UserId` value is dynamically populated from the Entra ID authentication token.
- The response is used to resolve the manager’s organizational unit and role.
- The resolved user context is reused for all subsequent SIF API requests.


## Main functionality

### Documents to Assign
This functionality enables managers to assign unassigned documents to appropriate
case handlers or organizational units.

#### Retrieve Documents Awaiting Assignment
Documents that have been routed to the manager's organizational unit but have not
yet been assigned to a responsible person are retrieved for display.

**SIF API Method:**  
`DocumentService/GetDocuments`

**Request Example:**
```json
{
  "IncludeCustomFields": true,
  "AdditionalFields": [
    {
      "Name": "ToOrgUnit",
      "Value": "506"
    },
    {
      "Name": "OurRef",
      "Value": "",
      "OperatorType": { 
        "Value": "IS NULL"
      }
    }
  ]
}
```

**Notes:**
- `ToOrgUnit`: The organizational unit of the logged-in manager (retrieved from user context)
- `OurRef` with `IS NULL`: Filters for documents without an assigned responsible person

#### Retrieve Available Case Handlers (Persons)
When the manager chooses to assign a document to a specific person, the application
retrieves a list of available case handlers within the organization.

**SIF API Method:**  
`ContactService/GetContactPersons`

**Request Example:**
```json
{
  "AdditionalFields": [
    {
      "Name": "EmployerID",
      "Value": "506"
    }
  ]
}
```

**Notes:**
- `EmployerID`: The enterprise recno representing the organization
- Value is derived from the user context established during authentication

#### Retrieve Available Organizational Units (Enterprises)
When the manager chooses to assign a document to another organizational unit, the
application retrieves a list of available enterprises.

**SIF API Method:**  
`ContactService/GetEnterprises`

**Request Example:**
```json
{
}
```

**Notes:**
- This request retrieves all enterprises available in the system
- No filtering is applied at the API level

#### Assign Document to Case Handler or Unit
After the manager selects either a person or an organizational unit, the document
is updated with the appropriate assignment.

**SIF API Method:**  
`DocumentService/UpdateDocument`

**Request Example:**
```json
{
  "DocumentNumber": "25/00052-4",
  "ResponsiblePersonRecno": "200012",
  "Remarks": [
    {
      "Content": "Assignment comment from manager",
      "RemarkType": "recno:1"
    }
  ]
}
```

**Notes:**
- `DocumentNumber`: The unique identifier of the document being assigned
- `ResponsiblePersonRecno`: The recno of the selected case handler (used when assigning to a person)
- `ResponsibleEnterpriseRecno`: The recno of the selected organizational unit (used when assigning to a unit)
- `Remarks`: Optional comment from the manager regarding the assigment. `Content`: Free-text field for manager's comment. `RemarkType`: Hardcoded to recno:1 for assignment remarks.


### Documents to Process
This functionality presents documents that require direct action or decision by the
manager. These are documents where the manager is assigned as the responsible person
and must either respond or sign off the document.

#### Retrieve Documents Requiring Manager Action
Documents assigned to the manager that have not yet been closed are retrieved for
display. The list is filtered to include only incoming documents and internal memos
with follow-up requirements.

**SIF API Method:**  
`DocumentService/GetDocuments`

**Request Example:**
```json
{
  "AdditionalFields": [
    {
      "Name": "OurRef", 
      "Value": "200009"
    },
    {
      "Name": "ClosedDate",
      "Value": "",
      "OperatorType": {
        "Value": "IS NULL"
      }
    }
  ],
  "AdditionalListField": [
    {
      "Name": "ToDocumentCategory",
      "FieldCriteria": [
        {
          "Name": "Recno",
          "Value": "110, 113"
        }
      ]
    }
  ],
  "IncludeCustomFields": true
}
```

**Notes:**
- `OurRef`: The ContactRecno of the logged-in manager (retrieved from user context)
- `ClosedDate` with `IS NULL`: Filters for specific document categories. `110`: Incoming Document, `113`: Internal memo with follow-up (Codetable: Document Category)
- `IncludeCustomFields`: Includes custom fields such as DueDate in the response
- `DueDate` CustomField query:
```xml
<QUERYDESC ENTITY="Document" NAMESPACE="SIRIUS" DATASETFORMAT="XML">
	<RESULTFIELDS>
		<METAITEM TAG="DueDate">DueDate</METAITEM>
	</RESULTFIELDS>
	<CRITERIA>
		<METAITEM NAME="DueDate" OPERATOR="IN">
			<VALUE>{0}</VALUE>
		</METAITEM>
	</CRITERIA>
</QUERYDESC>
```

#### Reply with Letter
When the manager chooses to reply with a formal letter, a new outgoing document
is created in the same case as the original document.
The new document inherits key metadata from the original document, ex: access code, access group etc.

**SIF API Method:**  
`DocumentService/CreateDocument`

**Request Example:**
```json
{
  "CaseNumber": "25/00123",
  "Archive": "2",
  "Category": "111",
  "Title": "Svar på: Application for Building Permit",
  "Status": "recno:1",
  "Files": [
    {
      "Title": "Response Letter",
      "Format": "docx",
      "Base64Data": "...",
      "Status": "recno:2"
    }
  ],
  "Contacts": [
    {
      "Role": "recno:6",
      "ReferenceNumber": "recno:123456",
      "IsUnofficial": false
    }
  ]
}
```

**Notes:**
- `CaseNumber`: Inherited from the original document
- `Archive`: Inherited from the original document
- `Category`: Document category for the reply. If original document category is `110` (Incoming Document), use `111` (Outgoing Document). If original document category is `113` (Internal memo with follow-up), use `113` (same category)
- `Title`: Automatically prefixed with "Reply on: " followed by the original document title
- `Status`: Set to `recno:2` (default document status)
- `Files`: Attached files uploaded by the manager
- `Contacts`: The recipient of the reply. `Role`: set to `recno:6` (Recipient). `ReferenceNumber`: The ContactRecno of the original document's sender. `IsUnofficial`: Inherited from the original document's sender contact

#### Reply with Email
When the manager chooses to reply via email, an email is sent directly to the sender
of the original document with optional carbon copies.

**Functionality:**
- To: Email address of the original document's sender (auto-populated)
- CC: Optional comma-separated list of email addresses
- Subject: Free-text field for email subject
- Message: Free-text field for email content
- Attachments: File upload component for adding attachments

#### Sign Off Document
When the manager has reviewed a document and no formal reply is required, they can
sign off the document with a response code and optional remarks.

**SIF API Method:**  
`DocumentService/SignOffDocument`

**Request Example:**
```json
{
  "Document": "25/10059-10",
  "ResponseCode": "TO",
  "Note": "Reviewed and noted. No further action required."
}
```

**Notes:**
- `Document`: The document number of the document being signed off
- `ResponseCode`: The response code indicating the type of sign-off. TO: "Til Orientering"
- `Note`: Free-text remarks from the manager regarding the sign-off 