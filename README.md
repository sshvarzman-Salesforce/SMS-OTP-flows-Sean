# SMS OTP Verification for Agentforce Service Agent

Deploys a ready-to-use Agentforce Service Agent that verifies a customer's identity via SMS one-time passcode (OTP) before they can access any other topics. The agent also answers questions from your Knowledge base and can escalate to a live agent.

---

## What you get after deployment

A fully configured **Agentforce Service Agent** (`Agentforce_Service_Agent`) with three topics already wired up:

| Topic | What it does |
|---|---|
| **Customer Verification** | Runs first on every conversation. Sends a 6-digit OTP via SMS, asks the customer to read it back, and verifies it. Gives one retry on a wrong code, then escalates to a human if verification fails twice. |
| **General FAQ** | Searches your Salesforce Knowledge articles to answer customer questions. |
| **Escalation** | Transfers the conversation to a live human agent on request. |

---

## Before you start — complete checklist

Work through each item before deploying. Skipping any of these is the most common cause of deployment failures.

### Your Salesforce org must have these features enabled

- [ ] **Agentforce** — The AI agent platform. Check in Setup → Agentforce Studio.
- [ ] **Digital Engagement / Messaging** — Required for the SMS channel. Check in Setup → Messaging Settings.
- [ ] **SMS messaging channel already created** — The channel must exist in your org *before* you deploy. See "How to create an SMS channel" below.
- [ ] **Salesforce Knowledge** — Required for the General FAQ topic. Check in Setup → Knowledge Settings.
- [ ] **Service Cloud Voice** *(required only for telephony/phone channel)* — If you only plan to use web chat (MIAW) or messaging, you can skip this. If you want phone/voice support, this license must be enabled.

### These Salesforce packages must already be installed

This package depends on two Salesforce managed packages. Deployment will fail if either is missing.

- [ ] **Service Copilot Template** (namespace: `SvcCopilotTmpl__`) — Provides the base templates for the Verification, FAQ, and Escalation topics, and the `VerifyCode` flow that the `Verify_SMS_Code_MFA` flow overrides.
- [ ] **Employee Copilot** (namespace: `EmployeeCopilot__`) — Provides the `AnswerQuestionsWithKnowledge` action used by the FAQ topic.

To check if these are installed: Setup → Installed Packages. If they are missing, contact your Salesforce AE or admin — they are part of the Agentforce for Service licensing bundle.

### On your computer

