<pre>
## Creating webhook integrations for Automation

To send messages using webhooks during an automation, create an integration. Integrations can be invoked during an automation by using the aws:invokeWebhook action in your runbook. If you haven't already created a webhook, see Creating webhooks for integrations. To learn more about the aws:invokeWebhook action, see aws:invokeWebhook – Invoke an Automation webhook integration.
As shown in the following procedures, you can create an integration by using either the Systems Manager Automation console or your preferred command line tool.
Creating integrations (console)

To create an integration for Automation (console)
Open the AWS Systems Manager console at https://console.aws.amazon.com/systems-manager/.
In the navigation pane, choose Automation.
Choose the Integrations tab.
Select Add integration, and choose Webhook.
Enter the required values and any optional values you want to include for the integration.
Choose Add to create the integration.
Creating integrations (command line)

To create an integration using command line tools, you must create the required SecureString parameter for an integration. Automation uses a reserved namespace in Parameter Store, a capability of Systems Manager, to store information about your integration. If you create an integration using the AWS Management Console, Automation handles this process for you. Following the namespace, you must specify the type of integration you want to create and then the name of your integration. Currently, Automation supports webhook type integrations.
The supported fields for webhook type integrations are as follows:

Description
headers
payload
URL

Before you begin
If you haven't already, install and configure the AWS Command Line Interface (AWS CLI) or the AWS Tools for PowerShell. For information, see Installing or updating the latest version of the AWS CLI and Installing the AWS Tools for PowerShell.

To create an integration for Automation (command line)
Run the following commands to create the required SecureString parameter for an integration. Replace each example resource placeholder with your own information. The /d9d01087-4a3f-49e0-b0b4-d568d7826553/ssm/integrations/webhook/ namespace is reserved in Parameter Store for integrations. The name of your parameter must use this namespace followed by the name of your integration. For example /d9d01087-4a3f-49e0-b0b4-d568d7826553/ssm/integrations/webhook/myWebhookIntegration.

Linux 

aws ssm put-parameter \
    --name "/d9d01087-4a3f-49e0-b0b4-d568d7826553/ssm/integrations/webhook/myWebhookIntegration" \
    --type "SecureString" \
    --data-type "aws:ssm:integration" \
    --value '{"description": "My first webhook integration for Automation.", "url": "myWebHookURL"}'

Creating webhooks for integrations
When creating webhooks with your provider, note the following:
Protocol must be HTTPS.
Custom request headers are supported.
A default request body can be specified.
The default request body can be overridden when an integration is invoked by using the aws:invokeWebhook action.

</pre>
