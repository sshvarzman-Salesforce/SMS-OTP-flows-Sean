# SMS OTP Verification Flows (Core Package)

This repo deploys only the core SMS OTP verification assets:

- `Sean_SDO_Send_SMS_Verification` flow
- `Sean_SDO_Verify_SMS_Code_MFA` flow
- `MessagingSession.Input_Phone__c` custom field
- `OTP_SMS` conversation message definition (SMS template)

It **does not** deploy any Agentforce planner bundle / agent actions.

Source org: `MainSDOSean`

---

## What this package is for

Use this package when you want reusable OTP verification flows and dependencies, while creating and managing your own Messaging Channel and agent configuration per org.

---

## Prerequisites

- Salesforce CLI installed.
- Authenticated target org.
- SMS Messaging Channel created manually in the target org.
- Required managed dependency for verify override:
  - `SvcCopilotTmpl__` (because `Sean_SDO_Verify_SMS_Code_MFA` overrides `SvcCopilotTmpl__VerifyCode`).

---

## Deploy

```bash
git clone https://github.com/sshvarzman-Salesforce/SMS-OTP-flows-Sean.git
cd SMS-OTP-flows-Sean
```

Deploy metadata:

```bash
sf project deploy start \
  --source-dir metadata/objects/MessagingSession.object \
  --source-dir metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition \
  --source-dir metadata/flows/Sean_SDO_Send_SMS_Verification.flow \
  --source-dir metadata/flows/Sean_SDO_Verify_SMS_Code_MFA.flow \
  --target-org <YOUR_ORG_ALIAS>
```

---

## Required post-deploy setup

### 1) Activate SMS template component (required)

Important: metadata deployment does **not** reliably activate the message definition in all orgs. Treat this as a mandatory manual step.

1. Setup → **Messaging Settings** → **Message Definitions**.
2. Find **OTP SMS**.
3. Click **Activate**.

### 2) Activate both flows

1. Setup → **Flows**.
2. Activate `Sean_SDO_Send_SMS_Verification`.
3. Activate `Sean_SDO_Verify_SMS_Code_MFA`.

### 3) Set channel ID input variable in send flow

Because every org has its own Messaging Channel, set it in the flow variable:

1. Setup → **Flows**.
2. Open `Sean_SDO_Send_SMS_Verification`.
3. Update variable `Input_Your_Messaging_Channel_ID` default value with your org’s Messaging Channel Id.
4. Save and activate the flow.

Why this matters:
- The send flow reads `Input_Your_Messaging_Channel_ID` when creating/using `MessagingEndUser`.
- This is the **single placeholder value** to update per org.
- Updating this value applies everywhere the flow references the channel id.

Quick verify:
- Open flow `Sean_SDO_Send_SMS_Verification`.
- Confirm `Create_Messaging_User -> MessagingChannelId` uses `Input_Your_Messaging_Channel_ID` (not a hardcoded Id).

---

## Included assets

| File | Purpose |
|---|---|
| `metadata/flows/Sean_SDO_Send_SMS_Verification.flow` | Generates OTP, resolves/creates MessagingEndUser, sends OTP SMS |
| `metadata/flows/Sean_SDO_Verify_SMS_Code_MFA.flow` | Verifies code entered by customer |
| `metadata/objects/MessagingSession.object` | Adds `Input_Phone__c` for messaging pre-chat phone capture |
| `metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition` | SMS template used by send flow |
| `package.xml` | Manifest for these core components |

---

## Troubleshooting

| Error | Likely cause | Fix |
|---|---|---|
| `Cannot find object: SvcCopilotTmpl__VerifyCode` | Managed dependency missing | Install `SvcCopilotTmpl__` |
| `Invalid MessagingChannelId` | Flow variable still placeholder or wrong Id | Update `Input_Your_Messaging_Channel_ID` in `Sean_SDO_Send_SMS_Verification` |
| OTP flow runs but no SMS sent | Template/flow not active | Activate `OTP_SMS` and both flows |
| `OTP_SMS` deployed but inactive | Message definitions not auto-activated by deploy | Activate `OTP_SMS` manually in Messaging Settings |