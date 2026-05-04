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

## Prerequisites and dependencies (important)

Review these before deploying to another org:

- **Custom field dependency**
  - `Send_SMS_Verification` references `MessagingSession.Input_Phone__c`.
  - If this custom field does not exist in target org, deployment/runtime will fail.

- **Hardcoded Messaging Channel Id**
  - `Send_SMS_Verification` creates `MessagingEndUser` with `MessagingChannelId = 0MjHn000000PFypKAG`.
  - This is org-specific and will usually be invalid in another environment.
  - Update the flow to the target org's Messaging Channel Id after deploy (or before deploy in source).

- **Messaging template/definition dependency**
  - `Send_SMS_Verification` calls `sendConversationMessages` with `messageDefinitionName = OTP_SMS`.
  - Ensure a matching messaging definition/template named `OTP_SMS` exists and is active in target org.

- **Invocable action availability**
  - Flows call platform actions:
    - `generateVerificationCode`
    - `sendConversationMessages`
    - `verifyCustomerCode`
  - These require the related Salesforce features/licenses to be enabled in target org.

- **Object/feature assumptions**
  - `Send_SMS_Verification` expects `VoiceCall` and `MessagingSession` records as launch context.
  - It reads `VoiceCall.FromPhoneNumber` and `MessagingSession.Input_Phone__c`.

- **Template override dependency**
  - `Verify_SMS_Code_MFA` includes:
    - `overriddenFlow = SvcCopilotTmpl__VerifyCode`
  - If this template flow is absent in target org, align/remove this override after deploy.

- **Agent Action mapping**
  - `agent-actions/*.json` include org-specific IDs (`PluginId`, `InvocationTarget`).
  - IDs are not portable; recreate actions and map by flow API name/label in target org.

## Source org details

- Org alias: `MainSDOSean`
- Flow names: `Send_SMS_Verification`, `Verify_SMS_Code_MFA`
- Agent Action labels: `Send SMS Verification`, `Verify SMS Code MFA`
