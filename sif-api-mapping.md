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
