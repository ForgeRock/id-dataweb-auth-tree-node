![ID DataWeb Logo](/images/idwLogo.png)

# ID DataWeb Integration
### Identity Verification, Fraud Detection and Adaptive Authentication 
Document version: 1.0 (March 2019)

# Table of Contents
1. ID DataWeb / Forgerock Integrated Solution Overview
2. ID DataWeb Overview
3. ID DataWeb Verification Templates
4. ID DataWeb / Forgerock Technical Integration Overview
5. POC Setup - ID DataWeb and Forgerock Prerequisites 
6. POC Setup - Forgerock Authentication Tree
7. POC Setup - Testing Your Configuration

## 1. Solution Overview

You can now add flexible **real time identity verification and fraud prevention** policies to your Forgerock workflow using ID DataWeb's Attribute Exchange Network (AXN.) This solution helps your organization answer a tough question: "How do I verify that my users are really who they claim to be" across B2C, B2B and B2E use cases. Forgerock offloads the process of identity verification to ID DataWeb, where organizations can configure exactly what attributes need to be collected, what data sources should be used for verification, and how the results should be interpreted into Trust Scores and Policy Decisions across the industry's top identity verification providers. These verification techniques can be be tailored to meet regulatory requirements like KYC/AML, NIST 800-63-3, or EPCS (Electronic Prescriptions for Controlled Substances,) as well as enterprise use cases (Supply chain verification, enterprise password reset, privileged account provisioning.)

## 2. ID DataWeb Overview

ID DataWeb’s Attribute Exchange Network can serve as the central identity verification, decisioning and workflow hub for an organization's account opening, password reset, or ongoing re-verification requirements. The AXN is a multi-tenant SaaS platform that currently integrates with 70+ of the industry’s top verification services, across human identity, affiliations, and environmental risk – presenting a single management console, cross-vendor Trust Score, and Policy Engine. In addition - the AXN integrates out of the box with ForgeRock through industry standard OpenID Connect - allowing Forgerock customers to easily add tailored identity verification policies to their workflows. 

There are several key steps to an identity proofing process:

* **Identity Resolution** – Collect the required identity attributes from the end user. 
* **Identity Validation** – Validating that the data provided ties to a legal identity. Note – this step does NOT prove that the user on the other side of the computer is who they say they are.
* **Identity Verification** – Verifies that the user is who they are claiming to be. 
* **Fraud detection & prevention** – Process of assessing risk of the user during the identity proofing process. 

Based on this framework, ID DataWeb offers several ways to acheive Identity Verification:

![levels of proofing](/images/levels.png)

These solutions are now integrated into ForgeRock Identity Platform using the **Authentication Tree** from ForgeRock **Access Management**.

## 3. ID DataWeb Verification Templates
Verification Templates represent the best practices for identity verification, fraud prevention and adaptive authentication across the Attribute Exchange Network. Each template is in production with multiple ID DataWeb customers today, and has been preconfigured to address the most common problems enterprises face while establishing digital trust.

### Identity Verification Template: MobileMatch
This template verifies the end user's identity by sending a one time pin (OTP) to the user's personal phone, then triggering a mobile carrier reverse lookup to verify that the end user's claimed identity matches what is on record for that device. In addition - this template validates the legal identity through a series of bureau checks, and evaluates the environmental risk of the user's device, location and network. Other fraud or compliance checks (OFAC, AML, Watchlist, Deceased, PO Box, etc) can be added to this policy to meet regulatory requirements. 

![mobilematch graphic](/images/mobileMatch.png)

**Validation**
* Validate accuracy of legal identity across credit bureaus

**Verification**
* Validate personal phone possession (OTP or inline check)
* Validate the phone number provided by end user is the same as the user asserted identity

**Fraud Prevention**
* Check for identity fraud indicators (deceased, synthetic, non-residential) 
* Analyze environmental risk (device, location, network, user behavior) across full transaction

### Identity Verification Template: BioGovID
The BioGovID Verification Template verifies the user's identity by validating the authenticity of their government issued ID, then doing a biometric comparison between a selfie and the license image. In addition - this template validates the legal identity through a series of bureau checks, and evaluates the environmental risk of the user's device, location and network. Other fraud or compliance checks (OFAC, AML, Watchlist, Deceased, PO Box, etc) can be added to this policy to meet regulatory requirements.

![bioGovID graphic](/images/bioGovID.png)

**Validation**
* Validate authenticity of license with fraud / spoof detection
* Extract PII from license, Validate accuracy of legal identity

**Verification**
* Verify that the face in the selfie matches the face on the license

