# Production phase 1: foundation only (nothing runs)

Deploys objects, fields, tabs, permission sets, list views, reports, the (sandbox) Named/External Credential metadata, and the QuickBooks Setting type.
Does NOT deploy: flows, the External Service, the setting record, or any schedule. No data is created or changed.

Run in Terminal from `~/scaleMattersProject`, logged in as your own admin user (not claude@scalematters.com, which is read-only).

## 1. Log in to production (once)
    sf org login web --alias scaleMattersProdAdmin --instance-url https://login.salesforce.com

## 2. Validate only (changes nothing in production)
    sf project deploy start --dry-run --target-org scaleMattersProdAdmin \
      --source-dir force-app/qb-integration/objects \
      --source-dir force-app/qb-integration/tabs \
      --source-dir force-app/qb-integration/permissionsets \
      --source-dir force-app/qb-integration/reports \
      --source-dir force-app/main/default/namedCredentials/QuickBooks_API_Sandbox.namedCredential-meta.xml \
      --source-dir force-app/main/default/externalCredentials/QuickBooks_Online_Sandbox.externalCredential-meta.xml

Paste the full output. Do not add a test-level flag: with no Apex in the package, production runs no tests.

## 3. Real deploy (only after step 2 passes)
Same command without `--dry-run`.

## 4. After deploy
- Assign `QuickBooks_Integration_Admin` to yourself and the Default Workflow User; `QuickBooks_Read_Only` to viewers.
- Add the QB tab and related lists to the Account page yourself (existing customized pages are not touched).
- Share the QuickBooks report folder with anyone who needs the reports.

## Backing out
Nothing depends on phase 1 yet. Ask Claude to prepare a delete package if needed.
