# SMS Verification OTP flows

## Use AI-assisted deployment (recommended)

This package is easiest to deploy correctly with an AI coding assistant such as **Cursor** or **Claude Code**.

When using an AI tool, give it:
- this repository URL (Copy this URL)
- your Salesforce org alias
- this instruction: _"Deploy this package, activate the deployed messaging component, activate both flows, and handle dependencies listed in README."_

Why: this package includes org-dependent metadata (messaging channel + IDs + template dependencies), and an AI assistant can quickly adjust those values in your target org.

## Repository summary

This repository packages reusable Salesforce metadata for an end-to-end SMS OTP verification process:

1. `Send_SMS_Verification` generates and sends a one-time SMS verification code.
2. `Verify_SMS_Code_MFA` validates the code entered by the user.

The implementation is designed so OTP generation and verification are handled in flow logic without exposing raw verification details to agents.

Source org: `MainSDOSean`

## Included metadata

- `metadata/flows/Send_SMS_Verification.flow`
- `metadata/flows/Verify_SMS_Code_MFA.flow`
- `metadata/objects/MessagingSession.object`  
  (contains only `Input_Phone__c` field metadata, not full object deployment)
- `metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition`
- `metadata/messagingChannels/TEXT_US_12012775572.messagingChannel`
- `package.xml`

## Deploy to another org

```bash
sf project deploy start --metadata-dir . --target-org <TARGET_ORG_ALIAS>
```

## Recommended deployment order (for AI tools)

Use this order when guiding Cursor/Claude Code or any deployment automation:

1. Deploy base dependencies first:
   - `MessagingSession.Input_Phone__c` (from `metadata/objects/MessagingSession.object`)
   - `OTP_SMS` conversation message definition
   - `TEXT_US_12012775572` messaging channel
2. Deploy flows second:
   - `Send_SMS_Verification`
   - `Verify_SMS_Code_MFA`

Why this order matters:
- `Send_SMS_Verification` references both:
  - `MessagingSession.Input_Phone__c`
  - `messageDefinitionName = OTP_SMS`
- Deploying dependencies first avoids reference failures in stricter CI/deployment pipelines.

Example (explicit two-step deployment):

```bash
# Step 1: dependencies
sf project deploy start \
  --source-dir metadata/objects/MessagingSession.object \
  --source-dir metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition \
  --source-dir metadata/messagingChannels/TEXT_US_12012775572.messagingChannel \
  --target-org <TARGET_ORG_ALIAS>

# Step 2: flows
sf project deploy start \
  --source-dir metadata/flows/Send_SMS_Verification.flow \
  --source-dir metadata/flows/Verify_SMS_Code_MFA.flow \
  --target-org <TARGET_ORG_ALIAS>
```

## Mandatory post-deploy activation (important)

After deployment, instruct the AI tool to **activate all runtime assets**:

1. Activate messaging component:
   - Activate `OTP_SMS` conversation message definition in Setup.
2. Activate both flows:
   - `Send_SMS_Verification`
   - `Verify_SMS_Code_MFA`

Suggested AI instruction:

_“After deploy, activate `OTP_SMS` conversation message definition and activate `Send_SMS_Verification` + `Verify_SMS_Code_MFA` flows. Confirm all three are Active.”_

## Prerequisites and dependencies (important)

Review these before deploying to another org:

- **Dependencies included in this package**
  - `MessagingSession.Input_Phone__c` custom field metadata
  - Conversation Message Definition `OTP_SMS`
  - Messaging Channel metadata: `TEXT_US_12012775572`

- **Messaging channel dependency (new)**
  - `Send_SMS_Verification` creates `MessagingEndUser` with hardcoded:
    - `MessagingChannelId = 0MjHn000000PFypKAG`
  - This repository now includes the source Messaging Channel metadata:
    - `metadata/messagingChannels/TEXT_US_12012775572.messagingChannel`
  - In a target org, update `MessagingChannelId` in `Send_SMS_Verification` to the deployed/target channel Id.
  - The channel metadata also references:
    - `sessionHandlerFlow = Demo_Messaging_Omni_Channel_inbound_flow`
    - `sessionHandlerQueue = Demo_Messaging`
    - `messageDefinitionName = SDO_Messaging_ConversationAcknowledgement` (auto-response)
  - If these assets don't exist in target org, either create them or adjust the messaging channel configuration after deploy.

- **Custom field dependency**
  - `Send_SMS_Verification` reads `MessagingSession.Input_Phone__c`.
  - Only this field is deployed (not the full `MessagingSession` object model).

- **Messaging template/definition dependency**
  - `Send_SMS_Verification` calls `sendConversationMessages` with:
    - `messageDefinitionName = OTP_SMS`

- **Invocable action availability**
  - Flows call platform actions:
    - `generateVerificationCode`
    - `sendConversationMessages`
    - `verifyCustomerCode`
  - These require related Salesforce features/licenses in target org.

- **Object/feature assumptions**
  - `Send_SMS_Verification` expects launch from `VoiceCall` or `MessagingSession`.
  - It reads:
    - `VoiceCall.FromPhoneNumber`
    - `MessagingSession.Input_Phone__c`

- **Template override dependency**
  - `Verify_SMS_Code_MFA` includes:
    - `overriddenFlow = SvcCopilotTmpl__VerifyCode`
  - If this flow template is unavailable in target org, update/remove override.

## Source org details

- Org alias: `MainSDOSean`
- Flow names:
  - `Send_SMS_Verification`
  - `Verify_SMS_Code_MFA`
