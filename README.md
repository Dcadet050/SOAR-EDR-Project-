# Objective

In this project I will demonstrate the importance of a cloud hosted SOAR solution that will automate the incident response lifecycle. With the integration of LimaCharlie EDR with Tines and Slack on Vultr Cloud Infrastructure, the system provides automated alerts, endpoint isolation, and real time detection. This project shows how orchestration and automation can help improve SOC workflows while reducing response times to endpoint security incidents.
# Highlights

- Built an end-to-end SOAR workflow using LimaCharlie, Tines, Slack, and Email

- Created a custom LimaCharlie detection rule to detect LaZagne activity

- Included an analyst approval step prior to automatically isolating endpoints

- Automated the process of isolating endpoints via LimaCharlie’s API

- Tested the entire SOAR workflow and validated that the endpoints could be automatically isolated and prevented from accessing the network
# Tools Used:

- Vultr Cloud Platform
- LimaCharlie
- Tines
- Slack
- LaZagne

# Tools Overview & Definitions

LimaCharlie is a cloud-native Endpoint Detection and Response (EDR) platform that provides real-time endpoint monitoring, threat detection, telemetry collection, and remote response capabilities. It enables security teams to detect malicious activity, investigate incidents, and perform response actions such as isolating compromised endpoints.

Tines is a Security Orchestration, Automation, and Response (SOAR) platform that automates security workflows by connecting security tools through APIs and webhooks. It enables organizations to streamline incident response, automate repetitive tasks, enrich security alerts, and orchestrate actions across multiple security products.

Slack is a cloud-based collaboration and communication platform used by organizations for team messaging and notifications. In security operations, Slack can integrate with security tools to deliver real-time alerts, incident updates, and analyst approval requests, enabling faster communication and response during security incidents.

Vultr is a cloud infrastructure provider that offers virtual machines, networking, storage, and other cloud computing services. In this project, Vultr was used to host the Windows virtual machine that served as the monitored endpoint, providing a cloud-based environment for testing detection and automated incident response workflows.

LaZagne is an open-source credential recovery tool used to retrieve stored passwords from Windows applications and browsers. In this project, LaZagne was executed in a controlled lab environment to simulate credential access activity, allowing LimaCharlie to detect the behavior and trigger the automated SOAR workflow. The tool was used solely for authorized security testing and educational purposes.

# Project Diagram

<img width="741" height="682" alt="Screenshot 2026-08-02 at 12 05 24 AM" src="https://github.com/user-attachments/assets/c9cc6bd1-f46c-4a98-9dfa-7d44099da0a3" />

This image shows a full SOAR Playbook of the project, starting with the malicious tool (LaZagne) being executed on the Windows machine. LimaCharlie then monitors the endpoint activity and detects malicious behavior based on the detection rule that was configured. As soon as activity is detected, LimaCharkie creates an alert and sends the events to Tines using a webhook. 

Tines receives the event and extracts information from the alert which includes:
Time
Computer name 
Source IP
Process name 
Command line
File path
Sensor ID
Link to the detection (If applicable)

After processing, Tines immediately sends the incident details to Slack and email. This is to make sure that the security analyst is notified instantly through different forms of communication as soon as a threat is detected.	Next, Tines will prompt the analyst with a question asking “Isolate?”. If the analyst says “No” a message will be sent to Slack saying “The computer was not isolated. Please investigate.” If “Yes” was chosen then LimaCharlie will isolate the machine and send a message to the Slack channel stating the isolation status as well as the computer name.


# Phase 1 - Environment Setup:

Virtual Machine Configuration: 

<img width="622" height="367" alt="Screenshot 2026-08-02 at 1 20 10 AM" src="https://github.com/user-attachments/assets/f2d68606-52ce-4535-a0b6-affe43c3f501" />

  | Component | Configuration |
  |-----------|---------------|
  | Cloud Provider | Vultr |
  | Operating System | Windows Server 2025 Standard |
  | Region | New Jersey |
  | vCPU | 1 |
  | Memory | 2 GB RAM |
  | Storage | 55 GB SSD |
  | Purpose | Endpoint |

