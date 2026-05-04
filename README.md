# SMS Verification OTP flows

This repository packages SMS OTP verification assets from `MainSDOSean` for reuse in other Salesforce orgs.

## Included metadata

- `metadata/flows/Send_SMS_Verification.flow`
- `metadata/flows/Verify_SMS_Code_MFA.flow`
- `package.xml` (Flow members)

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

## Source org details

- Org alias: `MainSDOSean`
- Flow names: `Send_SMS_Verification`, `Verify_SMS_Code_MFA`
