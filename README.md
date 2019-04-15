![ID DataWeb Logo](/images/xyz.gif)

# ID DataWeb Attribute Exchange Network Integration
Document version: 1.0 (March 2019)

## Solution Overview

You can now add flexible identity verification policies to your ForgeRock workflow using ID DataWeb's Attribute Exchange Network (AXN.) This solution helps you answer "How do I verify that my user's are really who they claim to be" across B2C, B2B and B2E use cases. ForgeRock offloads the process of identity verification to ID DataWeb, where customers can configure exactly what attributes need to be collected, what data sources should be used for verification, and how the results should be interpreted into Trust Scores and Policy Decisions across the industry's top identity verification providers. These verification techniques can be be tailored to meet regulatory requirements like KYC/AML, NIST 800-63-3, or EPCS (Electronic Prescriptions for Controlled Substances,) as well as enterprise use cases (Supply chain verification, enterprise password reset, privileged account provisioning.)

## ID DataWeb Overview

ID DataWeb’s Attribute Exchange Network can serve as the central identity verification, decisioning and workflow hub for an organization's account opening, password reset, or ongoing re-verification requirements. The AXN is a multi-tenant SaaS platform that currently integrates with 70+ of the industry’s top verification services, across human identity, affiliations, and environmental risk – presenting a single management console, cross-vendor Trust Score, and Policy Engine. In addition - the AXN integrates out of the box with ForgeRock through industry standard OpenID Connect - allowing ForgeRock customers to easily add tailored identity verification policies to their workflows. 

These solutions are now integrated into ForgeRock Identity Platform using the **Authentication Tree** from ForgeRock **Access Management**.

## ForgeRock & ID DataWeb Integration Overview

![ForgeRock ID DataWeb integration](/images/diagramA.png)

### Process steps: 
1. User accesses application, and clicks “create account”
2. Application calls ForgeRock (OpenID Connect or other method)
3. Based on the **Authentication Tree** policy, ForgeRock makes an OpenID Connect request to the AXN with the client id for the desired verification policy 
4. Browser redirect to IDW hosted and customer branded verification form, challenging the user for all required attributes for identity verification (customizable by use case.) 
5. When the user clicks submit, the data is sent against one or many attribute verification services (per the verification policy.)
6. The results from each vendor are aggregated, and the overall IDW Trust Score is generated. Based on Customer’s policy, a PASS, FAIL or STEP UP decision is created.
7. AXN sends the OpenID Connect response to ForgeRock, including test results, IDW Trust score, and policy decision
8. ForgeRock makes a decision based on the policy decision returned from AXN, and routes the user to the next step in the business process (If Policy Decision == "APPROVE" -> provision account)
9. Once the provisioning or access management process is complete, ForgeRock responds to the application. 


## Pre-requisites

### ID DataWeb AXN solution
For this integration, you must have an active Organization in AXN, with an active Verification Policy. To get started, please email sales@iddataweb.com.

Specifically, you will need:
* **OpenID Connect Discovery URL** - example: ```https://preprod1.iddataweb.com/preprod-axn/axn/oauth2/.well-known/openid-configuration```
* **Client ID** - Provided by ID DataWeb through the AXN Admin console. Uniquely identifies your Verification Policy. Example - ```forgerockVerificationPolicy```
* **Client Secret** - Provided by ID DataWeb through the AXN Admin console. Example - ```xyzer09g045y540hr0gdf09hj0495hjfegfdg```


### ForgeRock Access Management
This guide is targeting Access Management (AM) version 6.0+. It can be deployed using this set of [instructions](https://backstage.forgerock.com/docs/am/6/quick-start-guide/).

Access Management must have network connectivity to ID DataWeb's cloud solution.

## Create a new Authentication Tree

### Authentication Tree Diagram
You can create many types of authentication tree to match your specific deployment. Below are 2 variants than can typically be used for simple setups. The integration of the ID DataWeb authentication solutions is done via the OAuth 2.0 node.

#### Quick Demo Setup
For a **quick demo setup**, build this flow. It uses the **"Provision Dynamic Account"** node, that creates accounts directly to Access Management's Data Store. 


![Tree with temporary accounts](/images/provision_dynamic.png)

#### Persistent Accounts
For a more realistic deployment, we recommend to use this flow. It uses the **"Provision IDM Account"** node to **permanently persist user accounts** in an Identity Management (IDM). An IDM instance needs to be configured in Access Management.


![Tree with permanent accounts](/images/priovision_idm.png)


### Configure OAuth 2.0 node
In the authentication tree, select the OAuth 2.0 node. A form tab will open on the right. The information below shows a typical setup.

* **Node Name** - ```ID DataWeb Verification``` (Any name can be chosen)
* **Client ID** - ```<See pre-requisites>```
* **Client Secret** - ```<See pre-requisites>```
* **Authentication Endpoint URL** - Copy value from the entry **authorization_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
* **Access Token Endpoint URL** - Copy value from the entry **token_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
* **User Profile Service URL** - Copy value from the entry **userinfo_endpoint** in the **OpenID Discovery URL** web page (see the pre-requisites)
* **OAuth Scope** - ```openid```
* **Scope Delimiter** - ``` ```  (enter the space character)
* **Redirect URL** - Access Management redirect URL. Example: https://openam.mybank.com/openam/XUI/
* **Social Provider** - ```ID DataWeb Verification``` (Any name can be chosen)
* **Auth ID Key** - ```preferred_username```
* **Use Basic Auth** - ```enabled```
* **Account Provider** - ```org.forgerock.openam.authentication.modules.common.mapping.DefaultAccountProvider```
* **Account Mapper** - ```org.forgerock.openam.authentication.modules.common.mapping.JsonAttributeMapper```
* **Attribute Mapper** - ```org.forgerock.openam.authentication.modules.common.mapping.JsonAttributeMapper```
* **Account Mapper Configuration** - ```preferred_username=uid```
* **Attribute Mapper Configuration** - Enter the following entries:<br> 
  * ```given_name=givenName```
  * ```preferred_username=uid```
  * ```family_name=sn```
  * ```name=cn```
  * ```email=mail```
* **Save Attributes in the Session** - ```enabled```
* **OAuth 2.0 Mix-Up Mitigation Enabled** - ```disabled```
* **Token Issuer** - Copy value from the entry **issuer** in the **OpenID Discovery URL** web page (see the pre-requisites)

## Testing your configuration
Access Management provides a quick way to test your setup at this stage.
1. Trigger the chain using a browser by using its service name
Example: https://openam.mybank.com/openam/XUI/#login&service=idwVerify

2. This should automatically redirect you to the ID DataWeb Verification page via an OpenID Connect redirection


![Demo login page](/images/diagramB.png)


3. The user will be prompted to complete identity verification per the configured ID DataWeb Verification Policy. Once complete, the user will be redirected to the Access Management user portal.


![ForgeRock User Portal](/images/demo-userpage.png)


4. Logout of the User Portal using the top right menu.


