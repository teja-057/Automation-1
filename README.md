# Azure DevOps User Management Pipeline

## Overview

This Azure DevOps pipeline automates user management for your ADO Organization and Projects using PowerShell and REST APIs. It simplifies manual operations by enabling:

- Adding users to the Azure DevOps organization
- Removing users from the organization
- Assigning users to a specific project and group
- Updating license if the user already exists

## Features

- Built using YAML templates for modularity
- Uses step-level templates
- PowerShell-based logic with REST API calls
- Simple UI-triggered runs with input parameters
- Configuration driven entirely by pipeline parameters

## Pipeline Workflow

   Main Pipeline (ado-user-management.yaml)
   ├── action: addOrgUser → runs templates/add-user-org.yaml
   ├── action: removeUser → runs templates/remove-user-org.yaml
   └── action: addUserToProject → runs templates/add-user-project.yaml


Each template contains PowerShell logic interacting with Azure DevOps REST APIs.

## Parameters

| Name                | Required | Description |
|---------------------|----------|-------------|
| `action`            | Yes      | Action to perform: `addOrgUser`, `removeUser`, or `addUserToProject` |
| `userEmail`         | Yes      | Email address of the user |
| `accountLicenseType`| Yes (for add actions) | License type: `stakeholder`, `express(for Basic)`, `advanced(for Basic + Testplans)` |
| `projectName`       | Yes (for `addUserToProject`) | Name of the ADO project |
| `groupType`         | Yes (for `addUserToProject`) | Group type like `projectReader`, `projectContributor`, etc. |

## How to Run

1. Navigate to Azure DevOps portal
2. Go to Pipelines > Run Pipeline
3. Select the `ado-user-management.yaml` pipeline
4. Choose parameter values:
   - Action type
   - User email
   - License type
   - Project and group name (if applicable)
5. Click Run

## Required Variables

| Variable   | Scope              | Secret | Description                                      |
|------------|--------------------|--------|--------------------------------------------------|
| `ADO_ORG`  | YAML or pipeline UI| No     | Your Azure DevOps organization name (e.g., `vialto`) |
| `ADO_PAT`  | Pipeline UI        | Yes    | Azure DevOps Personal Access Token with appropriate scopes |

> Set `ADO_PAT` as a **secret variable** using the pipeline UI under Variables.

## Template Structure
   ├── ado-user-management.yaml # Main pipeline
   └── templates/
      ├── add-user-org.yaml # Adds or updates user in the organization
      ├── remove-user-org.yaml # Removes user from the organization
      └── add-user-project.yaml # Assigns user to a project and group

Each template uses step-based scripting.

## Behavior and Error Handling

- `addOrgUser`:
  - Checks if user exists
    - If yes, updates the license
    - If no, adds user with selected license

- `removeUser`:
  - Checks if user exists
    - If yes, deletes the user
    - If no, exits without error

- `addUserToProject`:
  - Fetches project ID using project name
  - Adds user to project with license and group

## Security Notes

- All REST calls are authenticated using a secret PAT
- PAT is securely stored as a secret variable
- No credentials are hardcoded

## Limitations

- License values must match Azure DevOps license naming
- Only fetches a maximum of 2500 users per run

## References

- [Get list of projects API](https://learn.microsoft.com/en-us/rest/api/azure/devops/core/projects/list?view=azure-devops-rest-7.1&tabs=HTTP)
- [Add user to organization API](https://learn.microsoft.com/en-us/rest/api/azure/devops/memberentitlementmanagement/user-entitlements/add?view=azure-devops-rest-7.1&tabs=HTTP)
- [Remove user from organization API](https://learn.microsoft.com/en-us/rest/api/azure/devops/memberentitlementmanagement/user-entitlements/delete?view=azure-devops-rest-7.1)