Firewall Configuration:

<img width="2048" height="1006" alt="Screenshot 2026-08-02 at 1 17 59 AM" src="https://github.com/user-attachments/assets/c7878896-aeb3-432c-9448-b4ed459be11c" />

| Protocol | Port | Action | Purpose |
|----------|------|--------|---------|
| RDP | 3389 | Allow | Remote administration only from my public IP |
| Any Other Traffic | Any | Deny | Blocks all unrequested inbound traffic |

By limiting RDP access to my public IP address, I ensure that only administrative machines can access the server, while all other traffic is blocked.

This configuration follows the principle of least privilege.

Why Vultr?

Vultr was chosen for its ability to provide:

- Fast deployment of Windows virtual machines
- Public IP connectivity
- Custom firewall groups
- Reliable cloud hosting infrastructure
- An isolated network environment for cybersecurity testing

Additionally, hosting the endpoint within the cloud environment allows the project to mimic the environment in which many modern organizations manage their remote systems.


# Phase 2 - Attack Simulation   

After creating the Windows Virtual Machine, the next step was to make sure the endpoint is prepared for detection testing. I created an installation key to install the LimaCharlie sensor. This enables endpoint monitoring and ensures response capabilities. LaZagne was also installed to simulate malicious credential access activity.    

<img width="2048" height="468" alt="Installation Keys vew cocs•" src="https://github.com/user-attachments/assets/9772c39b-fa32-4714-b7c8-dcafe8269672" />


Installing the LimaCharlie Sensor:

The LimaCharlie Windows sensor was downloaded from the LimaCharlie platform and installed on the virtual machine using PowerShell. The sensor successfully registered on the LimaCharlie cloud platform and began to collect telemetry from the endpoint.

Purpose:

Installing the sensor will allow the endpoint to:

- Monitor the endpoint for system activity

- Collect endpoint telemetry

- Send security events to the LimaCharlie platform

- Receive remote response actions to perform actions like isolating the endpoint from the network

Following installation, the sensor was visible on the LimaCharlie dashboard and ready to test for detection of malware.

<img width="1254" height="708" alt="Agent installed successfully" src="https://github.com/user-attachments/assets/5e276a57-a273-4fa6-9964-a00117899ea1" />

<img width="1250" height="474" alt="PS CUcersAdainistratceDonteade»  Thcp win xtl release 5 3 1 (1) exe-1" src="https://github.com/user-attachments/assets/3a29e266-3ff2-4eed-8115-4604d308a21c" />

Installing Lazagne:

In order to test the workflow, LaZagne was downloaded onto the Windows endpoint.

To execute LaZagne, the following command was used:

```powershell
.\LaZagne.exe all
```

Execution of LaZagne led to the detection of an event by LimaCharlie, which initiated the automated workflow on Tines.

Purpose:

LaZagne was used to simulate the techniques associated with credential access in order to:

- Generate telemetry from the endpoint
- Detect the simulated event with LimaCharlie
- Test the automated workflow on Tines
- Ensure notifications to analysts in Slack and Email
- Confirm that the endpoint was isolated remotely

<img width="1252" height="676" alt="The Lalagne Project" src="https://github.com/user-attachments/assets/e9db199c-33ba-466e-b958-018e71fa6906" />

# Phase 3 - Detection Engineering

The objective of this phase was to create the LimaCharlie detection rule that would identify the execution of LaZagne on the Windows endpoint and initiate the SOAR workflow automatically.

<img width="1252" height="658" alt="unknown" src="https://github.com/user-attachments/assets/46065c78-a9ae-4e81-981b-df815ecc9d4b" />

LimaCharlie Timeline showing the process creation events that resulted from the execution of LaZagne.exe. The timeline has been filtered to focus on the malicious process for investigation and rule development.

<img width="1268" height="566" alt="unknown" src="https://github.com/user-attachments/assets/e32f6a15-0762-434a-a07d-635864bff1e4" />

