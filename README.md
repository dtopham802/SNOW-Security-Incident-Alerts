# ServiceNow Security Incident Alerts
ServiceNow Security Incidents are security issues, like threats or vulnerabilities, within an organization that are managed through a structured, automated response workflow. The application integrates with xMatters to ingest alerts, prioritize issues using intelligent workflows and automation, and provides on-call notifications for investigation and remediation. xMatters helps to contain, eradicate, and recover from security events, minimizing impact by coordinating responses across IT and security teams 

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

# Pre-Requisites
* [ServiceNow Instance with Security Incidents Enabled](https://www.servicenow.com/products/security-incident-response.html)
* Have the [Everbridge Flow Designer app](https://store.servicenow.com/store/app/4f5cfd441b172e50c43e65b2604bcbad) installed and configured in your ServiceNow instance
* [Follow instructions](https://help.xmatters.com/ondemand/flowdesigner/servicenow-record-alerts.htm#InstallApp) and complete setup steps: Prepare ServiceNow, Configure xMatters, and Configure the Everbridge Flow Designer app in ServiceNow
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!
* ServiceNow API User with sn_si.admin role since the integration is reading and writing to the Security incident table.

# Files
* [ServiceNow Security Incident Alert.zip](ServiceNow-Security-Incident-Alert.zip) - The workflow zip for ServiceNow Security Incident Alerts.
* [ServiceNow Update Set](sys_update_set_xmatters_sir.xml) - The update set which includes the Business rule


# How it works
When a new Security Incident of a certain priority gets created, ServiceNow will trigger the business rule to send all the previous and current values of all the columns within the Security Incident table to the Everbridge Flow Designer app to then trigger an xMatters workflow. When the workflow is triggered, an on-call notification will be triggered to the assignment group and/or to the default recipients configured in the Trigger Profile in the app.

# Installation

## xMatters set up
1. Login to xMatters, navigate to the Workflow tab and import the [ServiceNow Security Incident Alert.zip](ServiceNow-Security-Incident-Alert.zip) workflow. Details [here](https://help.xmatters.com/ondemand/xmodwelcome/workflows/manage-workflows.htm#ImportExport)
2. Click on the **ServiceNow Security Incident Alert** workflow and then click the Flow Designer tab. Click on the **Security Incidents** canvas and then double click the **ServiceNow Record Alerts Security Incident [sn_si_incident]** Trigger step.
3. Click into the Endpoint Tab in the Trigger and set up your ServiceNow endpoint [following these instructions](https://help.xmatters.com/ondemand/integrationbuilder/configure-endpoints.htm?cshid=ServiceNowEndpoint#ServiceNowAuth).
4. In the Endpoint Tab, if you don't see the "Security Incident" table, double check the ServiceNow API user has the sn_si.admin rol. Confirm ServiceNow table "Security Incident [sn_si_incident]" is selected.
	
<img width="719" height="384" alt="image" src="https://github.com/user-attachments/assets/27c91616-4431-42dc-9295-295f6efcdf02" />

5. Click the Settings tab, select Basic Authentication, copy the URL and keep for future reference.

## ServiceNow set up
If you have already imported the ServiceNow XML update set skip to Step 7. To manually create a business rule, follow the steps below.

1. Navigate to System Definitions > Business Rules
2. Create a new Business Rule
   * Name: Enter a name for your business rule (i.e xMatters Security Incident Alerts)
   * Application: Everbridge Flow Designer
   * Table: Security Incident [sn_si_incident]
   * Enable Active
   * Enable Advanced
   
<img width="2280" height="470" alt="image" src="https://github.com/user-attachments/assets/134c4dc0-c2e5-4d7c-977f-4f5f2d7e4efe" />

3. Under "When to run" tab
   * When: Before
   * Order: 100
   * Enable Insert
   * Enable Update
   * Add filter conditions as shown in image below
	
<img width="1730" height="1272" alt="image" src="https://github.com/user-attachments/assets/e17842a8-4a38-45c0-9845-9a1aea46c8dd" />

4. Open "Advanced" tab; copy and paste the script below
   Make note of the triggerProfile value. The name will be used in step 9. 
   ```
   (function executeRule(current, previous /*null when async*/) {

	// Set up config
	let myConfig = {
	"triggerProfile": "Security Incident",// N.B.Matches name in Trigger Profile
	"signalMode": "State",
	"alertPriority": "Medium"
	};

	// Overwrite signalMode for Assignment
	if (current.operation()=='update' && current.assignment_group.changes()){
		myConfig.signalMode=='Assignment';
	}

	// Overwrite xMatters alert priority
	if (current.priority==1){
		myConfig.alertPriority=='High';
	}

	// Call Everbridge Flow Designer client, passing in the config
	let FlowDesignerClient = new x_xma_eb_fd.EBClient(config = myConfig);
	FlowDesignerClient.triggerWorkflow(current, previous);
	})(current, previous);
   ```
5.Click "Submit" to save the business rule
6. Navigate to System Applications > Application Cross-Scope Access
7. Create new cross scope privilege with the values in the screenshot
<img width="1874" height="568" alt="image" src="https://github.com/user-attachments/assets/2c940d9c-19d4-49b9-b708-ef6e7831507d" />

8. Navigate to Everbridge Flow Designer > Global Settings > Trigger Profiles
9. Click Create New
   * Name: The Trigger Profiles' name must match the "triggerProfile" value from step 4 (i.e Security Incident)
   * Credentials: Select the correct xMatters user credentials configured for the integration. This will enable a dropdown menu for the Workflow
   * Workflow: Select "ServiceNow Security Incident Alert" workflow
   * Trigger URL: Select "ServiceNow Record Alerts Security Incident [sn_si_incident]"
   * Default Alert Property: Medium
   * Default Signal Mode: (Optional)
   * Additional Recipients: (Optional)
   * ServiceNow API User: Select API user ([see pre req](https://github.com/dtopham802/SNOW-Security-Incident-Alerts))
<img width="2084" height="1320" alt="image" src="https://github.com/user-attachments/assets/9519dba1-87f1-4f4d-a49b-c043393a35a3" />

10. Click Submit to Save

## TEST
1. Navigate to Security Incident in the ServiceNow Navigator
2. Create a new Security Incident. Ensure the Priority is either Moderate, High, or Critical and the Assignment Group exsists in xMatters.
<img width="720" height="350" alt="image" src="https://github.com/user-attachments/assets/0b6bce3f-1a97-421e-bcb0-06fdc547780d" />