**Fraud Prevention**
* Check for identity fraud indicators (deceased, synthetic, non-residential)
* Analyze environmental risk (device, location, network, user behavior) across full transaction 

### Identity Verification Template: MobileMatch with Adaptive BioGovID Step Up
This Verification Template starts with MobileMatch (described above,) and conditionally falls back to BioGovID if the user cannot be verified. This demonstrates ID DataWeb's adaptive verification capabilities, where an enterprise can specify "fallback" capabilities based on the results of the primary verification technique. 

![adaptiveVerify graphic](/images/mobileMatch-bioGovID.png)

**Validation**
* Validate accuracy of legal identity across credit bureaus

**Verification**
* Validate personal phone possession (OTP or Real Time)
* Validate the phone number provided by end user is the same as the user asserted identity
* **If identity cannot be verified:** Step Up to GovID validation with biometric verification

**Fraud Prevention**
* Check for identity fraud indicators (deceased, synthetic, non-residential) 
* Analyze environmental risk (device, location, network, user behavior) across full transaction


## 4. ForgeRock & ID DataWeb Technical Integration Overview

![ForgeRock ID DataWeb integration](/images/diagramA.png)

### End User Flow - Technical Process Steps: 
1. User accesses application, and clicks “create account”
2. Application calls Forgerock (OpenID Connect or other method)
3. Based on the **Authentication Tree** policy, Forgerock makes an OpenID Connect request to the AXN with the client id for the desired verification policy 
4. Browser redirect to IDW hosted and customer branded verification form, challenging the user for all required attributes for identity verification (customizable by use case.) 
5. When the user clicks submit, the data is sent against one or many attribute verification services (per the verification policy.)
6. The results from each vendor are aggregated, and the overall IDW Trust Score is generated. Based on Customer’s policy, a PASS, FAIL or STEP UP decision is created.
7. AXN sends the OpenID Connect response to Forgerock, including test results, IDW Trust score, and policy decision
8. Forgerock makes a decision based on the policy decision returned from AXN, and routes the user to the next step in the business process (If Policy Decision == "APPROVE" -> provision account)
9. Once the provisioning or access management process is complete, Forgerock responds to the application. 


## 5. POC Setup - Prerequisites

### ID DataWeb AXN solution
For this integration, you must have an active Organization in AXN, with an active Verification Policy. To get started, please email sales@iddataweb.com.

Specifically, you will need:
* **OpenID Connect Discovery URL** - example: ```https://preprod1.iddataweb.com/preprod-axn/axn/oauth2/.well-known/openid-configuration```
* **Client ID** - Provided by ID DataWeb through the AXN Admin console. Uniquely identifies your Verification Policy. Example - ```forgerockVerificationPolicy```
* **Client Secret** - Provided by ID DataWeb through the AXN Admin console. Example - ```xyzer09g045y540hr0gdf09hj0495hjfegfdg```


### ForgeRock Access Management
This guide is targeting Access Management (AM) version 6.0+. It can be deployed using this set of [instructions](https://backstage.forgerock.com/docs/am/6/quick-start-guide/).

Access Management must have network connectivity to ID DataWeb's cloud solution.

## 6. POC Setup - Forgerock Authentication Tree

### Authentication Tree Diagram
You can create many types of authentication tree to match your specific deployment. Below are 2 variants than can typically be used for simple setups. The integration of the ID DataWeb authentication solutions is done via the OAuth 2.0 node.

#### Quick Demo Setup
For a **quick demo setup**, build this flow. It uses the **"Provision Dynamic Account"** node, that creates an account at the Identity Store that is connected to AM. Those accounts are deleted once the session expires.


![Tree with temporary accounts](/images/provision_dynamic.png)

#### Persistent Accounts
For a more realistic deployment, we recommend to use this flow. It uses the **"Provision IDM Account"** node to **permanently persist user accounts** in an IDM instance. IDM needs to be configured in Access Management.


![Tree with permanent accounts](/images/provision_idm.png)


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

## 7. Testing your configuration
Access Management provides a quick way to test your setup at this stage.
1. Trigger the chain using a browser by using its service name
Example: https://openam.mybank.com/openam/XUI/#login&service=idwVerify

2. This should automatically redirect you to the ID DataWeb Verification page via an OpenID Connect redirection


![Demo login page](/images/diagramB.png)


3. The user will be prompted to complete identity verification per the configured ID DataWeb Verification Policy. Once complete, the user will be redirected to the Access Management user portal.


![ForgeRock User Portal](/images/user_profile.png)


4. Logout of the User Portal using the top right menu.