Raw process event generated by LimaCharlie. This event contains indicators such as the full path of the executable, the arguments that were passed to it, the username that started the process, the process ID, the parent process of the process of interest, and the SHA-256 hash of the executable.

<img width="1250" height="386" alt="unknown" src="https://github.com/user-attachments/assets/79c66187-859f-46d9-9739-c57369b6311b" />

Using the LimaCharlie Timeline, we can investigate the characteristics of these process events to find any that are unique to the LaZagne malware. As seen in the above events, attributes such as the file path, command line arguments, process information, and the operating system can be used to create a detection rule for this malware. The detection rule will be monitoring Windows for NEW_PROCESS and EXISTING_PROCESS events where:

- The file path ends with LaZagne.exe
- The command line ends with all
- The command line contains lazagne

<img width="1258" height="236" alt="unknown" src="https://github.com/user-attachments/assets/509bf7b7-2465-41d2-ab2f-b529d21daabf" />

The response configuration defines the alert that is generated when the detection rule is triggered. An alert can be configured with a detection name, severity level, description, author, and a MITRE ATT&CK tag to provide context for the alert.

<img width="1330" height="568" alt="Match  4 operations were evaluated with the following results" src="https://github.com/user-attachments/assets/61649fc4-e19a-482b-9189-4b93db3c8ca5" />

Evaluation results of the rule demonstrating all detection criteria successfully matched the observed event. Each condition evaluates to “true”, validating the logic of the rule prior to the deployment.

<img width="1178" height="610" alt="Detections wew bocs +" src="https://github.com/user-attachments/assets/41c5755a-445f-4a49-b52e-bfb70c40212b" />


The LimaCharlie Detections dashboard displays the newly generated detection. This detection is indicative of the successful installation of the endpoint telemetry, detection, and reporting components of LimaCharlie.

# Phase 4 - SOAR Workflow Development 

The objective of this phase was to develop an automated SOAR workflow in Tines in response to detections from LimaCharlie and to distribute those alerts to Slack and email.

<img width="1252" height="780" alt="unknown" src="https://github.com/user-attachments/assets/219f3651-0948-4891-b7c4-7c9cd4f25ff6" />

The workflow begins with a Webhook action that receives events from LimaCharlie. Whenever the custom detection rule within LimaCharlie is triggered, the alert and all related telemetry will be sent to Tines for processing. After receiving the webhook, I inspected the JSON that was sent to us. The JSON contained fields like title, hostname, username, command line, source IP address, sensor ID, timestamp, and a link to the LimaCharlie detection.

<img width="1252" height="658" alt="4 slack" src="https://github.com/user-attachments/assets/7186c00d-8323-4aa0-8117-e2dfcf8515e5" />

To enable notifications to be sent to Slack, the application has to be authorized on Slack.

<img width="1250" height="664" alt="unknown" src="https://github.com/user-attachments/assets/598752b1-4e97-4d62-95e7-6f676f320e87" />

An action was added to the workflow to send notifications to Slack whenever a new detection is made by LimaCharlie.

<img width="1232" height="790" alt="Message" src="https://github.com/user-attachments/assets/7986ae32-c63f-45b3-848c-56742cd3798e" />

The dynamic fields from the webhook payload were inserted into the Slack message using Tines variables. This allows each alert to automatically include information regarding the specific detection.

The following fields from the detection were used:

```text
Detection title - <<retrieve_detections.body.cat>>
Timestamp - <<retrieve_detections.body.detect.routing.event_time>>
Hostname - <<retrieve_detections.body.detect.routing.hostname>>
Source IP - <<retrieve_detections.body.detect.routing.int_ip>>
Username - <<retrieve_detections.body.detect.event.USER_NAME>>
File path - <<retrieve_detections.body.detect.event.FILE_PATH>>
Command line - <<retrieve_detections.body.detect.event.COMMAND_LINE>>
Sensor ID - <<retrieve_detections.body.detect.routing.sid>>
Detection link - <<retrieve_detections.body.link>>
```

