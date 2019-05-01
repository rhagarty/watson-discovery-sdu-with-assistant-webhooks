# watson-discovery-sdu-with-assistant-webhooks

* Use Assistant dialog for better chatbot experience
* Use "customer-care" skill provided by Assistant
* Use webhooks to search Disco if question is "anything-else" and reply with disco passages

# What is a webhook?

A webhook sends a POST request callout to an external application to perform a programmatic function. You can then invoke the webhook from one or more dialog nodes.

A webhook is a mechanism that allows you to call out to an external program based on something happening in your program. When used in a dialog skill, a webhook is triggered when the assistant processes a node that has a webhook enabled. The webhook collects data that you specify or that you collect from the user during the conversation and save in context variables, and sends the data to the Webhook request URL as an HTTP POST request. The URL that receives the webhook is the listener. It performs a predefined action using the information that is provided by the webhook as specified in the webhook definition, and can optionally return a response.

# Differences between Webhooks and simple Cloud Functions

* No code required inside dialog node
* Webhooks requires setting once per skill, so can be used in multiple dialog nodes
* Setting action URL is easier (cut and paste). No need to parse and decode.
* Creds for webhooks in one place and easily set. No need to pass them into Assistant.

# TODO:

* Remove references to calling Disco directly
* Add action creds to .env
* Create better use case for invoking action from a real Assistant dialog node
* Format Disco results better in chatbot
* Use Assistant V2 API

# Prerequisites

* Create Watson Discovery collection as described in https://github.com/rhagarty/watson-discovery-sdu-ui
* Create Watson Assistant dialog skill as described in  https://github.com/rhagarty/watson-discovery-sdu-with-assistant-cloud-functions
* Create Watson Cloud Function action as described in https://github.com/rhagarty/watson-discovery-sdu-with-assistant-cloud-functions

# Steps:

## Enable webhook from Assistant

* Set up access to WebHook for the skill you created previously:

![](doc/source/images/assistant-define-webhook.png)

* In the cell you want to trigger action, click on `Customize`, and enable Webhooks for this node:

![](doc/source/images/assistant-enable-webhook-for-node.png)

* Then you can add params, change name of return variable, etc.

![](doc/source/images/assistant-node-config-webhook.png)

## Test in Assistant Tooling

* Using the `Try it` feature, add the context variable `my_creds` and see if it works (probably won't return anything meaningful in the test window, but you should NOT get an error due to credentials).

> Note: You must enter a response to trigger the assistant dialog node that calls the action.

![](doc/source/images/assistant-context-vars.png)

{"user":"7a4d1a77-2429-xxxx-xxxx-a2b438e15bea","password":"RVVEdpPFLAuuTwFXjjKujPKY0hUOEztxxxxxxxxxonHeF7OdAm77Uc34GL2wQHDx"}

These values are pulled from the `Functions` action panel, click on `API-KEY` which then takes you to the `API Key` panel, where the key is found:

```bash
7a4d1a77-2429-xxxx-xxxx-a2b438e15bea:RVVEdpPFLAuuTwFXjjKujPKY0hUOEztxxxxxxxxxonHeF7OdAm77Uc34GL2wQHDx
```

> Note: the value before the `:` is the user, and after is the password. Do not include the `:` in either value.

## Configure credentials

```bash
cp env.sample .env
```

Copy the `env.sample` file and rename it `.env` and update the `<***>` tags with the credentials from your Assistant service.

#### `env.sample:`

```bash
# Copy this file to .env and replace the credentials with
# your own before starting the app.

# Watson Discovery
ASSISTANT_WORKSPACE_ID=<add_assistant_workspace_id>
ASSISTANT_IAM_APIKEY=<add_assistant_iam_apikey>

# Run locally on a non-default port (default is 3000)
# PORT=3000
```

Credentials can be found by clicking the Service Credentials tab, then the View Credentials option from the panel of your created Watson service.

An additional `WORKSPACE_ID` value is required to access the Watson Assistant service. To get this value, select the `Manage` tab, then the `Launch tool` button from the panel of your Watson Assistance service. From the service instance panel, select the `Skills` tab to display the skills that exist for your service. For this tutorial, we will be using the `Custom Skill Sample Skill` that comes with the service:

<p align="center">
  <img width="300" src="doc/source/images/sample-skill.png">
</p>

Click the option button (highlighted in the image above) to view all of your skill details and service credentials:

![](doc/source/images/sample-skill-creds.png)

# Run locally

```bash
npm install
npm start
```

Access the UI by pointing your browser at `localhost:3000`.

Sample questions:

* **how do I set a schedule?**
* **how do I set the temperature?**
* **how do I set the time?**

# Sample Output

![](doc/source/images/sample-output.png)

## Access to results in application

* Results will be returned in Assistant context object:

```
{ conversation_id: "b59b187a-f4b7-4fe7-81ef-29073abbb8ee",
  system: 
   { initialized: true,
     dialog_stack: [ { dialog_node: 'root' } ],
     dialog_turn_counter: 2,
     dialog_request_counter: 2,
     _node_output_map: { node_15_1488295465298: [ 0 ] },
     last_branch_node: "node_2_1467831978407",
     branch_exited: true,
     branch_exited_reason: "completed" },
  webhook_result_1:
   { activationId: "88167283e985494b967283e985894b7e",
     annotations:
      [ { key: "path", value: "IBM Cloud Storage_dev/disco-action" },
        { key: "waitTime", value: 64 },
        { key: "kind", value: 'nodejs:8' },
        { key: "timeout", value: false },
        { key: "limits",
          value: { concurrency: 1, logs: 10, memory: 256, timeout: 60000 } } ],
     duration: 899,
     end: 1556222972039,
     logs: [],
     name: "disco-action",
     namespace: "IBM Cloud Storage_dev",
     publish: false,
     response:
      { result:
         { matching_results: 9,
           passages:
            [ { document_id: "3a5efee70d8cc9d70e2b94d22c15e2d1_2",
                end_offset: 2791,
                field: 'text',
                passage_score: 6.752501692678998,
```
