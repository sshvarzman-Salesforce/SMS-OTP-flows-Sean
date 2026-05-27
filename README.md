# SMS Verification + Customer Verification Agentforce Package

## What this package gives you

Deploying this repository gives you a reusable **Customer Verification** setup for Agentforce:

- Agentforce subagent/topic: **Customer Verification** (inside `Agentforce_Service_Agent`)
- Agent actions inside that subagent:
  - `Send SMS Verification`
  - `Verify SMS Code MFA`
- Flows used by those actions:
  - `Send_SMS_Verification`
  - `Verify_SMS_Code_MFA`
- Required dependencies:
  - `MessagingSession.Input_Phone__c` custom field
  - `OTP_SMS` conversation message definition
  - `TEXT_US_12012775572` messaging channel metadata

Source org: `MainSDOSean`

## Use Cursor or Claude Code (recommended)

The fastest way to deploy this correctly is with an AI coding agent (Cursor or Claude Code) that has access to your authorized Salesforce org.

Give the AI:
- this GitHub URL
- your target org alias
- the instruction below

Use this exact prompt:

_“Deploy this GitHub package to my target Salesforce org alias `<TARGET_ORG_ALIAS>`. Deploy dependencies first, then flows, then Agentforce planner bundle. Activate `OTP_SMS` conversation message definition, activate flows `Send_SMS_Verification` and `Verify_SMS_Code_MFA`, and verify the `Customer Verification` subagent in `Agentforce_Service_Agent` has both actions: `Send SMS Verification` and `Verify SMS Code MFA`.”_

## Included metadata

- `metadata/flows/Send_SMS_Verification.flow`
- `metadata/flows/Verify_SMS_Code_MFA.flow`
- `metadata/objects/MessagingSession.object` (scoped to `Input_Phone__c`)
- `metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition`
- `metadata/messagingChannels/TEXT_US_12012775572.messagingChannel`
- `metadata/genAiPlannerBundles/Agentforce_Service_Agent/*`
- `package.xml`

## Deployment steps (manual or AI-guided)

### Step 1: Deploy dependencies

```bash
sf project deploy start \
  --source-dir metadata/objects/MessagingSession.object \
  --source-dir metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition \
  --source-dir metadata/messagingChannels/TEXT_US_12012775572.messagingChannel \
  --target-org <TARGET_ORG_ALIAS>
```

### Step 2: Deploy flows

```bash
sf project deploy start \
  --source-dir metadata/flows/Send_SMS_Verification.flow \
  --source-dir metadata/flows/Verify_SMS_Code_MFA.flow \
  --target-org <TARGET_ORG_ALIAS>
```

### Step 3: Deploy Agentforce subagent/actions

```bash
sf project deploy start \
  --source-dir metadata/genAiPlannerBundles/Agentforce_Service_Agent \
  --target-org <TARGET_ORG_ALIAS>
```

## Post-deploy activation and checks (required)

1. Activate messaging component:
   - `OTP_SMS` conversation message definition
2. Activate both flows:
   - `Send_SMS_Verification`
   - `Verify_SMS_Code_MFA`
3. In Agent Builder, confirm:
   - planner bundle/agent: `Agentforce_Service_Agent`
   - subagent/topic: `Customer Verification`
   - actions present:
     - `Send SMS Verification`
     - `Verify SMS Code MFA`

## Important notes

- `Send_SMS_Verification` has a hardcoded `MessagingChannelId` value from source org. After deploy, update it to your target org's channel ID if different.
- Messaging channel metadata references additional assets (for example session handler flow/queue); align these in target org if needed.
- `Verify_SMS_Code_MFA` includes `overriddenFlow = SvcCopilotTmpl__VerifyCode`; update/remove if that template doesn't exist in the target org.
