# QuickBooks integration: sandbox vs production

Two things differ per org and are kept out of the shared flows/objects:

| Setting | Where it lives | Sandbox | Production |
|---|---|---|---|
| Which QuickBooks credential the flows call | `force-app/qb-env/<env>/externalServiceRegistrations/` | `QuickBooks_API_Sandbox` | `QuickBooks_API_Production` (not created yet) |
| Which QuickBooks company (realm) ID is queried | `force-app/qb-env/<env>/customMetadata/QuickBooks_Setting.Default.md-meta.xml` | 9341458629725147 | REPLACE_WITH_PRODUCTION_COMPANY_ID |

Always deploy by path (never `force-app` as a whole): `qb-env/sandbox` and `qb-env/production` would collide.

## After a sandbox refresh from production
The refreshed sandbox inherits production's credential reference and company ID, so repoint it before anything runs:
1. Deploy `force-app/qb-env/sandbox` to the sandbox (this resets the credential reference and company ID).
2. In Setup, re-authorize the sandbox QuickBooks External Credential principal if the login did not carry over (check this on the first refresh).
3. Make sure the Default Workflow User has the `QuickBooks_Integration_Admin` permission set.
4. Deactivate the Nightly flows until step 1-3 are confirmed, so the sandbox never calls production QuickBooks.
5. Test: create a QuickBooks Sync Request (Invoice) and check the Sync Log.

## Deploying to production (order)
1. Objects, fields, tabs, permission sets, reports (`qb-integration`, minus flows).
2. Sandbox Named/External Credential metadata (for refreshes) plus the new production pair (created in Setup; secrets entered by hand).
3. `qb-env/production` (fill in the real company ID first).
4. Flows, deployed inactive; activate one at a time. Nightly flows last.
5. Assign `QuickBooks_Integration_Admin` to the Default Workflow User; `QuickBooks_Read_Only` to everyone who should view.

Notes: scheduled flows cannot call QuickBooks directly, so the Nightly flows only create a Sync Request and `sMI_QuickBooks_Sync_Request_Run` (async path) does the call. Scheduled flow times use the Default Workflow User's time zone.