<img width="1254" height="1124" alt="lest send a message" src="https://github.com/user-attachments/assets/f8aaaa04-651a-4a3b-9a35-6c41e4cd369f" />

The Slack action was tested using the previously captured webhook event to ensure that the message was formatted appropriately before it was published to the workflow.

<img width="1256" height="898" alt="unknown" src="https://github.com/user-attachments/assets/95fbf39d-1564-4f6f-bbba-305985b90538" />

The logs confirm that the workflow successfully submitted a request to the Slack API and included authentication credentials.

<img width="1252" height="1040" alt="O Copy request as cURt X" src="https://github.com/user-attachments/assets/103c4fd1-f22b-4333-bd89-2b1e00dae711" />

The successful response of 200 confirms to us that Slack accepted the request and sent the notification.

<img width="1244" height="798" alt="27 # alerts" src="https://github.com/user-attachments/assets/af1f2de3-4462-4bfe-810f-4584348523a8" />

The message that was sent to Slack upon completion of the detection contains detailed information about the detection and allows the analyst to jump to the LimaCharlie investigation page directly from Slack.

Email Integration:

<img width="1250" height="596" alt="unknown" src="https://github.com/user-attachments/assets/03569bf6-c8df-4883-a036-0508655cae01" />

An email action was configured to provide additional alert notifications.

<img width="1256" height="598" alt="unknown" src="https://github.com/user-attachments/assets/1dd9ef08-d16d-43ac-8838-bd099f57463b" />

The template was created with dynamic variables from the webhook payload to automatically include detection information from LimaCharlie into the email alerts.

<img width="1250" height="1134" alt="Test Send Email" src="https://github.com/user-attachments/assets/70bf10eb-248f-4981-9e54-abd4d246dec1" />

The email action was tested to ensure that the workflow completed successfully.

<img width="1256" height="766" alt="unknown" src="https://github.com/user-attachments/assets/35ba79a8-b4c8-43a3-b107-af99b4a7a351" />

The final email contains the same detection information as was sent to Slack but provides a secondary means of being alerted of these events.

# Phase 5 - User Approval
The User Prompt displays the details of the endpoint detection. The analyst must make a selection of whether to isolate the endpoint. If the analyst selects the Yes button, the endpoint will be isolated. If the analyst selects the No button, the analyst will proceed to the next stage of the workflow without isolating the endpoint and will receive a notification of such an action.
<img width="1264" height="1272" alt="DCADET-SOAR-EDR" src="https://github.com/user-attachments/assets/3f857d8f-884f-4fd7-aa98-7c0349fc5685" />

<img width="1252" height="652" alt="Inank you, you can now close this window" src="https://github.com/user-attachments/assets/83ec7599-9427-4dab-abf9-4c5bd4fde1d8" />

<img width="1256" height="524" alt="ecton Link htapplacharlie  longsObaec50-257 430-725-1645166110sensors894247-3921-410-62b-28785020330tele" src="https://github.com/user-attachments/assets/61d60905-18e0-474c-8b4c-85ebb2b6e6fc" />

Screenshot of the Slack message stating that the endpoint was not isolated.

# Phase 6 - Endpoint Isolation
 Isolation Workflow:

<img width="754" height="1100" alt="HITP Request" src="https://github.com/user-attachments/assets/f4b2356b-ca75-4a26-a400-6db711b03379" />


<img width="1252" height="1248" alt="DCADET-SOAR-EDR" src="https://github.com/user-attachments/assets/2d0788db-fa6d-415a-8f5c-dd89951b7a55" />

After the analyst clicks on the Yes button, the workflow will follow the containment branch.  Tines will send an isolation request to LimaCharlie, check the status of the isolation of the sensor, and create a notification that will be sent to Slack.

