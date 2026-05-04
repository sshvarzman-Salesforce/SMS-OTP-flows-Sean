# SMS Verification OTP flows

This repository packages SMS OTP verification assets from `MainSDOSean` for reuse in other Salesforce orgs.

## Included metadata

- `metadata/flows/Send_SMS_Verification.flow`
- `metadata/flows/Verify_SMS_Code_MFA.flow`
- `package.xml` (Flow members)

## Included Agent Action exports

- `agent-actions/genAiFunctionDefinitions.json`
- `agent-actions/genAiPluginFunctionDefs.json`

These `agent-actions` files are exported via Tooling API because the related Agent Action artifacts are not retrievable as deployable source metadata with this CLI version.

## Deploy flows to another org

```bash
sf project deploy start --metadata-dir . --target-org <TARGET_ORG_ALIAS>
```

or (from another SFDX project):

```bash
sf project deploy start \
  --source-dir metadata/flows/Send_SMS_Verification.flow \
  --source-dir metadata/flows/Verify_SMS_Code_MFA.flow \
  --target-org <TARGET_ORG_ALIAS>
```

## Recreate / align Agent Actions in target org

1. Deploy both flows first.
2. In target org, create/update Agent Actions with labels:
   - `Send SMS Verification`
   - `Verify SMS Code MFA`
3. Point each action to the matching deployed flow (`InvocationTargetType = flow`).
4. If using plugins/planners, map according to IDs and links captured in:
   - `agent-actions/genAiFunctionDefinitions.json`
   - `agent-actions/genAiPluginFunctionDefs.json`

## Source org details

- Org alias: `MainSDOSean`
- Flow names: `Send_SMS_Verification`, `Verify_SMS_Code_MFA`
- Agent Action labels: `Send SMS Verification`, `Verify SMS Code MFA`
