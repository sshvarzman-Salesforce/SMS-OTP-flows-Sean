# SMS Verification OTP flows

## Repository summary

This repository is a deployment package for two Salesforce flows that implement an SMS OTP verification process end-to-end:

1. `Send_SMS_Verification` generates a random one-time SMS verification code and sends it to the customer.
2. `Verify_SMS_Code_MFA` validates the code entered by the customer to confirm it matches the issued OTP.

Together, these flows support a secure SMS-based verification journey. The implementation is designed so the OTP is generated and delivered to the customer, and later validated, without exposing the raw verification code in the normal flow outputs used by agents or downstream process steps.

This repository packages SMS OTP verification assets from `MainSDOSean` for reuse in other Salesforce orgs.

## Included metadata

- `metadata/flows/Send_SMS_Verification.flow`
- `metadata/flows/Verify_SMS_Code_MFA.flow`
- `metadata/objects/MessagingSession.object` (contains `Input_Phone__c` field metadata)
- `metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition`
- `package.xml` (flows + `MessagingSession.Input_Phone__c` + `OTP_SMS`)

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

- **Dependencies included in this package**
  - `MessagingSession.Input_Phone__c` custom field metadata is included.
  - Conversation Message Definition `OTP_SMS` metadata is included.

- **Custom field dependency**
  - `Send_SMS_Verification` references `MessagingSession.Input_Phone__c`.
  - This package deploys the field (`MessagingSession.Input_Phone__c`).

- **Hardcoded Messaging Channel Id**
  - `Send_SMS_Verification` creates `MessagingEndUser` with `MessagingChannelId = 0MjHn000000PFypKAG`.
  - Create the messaging channel in each target org from scratch.
  - Then update the flow to use that target org channel Id where `MessagingChannelId` is set.

- **Messaging template/definition dependency**
  - `Send_SMS_Verification` calls `sendConversationMessages` with `messageDefinitionName = OTP_SMS`.
  - This repo includes metadata for `OTP_SMS` conversation message definition.

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
