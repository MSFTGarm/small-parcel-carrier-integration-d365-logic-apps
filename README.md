# Small parcel carrier integration from D365 SCM via Azure Logic Apps

## Overview
Applies to: Dynamics 365 Finance, Dynamics 365 Supply Chain Management, Azure Logic Apps, Azure Functions, Azure Key Vault.
This solution combines Dynamics 365 Finance, Dynamics 365 Supply Chain Management, Azure Logic Apps, Azure Functions, and Azure Key Vault, to build an integration for Dynamics 365 Supply Chain Management (D365 SCM) with small parcel carriers (e.g., FedEx/UPS/DHL) using Azure Logic Apps and Azure services. It is applicable to any industry using small parcel carriers. 

## Introduction
This reference architecture describes an integration pattern for connecting Dynamics 365 Supply Chain Management (D365 SCM) with small parcel carriers (for example FedEx, UPS, DHL, local carriers) using Azure Logic Apps as the orchestration layer. The goal is to standardize how shipping requests are sent from D365, how carrier-specific APIs are called, and how results such as shipping labels, tracking numbers, void/cancel actions, and shipment status updates are returned—securely, reliably, and in a scalable way.
This architecture is a fit for organizations that ship small parcels and need to integrate D365 with one or more carriers, especially when they want a consistent approach across regions and carriers.
Typical use cases include:
-	Label generation and tracking number creation during packing/ship confirmation
-	Rate shopping (optional) and service selection based on rules (cost, SLA, destination)
-	Void/cancel shipment and re-labeling flows
-	Shipment status tracking (polling or event-based) and updating D365 shipment records
-	Supporting multiple carriers with a common canonical message model and reusable integration components
Industries where this is commonly applicable:
- Retail and eCommerce (B2C shipping volumes, returns)
- Wholesale distribution and 3PL operations
- Manufacturing (spare parts, service logistics, drop shipments)
- Healthcare / medical devices (time-critical shipments, compliance requirements)
- Any business with multi-country shipping and frequent carrier changes or onboarding needs
A successful carrier integration requires alignment across business, functional, and technical teams. Key stakeholders typically include:
- Business Owner / Logistics Lead (shipping policies, carrier contracts, operational rules)
- Warehouse Operations representatives (packing process, label printing, exceptions handling)
- D365 Functional Consultant (WMS/Transportation/Ship confirmation process design)
- D365 Technical Architect / Developer (extensions, integrations, data model, performance)
- Integration Architect (canonical model, orchestration, error handling, patterns)
- Azure/Platform Architect (Logic Apps, APIM, Service Bus, networking, scaling)
- Security / Identity team (Key Vault, certificates, secrets, RBAC, audit requirements)
- Carrier/3rd-party technical contacts (API specs, certification/testing, production enablement)
- Operations / Support (DevOps/SRE) (monitoring, alerting, runbooks, incident response)

