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