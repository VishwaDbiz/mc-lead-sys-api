### Business Requirement: Retrieve Lead Details
**Requirement ID:** BR-001  
**Endpoint:** GET https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/lead(leadId)

**Business Objective**  
The purpose of this API is to retrieve detailed information about a specific lead using their unique lead ID. This functionality is essential for sales and marketing teams to access lead data quickly and efficiently, enabling better customer engagement and follow-up actions.

**API Contract**

| Field     | Type    | Required | Description |
|-----------|---------|----------|-------------|
| leadId    | integer | Yes      | Unique identifier for the lead. |

**Response Schema**

| HTTP Status | Field     | Type    | Description                          |
|-------------|-----------|---------|--------------------------------------|
| 200         | leadId    | integer | Unique identifier for the lead.     |
| 200         | firstName | string  | First name of the lead.             |
| 200         | lastName  | string  | Last name of the lead.              |
| 200         | email     | string  | Email address of the lead.          |
| 200         | phone     | string  | Phone number of the lead (optional).|
| 404         | code      | string  | Error code indicating the issue.    |
| 404         | message   | string  | Description of the error.           |

**Flow Implementation**
1. The system shall log a message at INFO level.
2. The system shall transform the payload using DataWeave.
3. The system shall invoke the sub-flow "get-lead-implFlow".
4. The system shall log a message at INFO level within the sub-flow.
5. The system shall transform the payload using DataWeave within the sub-flow.
6. The system shall query Salesforce for lead details.
7. The system shall log a message at INFO level after querying Salesforce.
8. The system shall end the sub-flow "get-lead-implFlow".
9. The system shall transform the payload using DataWeave after returning from the sub-flow.
10. The system shall log a message at INFO level.

**Field Mapping / Data Transformation**

| Source Field                | Target Field | Transformation Logic                                      |
|-----------------------------|--------------|----------------------------------------------------------|
| attributes.uriParams.leadId | vars.leadId  | Direct assignment from URI parameters to variable.       |
| vars.leadId                 | SQL Query    | Concatenation to form SQL query: "SELECT Id,FirstName,LastName,Company,Phone,Email FROM Lead where id = 'leadId'" |
| payload                     | Response     | Conditional transformation: if payload is empty, return {"message": "No Record Found"} else return payload. |

**Connected Systems**

| System     | Protocol           | URL / Destination                                           | Direction     | Purpose                          |
|------------|--------------------|-----------------------------------------------------------|---------------|----------------------------------|
| Salesforce | salesforce:query   | Salesforce_Config                                          | Bidirectional | To query lead details from Salesforce. |

---

### Business Requirement: Create Lead
**Requirement ID:** BR-001  
**Endpoint:** POST https://mc-lead-sys-api-kd7ppn.9675fv.sgp-s1.cloudhub.io/lead  

**Business Objective**  
The purpose of this API is to facilitate the creation of new leads in the system. By allowing users to submit lead information, the organization can effectively manage potential customers and enhance sales opportunities. This API will streamline the lead generation process and ensure that all relevant data is captured and stored in the Salesforce system.

**API Contract**  
| Field      | Type    | Required | Description |
|------------|---------|----------|-------------|
| firstName  | string  | Yes      | The first name of the lead. |
| lastName   | string  | Yes      | The last name of the lead. |
| email      | string  | Yes      | The email address of the lead. |
| phone      | string  | No       | The phone number of the lead. |
| company    | string  | No       | The company name associated with the lead. |

**Response Schema**  
| HTTP Status | Field   | Type    | Description                          |
|-------------|---------|---------|--------------------------------------|
| 201         | message | string  | Confirmation message for lead creation. |
| 201         | leadId  | integer | Unique identifier for the created lead. |
| 400         | code    | string  | Error code indicating the type of error. |
| 400         | message | string  | Detailed error message.              |

**Flow Implementation**  
1. The system shall log a message at INFO level.
2. The system shall transform the payload using DataWeave.
3. The system shall invoke the sub-flow "post-lead-implFlow".
4. The system shall log a message at INFO level within the sub-flow.
5. The system shall transform the payload using DataWeave within the sub-flow.
6. The system shall create a record in Salesforce.
7. The system shall route based on conditions (1 branch(es) + default).
8. The system shall execute the branch when a duplicate record is detected and raise an error (CUSTOM:DUPLICATERECORD).
9. The system shall execute the default branch otherwise.
10. The system shall transform the payload using DataWeave in the default branch.
11. The system shall log a message at INFO level in the default branch.
12. The system shall end the execution of the sub-flow "post-lead-implFlow".
13. The system shall log a message at INFO level after returning from the sub-flow.

**Field Mapping / Data Transformation**  
| Source Field       | Target Field   | Transformation Logic                          |
|--------------------|----------------|-----------------------------------------------|
| payload            | payload        | Store the entire incoming payload as is.     |
| vars.incomingPayload.firstName | FirstName      | Direct mapping from firstName to FirstName.  |
| vars.incomingPayload.lastName  | LastName       | Direct mapping from lastName to LastName.    |
| vars.incomingPayload.email     | Email          | Direct mapping from email to Email.          |
| vars.incomingPayload.phone     | Phone          | Direct mapping from phone to Phone.          |
| vars.incomingPayload.company   | Company        | Direct mapping from company to Company.      |
| payload.items               | message        | Maps to a message indicating lead creation success. |
| payload.items.id            | id             | Maps to the lead ID returned from Salesforce. |

**Connected Systems**  
| System     | Protocol                | URL / Destination                                           | Direction     | Purpose                          |
|------------|-------------------------|------------------------------------------------------------|---------------|----------------------------------|
| Salesforce | salesforce:create       | Salesforce_Config                                          | Bidirectional  | To create and manage lead records. |