## Architecture
The following diagram illustrates the architecture for the solution. 
[Diagram image here](https://github.com/MSFTGarm/small-parcel-carrier-integration-d365-logic-apps/blob/main/d365-small-parcel-carrier-integration-reference-architecture.png)
[Download a PowerPoint file with this architecture.](https://github.com/MSFTGarm/small-parcel-carrier-integration-d365-logic-apps/blob/main/d365-small-parcel-carrier-integration-reference-architecture.pptx) 

## Dataflow
1.	User creates a shipment and triggers an action (for example, packing completed).
2.	D365 sends a ship request (shipment + packages + addresses + service).
3.	Azure Key Vault credentials stored in D365 are used to access Azure Logic Apps.
4.	Request is sent directly to Logic Apps.
5.	Logic App validates and selects carrier/service.
6.	Logic App calls API Management.
7.	APIM routes to the correct carrier adapter (Function).
8.	Adapter maps canonical payload to carrier-specific API and sends request.
9.	Carrier returns label + tracking.
10.	Label is stored in Blob Storage; tracking stored with correlation id.
11.	Response is sent back to D365 (tracking + label reference).
12.	Monitoring and alerts capture end-to-end trace; failures go to DLQ.

## Components
The following components are used in the reference architecture.
- Dynamics 365 Supply Chain Management
- Azure Logic Apps (orchestration)
- Azure API Management (single façade endpoint, policies)
- Azure Service Bus (async decoupling, retries, DLQ)
- Azure Functions (carrier adapters + transformations)
- Azure Key Vault (secrets/certificates)
- Azure Storage (labels/ETD PDFs)
- Azure Monitor + Application Insights (observability)

## Scenario details
Many organizations running Dynamics 365 Supply Chain Management (D365 SCM) need to ship parcels through multiple small parcel carriers across regions, but carrier integrations are often implemented as point-to-point connections that are hard to scale, expensive to maintain, and fragile when carrier APIs change. Common pain points include inconsistent label formats, slow or unreliable label generation during peak shipping hours, limited monitoring and troubleshooting, and the effort required to onboard new carriers or shipping services without disrupting warehouse operations.
This reference architecture addresses those challenges by introducing a standardized integration layer using Azure Logic Apps for orchestration and workflow control, typically combined with services such as Azure API Management (single façade and routing), Azure Service Bus (decoupling and resiliency), Azure Key Vault (secure secret/certificate storage), and optionally Azure Functions (carrier adapters and transformations) and Azure Storage (label and document storage). The scenario demonstrates how D365 can send a canonical shipping request and reliably receive outputs like tracking numbers, shipping labels, ETD/commercial documents, and shipment status updates, while supporting retries, error handling, and end-to-end observability.
The customer’s goals are to reduce integration complexity, make carrier onboarding faster, improve reliability and throughput during shipping peaks, and give support teams clear visibility into failures and processing times. Benefits typically include a reusable architecture for multiple carriers, fewer production incidents caused by API changes, improved security and compliance for credentials and documents, and a smoother warehouse shipping process with predictable label creation and tracking updates.

## Potential use cases
This solution was created for a retail / distribution organization shipping a high volume of small parcels from warehouses and fulfillment locations using Dynamics 365 Supply Chain Management (D365 SCM) and multiple carrier services. It can also be applied to industries like retail, manufacturing, healthcare, telecommunications, government, travel, and facilities where parcel shipping is a core operational process and shipping integrations must be reliable and scalable.
It can be used by any organization who:
- Ships small parcels (B2C or B2B) and needs labels + tracking generated from D365 SCM
- Uses more than one carrier, operates in multiple countries, or expects to add/replace carriers over time
- Needs centralized security, monitoring, and error handling instead of point-to-point integrations
- Requires an integration approach that can handle peak volumes and carrier API variability without blocking warehouse operations
Additional fits include automotive (spare parts distribution), education (campus logistics / equipment shipments), nonprofit (distribution of goods), and media (equipment shipping for events/production). These scenarios are similar in that they involve frequent parcel shipments and the same core needs: label generation, tracking, reliability, and fast carrier onboarding. The main differences are usually compliance requirements (for example in healthcare or government) and volume patterns (seasonal peaks in retail).
You can use this solution to:
- Generate shipping labels and tracking numbers for multiple carriers using a consistent D365-to-Azure integration pattern
- Orchestrate carrier-specific workflows (rate shopping, service selection rules, ETD/commercial documents, void/cancel) with standardized error handling and retries
- Improve operational reliability and supportability through centralized monitoring, alerting, auditing, and scalable message-based processing

## Considerations
These considerations help implement a solution that includes Dynamics 365. Learn more at Dynamics 365 guidance documentation.

## Cost optimization
Cost optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see Overview of the cost optimization pillar.
This architecture’s run cost is the sum of (A) per-transaction charges and (B) provisioned capacity charges:
A) Per-transaction / usage-driven (mostly linear with volume)
These tend to scale roughly with “shipments processed”:
- Logic Apps Consumption: charged based on the triggers + actions you execute in the workflow. 
- API Management (Consumption): billed by API operations/requests (a single API operation within an HTTP request is the billing unit). 
- Service Bus (Standard): billed by messaging operations, plus there’s a subscription-level base charge. 
- Azure Functions (Consumption): billed by executions, execution time, and memory used. 
- Key Vault: billed by transactions/operations (for example, secret gets). 
- Blob Storage: driven by GB stored + operations/transactions + replication choice + data transfer. 
- Azure Monitor / Log Analytics: driven by data ingested and retention. 
B) Provisioned / capacity-driven (stepwise, not linear)
These introduce a “minimum monthly baseline” even at low traffic:
- Logic Apps Standard (single-tenant): you pay for the underlying compute capacity you provision (plus storage/connectors depending on design). 
- API Management (Basic/Standard/Premium tiers): you pay per unit; you scale by adding units (capacity increases proportionally). 
- Service Bus Premium: dedicated resources for predictable performance; no per-message transaction charges like other tiers (but you pay for the dedicated allocation). 
- Functions Premium: at least one instance allocated at all times (baseline capacity). 

## Implementing Small parcel carrier integration from D365 SCM via Azure Logic Apps
This reference architecture is implemented by configuring a secure, reusable integration layer in Azure and connecting it to shipping processes in Dynamics 365 Supply Chain Management. At a high level, you will implement (1) the Azure integration platform that orchestrates requests and routes them to the correct carrier, (2) the carrier adapters that translate a canonical shipment message into each carrier’s API, and (3) the D365 SCM touchpoints that trigger label creation and consume tracking/label results. The configuration steps typically split into platform setup (networking, identity, secrets, monitoring), messaging and orchestration (Service Bus + Logic Apps + API Management), carrier onboarding (credentials, endpoints, mappings), and D365 configuration to enable the end-to-end shipping flow.
The following tables (or procedures) describe the major configuration areas:
- Azure foundation: resource group(s), identity, Key Vault, monitoring/logging
- Integration runtime: Service Bus entities, Logic App workflows, API Management façade and routing
- Carrier onboarding: per-carrier settings, certificates/secrets, request/response mappings, test endpoints
- D365 SCM integration: triggering mechanism, data contract/canonical payload, label retrieval and status updates

//## Next step
//
//
//## Related patterns
//The following patterns are available to help guide your implementation of the set customer credit limits business process.
//- Pattern link 1
//-	Pattern link 2
//-	Pattern link n…

## Related resources
Review the following related architecture guides, solutions, and other guidance content:
- [Overview of Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview)

## Tags
Industries: Manufacturing (20-39), Wholesale Trade (50-51), Retail Trade (52-59)
Stakeholders: Transportation, Warehouse
Products: Dynamics 365 Finance, Dynamics 365 Supply Chain Management, Azure Logic Apps, Azure Functions, Azure Key Vault 

## Contributors
This article is maintained by Microsoft. It was originally written by the following contributors.
Principal author:
- [Oleksandr Baranov](https://www.linkedin.com/in/author-account/) | Solution Architect 

