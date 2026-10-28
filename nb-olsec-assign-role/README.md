# OneLake table-reader role assignment

This README covers [onelake_table_reader.ipynb](./onelake_table_reader.ipynb). The notebook creates or extends **one OneLake data access role per run**, granting `Read` access to specified lakehouse tables and adding Microsoft Entra security groups as role members.

## Inputs required from the administrator

Obtain and verify the following values before running the notebook. Enter them in its first code cell.

| Parameter | Required value | Where to obtain it / what to confirm |
| --- | --- | --- |
| `workspace_id` | Fabric workspace ID (GUID). | Obtain the ID from the target workspace URL or its administrator. Confirm this is the workspace containing the lakehouse, not the workspace hosting the notebook if they differ. |
| `lakehouse_id` | Target lakehouse item ID (GUID). | Obtain the item ID from the lakehouse URL or its administrator. Use the lakehouse ID, not a SQL analytics endpoint ID. |
| `name_of_role` | Name of the OneLake data access role to create or extend. | Have the administrator approve the exact role name. If it already exists, review its current permissions and members before proceeding. |
| `tables` | A single string containing one or more table names, separated by commas. | Have the lakehouse administrator supply the exact table names, including schemas where applicable, and approve the requested access. |
| `security_group_ids` | A Python list of Microsoft Entra security group object IDs (GUIDs). | Obtain each group's **Object ID** from Microsoft Entra ID or its administrator. Do not use group display names, email addresses, application/client IDs, or tenant IDs. Confirm group membership is appropriate for the requested data access. |


## Administrator prerequisites and approvals

Before handing the notebook to the operator, the administrator should:

1. Confirm the target workspace and lakehouse exist and that OneLake security/data access roles are available and configured for that lakehouse.
2. Confirm the identity running the notebook is authorized to create, retrieve, and overwrite data access roles on the target lakehouse. A group being added as a reader does not itself authorize the operator to manage roles.
3. Confirm the intended tables exist, the security groups are correct, and the proposed access is approved.
4. Confirm the groups belong to the tenant used by the notebook's Fabric authentication. The notebook takes `tenant_id` from the signed-in identity's token and uses that tenant for **every** supplied group; there is no separate tenant input.
5. Review and retain a copy of any existing role definition before allowing an overwrite. Record the approved permissions and members so changes can be checked and reversed manually if needed.
6. Confirm the preview API used by this sample is acceptable for the environment. The notebook sends requests with `preview=true`.

No client secret, password, or manually supplied access token is required as a configuration input.

## Execution and role-update behavior

1. Review the notebook, its limitations above, and the administrator-approved configuration in a non-production environment.
2. Replace all parameter placeholders and run the parameter cell.
3. Run the create-or-extend cell only after approving the intended changes:
   - It obtains a Fabric API token and derives the tenant ID.
   - It prints a proposed role containing table paths, the `Read` action, and the supplied groups.
   - It attempts to create the role with `dataAccessRoleConflictPolicy=Abort`.
   - If the create request fails, it attempts to retrieve the named role, merge new permission values into its **first decision rule**, add missing group members, and submit the role with `dataAccessRoleConflictPolicy=Overwrite`.
4. Inspect the printed HTTP status codes and response bodies. A printed response does not by itself mean the operation succeeded.
5. Retrieve the resulting role and verify its table paths, actions, and members against the approved configuration.
6. Test access using a representative member of each intended group and confirm the resulting access is appropriate, taking account of other roles and permissions that identity already holds.

**Important:** Printing a payload does not pause execution or request confirmation. Both the create and overwrite requests execute automatically within the same cell. The failed-create branch runs for **any** failed create response, not just a role-name conflict, and does not check GET success before processing the response.

The merge is additive: it does not remove existing table permissions or members. Adding groups to an existing role gives those groups the role's existing permissions as well as the newly added permissions. Existing members also receive the added table permissions. Do not reuse a role unless this combined access is intended. Roles with complex or multiple decision rules require manual review; the sample only merges permissions into the first rule.

## Sample disclaimer and license

This is a demonstration sample, not a production-ready or officially supported Microsoft tool. It changes data access permissions. Review, validate, and test it before production use; the operator is responsible for authorization, security, recovery, and compliance.

See the notebook's full disclaimer and the [MIT License](./LICENSE).