<img width="1250" height="662" alt="unknown" src="https://github.com/user-attachments/assets/91a1279a-ca41-4a3a-b9aa-657a8553ae25" />
The Isolate Sensor HTTP Request will send a POST request to LimaCharlie’s API with the Sensor ID of the endpoint to be isolated.
<img width="1252" height="720" alt="unknown" src="https://github.com/user-attachments/assets/316f5e01-acc7-4a4f-96fe-ebfe81282689" />
Figure 6.3 - Isolation Request Successful
The API returned the following JSON:

{
  "isolate_sensor": {
    "status": 200
  }
}

This response indicates that the endpoint was successfully isolated by LimaCharlie.
# Phase 7 - Verification & Notification

<img width="640" height="1140" alt="Send a message" src="https://github.com/user-attachments/assets/55e491de-af55-4c13-b02a-d5bd8d1602ec" />
The fields within the Slack notification will be dynamically populated with the results from the previous actions in the playbook.

<img width="1754" height="920" alt="Today 3160171atsensors8-94-249 3621-4650-18" src="https://github.com/user-attachments/assets/337f3491-9bf9-4bab-981f-9f28d7a55f0f" />
The computer: dcadet-soar-edr
has been isolated”)
After isolating the endpoint device, Tines will send a message to your Slack channel to indicate that the operation was successful.

<img width="1248" height="470" alt="0-34-71-11-12" src="https://github.com/user-attachments/assets/6c562d76-bef0-465f-9fbc-8e97d639b485" />
In this screen within LimaCharlie, the administrator can confirm that the endpoint has been successfully isolated from the network. The sensor remains online for management, but cannot communicate with other devices on the network.

<img width="1254" height="730" alt="unknown" src="https://github.com/user-attachments/assets/d23ebad6-2a33-467c-982a-5dae65846b96" />
Prior to isolating the endpoint, the “ping” command was successfully able to communicate with Google.com. After initiating the isolation request from LimaCharlie, those communication successes changed to “General failure” indicating that the endpoint is no longer able to communicate with other devices on the network.
# Full Tines Workflow
<img width="1106" height="1402" alt="unknown" src="https://github.com/user-attachments/assets/8f1a1a14-511e-43f5-84b5-1e41493a4d1f" />

# Conclusion

This project demonstrates the implementation of a Security Orchestration, Automation, and Response (SOAR) platform using LimaCharlie and Tines to automate the response to endpoint detection. The automation of these steps from detection to endpoint isolation, verification, and providing notifications to the security analysts via Slack will streamline the response to security detections. Overall, the SOAR implementation will improve the security analysts’ ability to respond to security detections.

## Skills Demonstrated

| Skill | Description |
|--------|-------------|
| SOAR Workflow Development | Designed and implemented an end-to-end incident response workflow using Tines. |
| Detection Engineering | Created a custom LimaCharlie detection rule to identify LaZagne credential access activity. |
| Endpoint Detection & Response (EDR) | Configured and managed LimaCharlie to monitor endpoints and respond to security events. |
| Security Automation | Automated detection, analyst notifications, endpoint isolation, and verification tasks. |
| API Integration | Configured authenticated HTTP requests to interact with the LimaCharlie REST API. |
| Webhook Configuration | Integrated LimaCharlie detections with Tines using webhook-based event ingestion. |
| Incident Response | Implemented automated endpoint containment following analyst approval. |
| Conditional Logic | Built workflow branches to handle analyst decisions and execute different response paths. |
| Slack Integration | Configured dynamic Slack notifications using workflow variables and event data. |
| Email Automation | Created automated email alerts containing detection details and incident information. |
| JSON Data Parsing | Parsed and utilized JSON payloads from webhook events throughout the workflow. |
| Endpoint Isolation | Automated network isolation of compromised endpoints using the LimaCharlie API. |
| Workflow Validation | Verified successful containment through API responses, EDR status, and connectivity testing. |
| Cloud Infrastructure | Deployed and managed a Windows Server virtual machine in Vultr for security testing. |
| Threat Simulation | Simulated credential access activity using LaZagne to validate detections and response actions. |
| Technical Documentation | Documented the architecture, workflow, implementation process, and validation results. |
































































