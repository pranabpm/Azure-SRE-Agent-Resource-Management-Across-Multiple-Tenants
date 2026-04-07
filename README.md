# Azure-SRE-Agent-Resource-Management-Across-Multiple-Tenants
Azure SRE agent Demo to manage Azure resources across multiple tenants managed via Azure Lighthouse

Azure SRE Agent is an AI‑powered reliability assistant that helps teams diagnose and resolve production issues faster while reducing operational toil. It analyzes logs, metrics, alerts, and deployment data to perform root‑cause analysis and recommend or execute mitigations with human approval. Its capable of integrating with azure services across subscriptions and resource groups that you need to monitor and manage. Todays enterprise customers live in a multi tenant world and their are multiple reason to that due to aquisitions, complex corporate structures, managed service providers or IT partners. Azure Lighthouse enables enterprise IT teams and managed service providers to manage resources across multiple azure tenants from a single control plane. In this demo I will walk you through how to set up Azure SRE anent to manage and monitor multi tenant resoruces delegated through Azure Lighthouse.

Navigate to the Azure SRE agent and select Create agent. Fill in the required details along with the deployment region and deploy the SRE agent

<img width="1129" height="754" alt="image" src="https://github.com/user-attachments/assets/0229b669-3495-4702-b1dd-e2d9c9079736" />

Once the deployment is complete hit Set up your agent. Select the Azure resource you would like your agent to analyze like resource groups or subscriptions.

<img width="779" height="714" alt="image" src="https://github.com/user-attachments/assets/2a2e6c7f-b898-4ddf-9097-5a866ecb3408" />

This will land you to the popup window that allow you to select the subscriptions and resource groups that you would like SRE agent to monitor and manage. You can then select the subscriptions and resource groups under the same tenant, Great! As a MSP or ISV you have multiple tenants that you are managing via Azure Lighthouse and you need to have SRE agent access to those. 

So in order to demo this will need to set up Azure Lighthouse with correct set of roles and configuration to delegate access to management subscription where the Centralized SRE agent is running.

From Azure portal search Lighthouse
Navigate to the Lighthouse Home page and select “Manage your customers”. On My customers Overview select “Create ARM Template”

<img width="1048" height="579" alt="image" src="https://github.com/user-attachments/assets/a1c14e5a-8149-4b22-b6ab-01e2c842009f" />

Provide a Name and Description. Select subscription on Delegated scope. Select + Add authorization which will take you to Add authorization window. Select Principal type, which I am selecting User for demo purpose. The pop up window will allow Select user from the list.
<img width="1589" height="581" alt="image" src="https://github.com/user-attachments/assets/4fe6bf5a-be44-475a-a858-19c66224cb09" />
Select the checkbox next to the desired user who you want to delegate the subscription and hit “Select”
Then select the Role that you would like to assign the User from the managing tenant to the delegated tenant and select add. You can add multiple roles by adding additional authorization to the selected user. This step is very important to make sure the delegated tenant is assigned with the right role in order for SRE agent to add it as Azure source. 
<img width="1554" height="379" alt="image" src="https://github.com/user-attachments/assets/a761129c-bb39-4b95-b592-5622e347808b" />
Azure SRE agent requires as Owner or User Administrator RBAC role in order to assign the subscription in the list of managed resources. As per Lighthouse role support Owner role isn’t supported and User access Administrator role is supported ,but only for limited purpose (follow the documentation for additional information). If not defined correctly you might see an error stating:
“The 'delegatedRoleDefinitionIds' property is required when using certain roleDefinitionIds for authorization.”

<img width="464" height="267" alt="image" src="https://github.com/user-attachments/assets/a7b7f611-63ae-4863-9e38-5a8ebf0951e0" />

To allow a principalId to assign roles to a managed identity in the customer tenant, set its roleDefinitionId to User Access Administrator. While this role isn't generally supported for Azure Lighthouse, it can be used in this specific scenario. Download the template and add specific Azure built-in roles defined in the delegatedRoleDefinitionIds property. You can include any supported Azure built-in role except for User Access Administrator or Owner. Here is an example
{
    "principalId": "00000000-0000-0000-0000-000000000000",
    "principalIdDisplayName": "Policy Automation Account",
    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "delegatedRoleDefinitionIds": [
         "b24988ac-6180-42a0-ab88-20f7382dd24c",
         "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
    ]
}
In addition SRE agent would require certain roles at the managed identity level inorder to access and operate on those services. Locate SRE agent User assigned managed identity and add roles to the service principal. For the demo purpose I will assign Reader, Monitoring Reader, and Log Analytics Reader role.
<img width="1545" height="295" alt="image" src="https://github.com/user-attachments/assets/79cb30d0-c971-44ea-8060-7d0672f802b3" />
The sample template used for this demo can be found here.????
Login to the customers tenant and navigate to the service providers from the Azure Portal. From Service Providers overview screen select service provider offer from the left navigation pane.
From the top menu select the Add offer drop down and select Add via template
<img width="1289" height="354" alt="image" src="https://github.com/user-attachments/assets/249a53f9-0194-4b38-b0b5-81a5b5cb41d4" />
In the Upload Offer Template window drag and drop or upload the template file that was created in the earlier step and hit Upload. Once the file uploaded select Review + create. 
This will take a few minutes to deploy the template, and a successful deployment page should be displayed
<img width="1401" height="473" alt="image" src="https://github.com/user-attachments/assets/88b826c6-ae33-4740-af80-0df8b0148177" />
Navigate to Delegations from Lighthouse overview and validate if you see the delegated subscription and the assigned role
Once the Lighthouse delegation is setup sign in to the managing tenant and navigate to the deployed SRE agent. Navigate to Azure resources from top menu or via settings -> Managed resources. Navigate to Add subscriptions to select customers subscriptions that you need SRE agent to manage
<img width="1123" height="742" alt="image" src="https://github.com/user-attachments/assets/293cd558-a910-4dcb-b253-ef99a9b3242c" />
Adding subscription will automatically add required permission for the agent.
<img width="792" height="304" alt="image" src="https://github.com/user-attachments/assets/4b09c511-662a-4749-aa85-1056b4752f4b" />
Once the appropriate roles are added the subscriptions are ready for the agent to manage and monitor resources within them.
<img width="819" height="544" alt="image" src="https://github.com/user-attachments/assets/a3c73112-a958-41eb-8cc4-c2c8af06473f" />