- [ ] **Salesforce CLI** installed — Download from [developer.salesforce.com/tools/salesforcecli](https://developer.salesforce.com/tools/salesforcecli)
- [ ] **Authenticated to your target org** — Run `sf org login web --alias myOrg` and complete the browser login, then verify with `sf org display --target-org myOrg`.

---

## How to create an SMS channel (if you haven't yet)

1. In Salesforce Setup, search for **Messaging Settings**.
2. Click **New Channel** → select **SMS** → follow the wizard.
3. Once the channel is created, open it and copy the **Channel ID** from the browser URL. It looks like: `0MjHn000000PFypKAG` (the segment between `LiveMessageSetup/` and `/view`).
4. Save that Channel ID — you will need it in Step 3 of the post-deployment setup below.

---

## Deploy

Run these three commands in order from your terminal. Replace `<YOUR_ORG_ALIAS>` with the alias you used when you logged in.

### Step 1 — Clone this repository

```bash
git clone https://github.com/sshvarzman-Salesforce/SMS-OTP-flows-Sean.git
cd SMS-OTP-flows-Sean
```

### Step 2 — Deploy dependencies

```bash
sf project deploy start \
  --source-dir metadata/objects/MessagingSession.object \
  --source-dir metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition \
  --source-dir metadata/labels/CustomLabels.labels \
  --target-org <YOUR_ORG_ALIAS>
```

### Step 3 — Deploy flows

```bash
sf project deploy start \
  --source-dir metadata/flows/Send_SMS_Verification.flow \
  --source-dir metadata/flows/Verify_SMS_Code_MFA.flow \
  --target-org <YOUR_ORG_ALIAS>
```

### Step 4 — Deploy the agent

```bash
sf project deploy start \
  --source-dir metadata/genAiPlannerBundles/Agentforce_Service_Agent \
  --target-org <YOUR_ORG_ALIAS>
```

If any step fails, check the **Troubleshooting** section at the bottom of this page.

---

## After deployment — required setup steps

These steps must be done in your Salesforce org after the deploy commands succeed. The agent will not work correctly without them.

### Step 1 — Activate the SMS message template

1. Go to Setup → **Messaging Settings** → **Message Definitions**.
2. Find **OTP SMS** and click **Activate**.

### Step 2 — Activate the flows

1. Go to Setup → **Flows**.
2. Find `Send_SMS_Verification` → open it → click **Activate**.
3. Find `Verify_SMS_Code_MFA` → open it → click **Activate**.

### Step 3 — Set your Messaging Channel ID

The flow that sends the SMS needs to know which of your messaging channels to use.

1. Go to Setup → **Custom Labels**.
2. Find **SMS_Messaging_Channel_Id** and click **Edit**.
3. Replace the value `REPLACE_WITH_YOUR_MESSAGING_CHANNEL_ID` with the Channel ID you copied when you created your SMS channel.
4. Click **Save**.

No flow editing is needed — the flow reads this label automatically.

### Step 4 — Wire the authentication key in the agent

The OTP code generated by the Send action must be passed to the Verify action. You do this by creating a context variable that holds it between the two steps.

1. Go to **Agentforce Studio** → open **Agentforce Service Agent**.
2. Click **Variables** (or **Context Variables**) → **New Variable**:
   - Name: `authentication key`
   - Type: `Text`
   - Leave both checkboxes unchecked.
3. Click the **Customer Verification** topic → open the **Send SMS Verification** action:
   - Find the output `AuthenticationKey`
   - Set **Map to variable** → `authentication key`
4. Open the **Verify SMS Code MFA** action in the same topic:
   - Find the input `authenticationKey`
   - Set **Assign from variable** → `authentication key`
5. Save and activate the agent.

### Step 5 — (Web chat / MIAW only) Collect the customer's phone number

If you are using an embedded web chat (Messaging for In-App and Web), the flow needs to know the customer's phone number. Do the following in your org:

1. Add a **pre-chat form** to your messaging channel that asks for the customer's phone number.
2. In your inbound **Omni-Channel Flow** for that channel, map the phone number value from the pre-chat form to the `MessagingSession.Input_Phone__c` field.

For voice/phone channels, the phone number is read automatically from the call record — no extra setup needed.

---

## Verify it works

1. Open a test conversation with your Agentforce Service Agent (via MIAW or phone).
2. The very first thing the agent should do is say it is sending a verification code and run the Send SMS Verification action — **without you prompting it**. If it does not, check the topic instructions and make sure the Customer Verification topic is active and listed first.
3. You should receive an SMS with a message like: *"Your verification code is 123456. This code will expire in 5 minutes."*
4. Reply with the code. The agent should confirm your identity and ask how it can help.
5. Try an incorrect code — the agent should give you one more attempt with a new code.
6. On a second wrong code, the agent should escalate to a human agent.

---

## Troubleshooting

| Error | Likely cause | Fix |
|---|---|---|
| `Cannot find object: SvcCopilotTmpl__VerifyCode` | Service Copilot Template package not installed | Install `SvcCopilotTmpl__` managed package first |
| `Cannot find invocable action: streamKnowledgeSearch` | Knowledge not enabled | Enable Salesforce Knowledge in Setup |
| `Invalid MessagingChannelId` | Custom Label not updated | Complete Step 3 of post-deployment setup |
| `No consent record found` | Messaging consent not set up | Consent enforcement is disabled by this package — if you still see this error, check your org's messaging consent configuration |
| Flow deploys but SMS is never sent | Flows not activated, OR channel ID still a placeholder | Activate both flows (Step 2) and check the Custom Label value (Step 3) |
| Agent does not run verification first | Context variable not mapped | Complete Step 4 of post-deployment setup |
| Deployment fails with `Atlas__VoiceAgent` error | Voice/telephony not licensed in this org | Remove the `voiceDefinition` block and the `Telephony` surface block from `Agentforce_Service_Agent.genAiPlannerBundle` before deploying |

---

## Voice / phone channel notes

This agent is configured to work on voice/phone calls as well as messaging. Voice support requires **Service Cloud Voice** licensing. If your org does not have this license:

- Remove the `<plannerSurfaces>` block with `<surfaceType>Telephony</surfaceType>` and the `<voiceDefinition>` block from `metadata/genAiPlannerBundles/Agentforce_Service_Agent/Agentforce_Service_Agent.genAiPlannerBundle` before deploying.
- The agent will still work on web chat and messaging channels.

If you do use voice, update the outbound route name `Outbound_route_VoiceCall_to_Queue` in the bundle to match your org's Omni-Channel queue flow.

---

## What is included in this package

| File | Purpose |
|---|---|
| `metadata/flows/Send_SMS_Verification.flow` | Generates the OTP, looks up or creates the MessagingEndUser, and sends the SMS |
| `metadata/flows/Verify_SMS_Code_MFA.flow` | Validates the code the customer provides |
| `metadata/conversationMessageDefinitions/OTP_SMS.conversationMessageDefinition` | The SMS message template ("Your verification code is…") |
| `metadata/labels/CustomLabels.labels` | `SMS_Messaging_Channel_Id` — holds your org's Messaging Channel ID |
| `metadata/objects/MessagingSession.object` | Adds `Input_Phone__c` field to MessagingSession (used for MIAW phone capture) |
| `metadata/genAiPlannerBundles/Agentforce_Service_Agent/` | The complete Agentforce Service Agent with all topics, instructions, and actions |
| `package.xml` | Deployment manifest |

Source org: `MainSDOSean`