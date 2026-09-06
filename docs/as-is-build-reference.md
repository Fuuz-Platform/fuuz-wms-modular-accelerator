# WMS Accelerator — AS IS Build Reference

> Converted from the v1.0.0 Word document. Full data model and data flow reference.

### 1. Executive Summary

### 2. Package Overview

### 3. Architecture Overview

### 3.1 Functional Areas & Module Map

### 3.2 End-to-End Warehouse Process Flow

### 3.3 Screen-Driven Flow Pattern

### 4. Core Components — Data Models

### 4.1 Receiving

### 4.2 Inventory & Product Management

### 4.3 Cycle Counting

### 4.4 Picking

### 4.5 Shipping & Carrier Management

### 4.6 Order & Load Management

### 4.7 Order Fulfillment & Business Partners

### 4.8 Site Management

### 5. Core Components — Data Flows

### 5.1 Receiving

### 5.2 Inventory & Product Management

### 5.3 Cycle Counting

### 5.4 Picking

### 5.5 Packing

### 5.6 Shipping & Carrier Management

### 5.7 Order & Load Management

### 5.8 Order Fulfillment & Business Partners

### 5.9 Quality (Material Review Board)

### 5.10 Configuration

### 5.11 Dashboard & Reporting

### 5.12 Testing & Utility

### 6. Core Components — Screens

### 6.1 Receiving

### 6.2 Inventory & Product Management

### 6.3 Cycle Counting

### 6.4 Picking

### 6.5 Packing

### 6.6 Shipping & Carrier Management

### 6.7 Order & Load Management

### 6.8 Order Fulfillment & Business Partners

### 6.9 Site Management

### 6.10 Configuration

### 6.11 Dashboard & Reporting

### 6.12 Testing & Utility

### 7. Document Designs

### 8. Application Configuration & Reference Data

### 8.1 Application Configurations

### 8.2 Status & Type Reference Data

#### 8.2.1 Order Status

#### 8.2.2 Order Type

#### 8.2.3 Order Line Release Status

#### 8.2.4 Order Line Release Type

#### 8.2.5 Load Status

#### 8.2.6 Pick Status

#### 8.2.7 Pick Step Status

#### 8.2.8 Unpick Reason

#### 8.2.9 Inventory Status

#### 8.2.10 Handling Unit Type

#### 8.2.11 Transaction Type

#### 8.2.12 Shipment Type

#### 8.2.13 Standard Carrier Alpha Codes (SCAC)

### 9. Status & Lifecycle Reference

### 9.1 Order Lifecycle

### 9.2 Receipt & Receiving Lifecycle

### 9.3 Pick & Pick Step Lifecycle

### 9.4 Load Lifecycle

### 9.5 Cycle Count Lifecycle

### 10. Sequences

### 11. Package Dependencies & Prerequisites

### 12. Glossary

### 1. Executive Summary

This document is the AS IS build description of the Fuuz WMS Accelerator, package version 0.0.7. It is the authoritative reference for the warehouse management functionality shipped in the package: the data models, data flows, screens, document designs, and reference data that make up a working warehouse management system on the Fuuz platform.

The WMS Accelerator provides end-to-end warehouse operations: receiving inbound shipments against purchase orders, managing inventory (including lots, handling units, and storage locations), cycle counting, order allocation and picking, packing, and outbound shipping — including carrier management, load consolidation, and shipping document generation (Bill of Lading, Commercial Invoice, Packing List, Pick Sheet, and Shipping Label).

The package is intended as a starting point for partners and prospective customers evaluating the Fuuz Industrial Intelligence Platform: it ships a complete, importable set of data models, screens, and data flows that can be run largely as-is, or extended and adapted to a specific warehouse’s processes. This document describes what the package contains and how it is structured; the companion Setup & Configuration Guide describes how to get it running in a new environment, and the companion Technical Overview provides a slide-level summary for a broader audience.

### 2. Package Overview

| Attribute | Value |
|---|---|
| Package Name | WMS Accelerator - Mfg |
| Package ID | appdev / build |
| Package Version | 0.0.7 |
| Platform Version | 2026.7.0 |
| Spec Version | 2.0.0 |
| Primary Module Group | Order Fulfillment, Inventory Management, Materials Management, Receiving, Site Management |

The package ships the following artifacts:

| Artifact Type | Items |
|---|---|
| Data Models (52) | Reference models covering orders, receipts, inventory, picks, packs, loads, shipments, cycle counts, business partners, and site/storage structure. |
| Data Flows (49) | Screen-bound and system flows that implement receiving, counting, picking, packing, shipping, and inventory adjustment transactions. |
| Screens (43) | Operator- and administrator-facing screens: tables, forms, widgets, and a live WMS dashboard. |
| Document Designs (6) | Printable shipping and warehouse documents: Bill of Lading, Box Content Label, Commercial Invoice, Packing List, Pick Sheet, Shipping Label. |
| Application Configurations (4) | Tenant-level configuration objects: Company Information, Move Inventory Options, Pack Options, Pick Options. |
| Sequences (19) | Auto-numbering sequences for receipts, picks, loads, shipments, counts, and related identifiers. |
| Reference Data Sets | Status and type lookups (Order/Load/Pick/Inventory status, carrier codes, handling unit types, and more), pre-populated as usable reference records. |

### 3. Architecture Overview

### 3.1 Functional Areas & Module Map

The package organizes its artifacts into functional areas that mirror the physical and operational flow of a warehouse. The table below summarizes how data models, data flows, and screens are distributed across these areas.

| Functional Area | Data Models | Data Flows | Screens |
|---|---|---|---|
| Receiving | 6 | 5 | 6 |
| Inventory & Product Management | 14 | 7 | 11 |
| Cycle Counting | 5 | 9 | 5 |
| Picking | 6 | 5 | 4 |
| Packing | 0 | 4 | 1 |
| Shipping & Carrier Management | 1 | 8 | 2 |
| Order & Load Management | 14 | 4 | 3 |
| Order Fulfillment & Business Partners | 2 | 1 | 7 |
| Site Management | 4 | 0 | 1 |
| Quality (Material Review Board) | 0 | 2 | 0 |
| Configuration | 0 | 1 | 1 |
| Dashboard & Reporting | 0 | 1 | 1 |
| Testing & Utility | 0 | 1 | 1 |

### 3.2 End-to-End Warehouse Process Flow

- Receiving — inbound orders/purchase orders are received against a Receipt, line by line (ReceiptLine), with exceptions captured via ReceiptException when quantities or products differ from expectation.

- Inventory & Product Management — received quantities become Inventory records tied to a Product, Lot, and StorageUnit/Area, tracked through HandlingUnit containers, with adjustments, merges, splits, and moves recorded via InventoryTrace.

- Cycle Counting — CountParameters define the scope of a Count; CountLine/CountLineInventory record expected vs. counted quantities, driving reconciliation without interrupting normal operations.

- Order & Load Management — Order/OrderLine records (Purchase or Sale) generate OrderLineRelease schedules; releases are allocated to Pick and Shipment records, which are grouped onto a Load for outbound transport.

- Picking — a Pick is broken into PickStep records against specific storage locations; NonConformance and Unpick capture exceptions when picked inventory cannot be fulfilled as planned.

- Packing & Shipping — picked inventory is packed into HandlingUnits, associated with Shipments and a Load, and released for transport with carrier assignment (StandardCarrierAlphaCode) and generated shipping documents.

- Quality (MRB) — Material Review Board flows (MRB Adjust Inventory, MRB Complete Return) handle non-conforming inventory and customer returns outside the standard pick/pack/ship path.

- Site Management & Business Partners — StorageZone/StorageUnit/Area define the physical warehouse layout; BusinessPartner/BusinessPartnerAddress hold the customers, suppliers, and carriers referenced throughout the other flows.

### 3.3 Screen-Driven Flow Pattern

Most data flows in the package are of type Screen: they are invoked directly by a screen action (a button, a form submit, or a table row action) rather than by a schedule or topic subscription. A typical screen-driven flow follows a consistent shape: a Request node receives the screen’s payload, Query nodes fetch supporting records, Validate/If-Else/Switch nodes branch on business rules, Mutate nodes write the resulting change, and a Response node returns the outcome to the screen, often paired with a Snackbar node to surface a success or error message to the user. Confirm and Form Dialog nodes are used where the screen needs to collect additional input or confirm a destructive action (such as canceling a count or unpicking a line) before the flow proceeds.

NOTE  Section 5 documents each data flow’s node composition (node names and types) as captured in the package. Node-level business-rule detail (specific JSONata conditions, field mappings) is available by opening the flow in the Data Flow Designer; this document describes structure and purpose, not full expression-level logic.

### 4. Core Components — Data Models

The package ships 52 Reference data models, grouped below by functional area. Each model section lists its model version, kind, module, and full field list as defined in the package.

### 4.1 Receiving

Receipt

Receipt - 1 Receipt per BOL/PackingSilp or ASN number.  A single receipt may have multiple line items being received.  Receipts with active exceptions can not be confirmed.

Kind: Reference  |  Module: orderReceiving  |  Model ID: receipt  |  Fields: 18

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | System generated sequence number. |
| asnNumber | String | — |
| bolNumber | String | — |
| carrier | String | — |
| externalId | String | — |
| note | String | — |
| trackingNumber | String | — |
| truckNumber | String | — |
| receivedAt | DateTime | The actual received at Date time - not the same as when this receipt was created. |
| confirmedAt | Date | — |
| receiptStatusId | ID! | — |
| receiptStatus | ReceiptStatus! | — |
| dockStorageUnitId | ID | — |
| dockStorageUnit | StorageUnit | — |
| orderId | ID | — |
| order | Order | — |
| receiptLines | [ReceiptLine!]! | — |

ReceiptException

Represents a report on received or expected inventory that was different from what was ordered; an exception to the order.

Kind: Reference  |  Module: orderReceiving  |  Model ID: receiptException  |  Fields: 15

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | System generated sequence number. |
| active | Boolean! | — |
| description | String | — |
| externalId | String | — |
| resolutionNotes | String | — |
| resolvedAt | DateTime | — |
| customData | JSON | — |
| integrationData | JSON | — |
| receiptLineId | ID! | — |
| receiptLine | ReceiptLine! | — |
| resolvedByUserId | ID | — |
| resolvedByUser | User | — |
| receiptExceptionReasonId | ID | — |
| receiptExceptionReason | ReceiptExceptionReason | — |

ReceiptExceptionReason

Reference model in the orderReceiving module (module group receiving).

Kind: Reference  |  Module: orderReceiving  |  Model ID: receiptExceptionReason  |  Fields: 4

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| usable | Boolean | — |
| name | String | — |
| receiptExceptions | [ReceiptException!]! | — |

ReceiptLine

Receipt Lines are specific products being received for a given Receipt.  Receipt lines are generally summarized, by product. Except for multiple Lots - if multiple lots of a single product, each lot must have its own receipt line.

Kind: Reference  |  Module: orderReceiving  |  Model ID: receiptLine  |  Fields: 24

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| numberOfInventory | Int! | Number of handling units received for this line - correlates to the number of labels/inventory that will be generated. Inventory.Quantity = receiptLine.quantity / HandlingUnits |
| addToHandlingUnit | Boolean! | — |
| externalId | String | — |
| note | String | — |
| number | String! | System generated sequence number. |
| quantity | Measure! | receipt quantity for this line item only. |
| productId | ID! | — |
| product | Product! | — |
| receiptId | ID! | — |
| receipt | Receipt! | — |
| receiptStatusId | ID! | — |
| receiptStatus | ReceiptStatus! | — |
| lotId | ID | — |
| lot | Lot | — |
| orderLineId | ID | — |
| orderLine | OrderLine | — |
| putAwayStorageUnitId | ID | — |
| putAwayStorageUnit | StorageUnit | — |
| receiptException | ReceiptException | — |
| storageUnitId | ID | — |
| storageUnit | StorageUnit | — |
| inventories | [Inventory!]! | — |
| receiptLineOrderLineReleases | [ReceiptLineOrderLineRelease!]! | — |

ReceiptLineOrderLineRelease

Reference model in the orderReceiving module (module group receiving).

Kind: Reference  |  Module: orderReceiving  |  Model ID: receiptLineOrderLineRelease  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| orderLineReleaseId | ID! | — |
| orderLineRelease | OrderLineRelease! | — |
| receiptLineId | ID! | — |
| receiptLine | ReceiptLine! | — |

ReceiptStatus

Reference model in the orderReceiving module (module group receiving).

Kind: Reference  |  Module: orderReceiving  |  Model ID: receiptStatus  |  Fields: 14

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| receiptExceptionUse | Boolean! | — |
| receiptLineUse | Boolean! | — |
| receiptUse | Boolean! | — |
| system | Boolean! | — |
| usable | Boolean! | — |
| code | String! | — |
| description | String | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| receipts | [Receipt!]! | — |
| receiptLines | [ReceiptLine!]! | — |
| receiptExceptions | [ReceiptException!]! | — |

### 4.2 Inventory & Product Management

Adjustment

The type of / reason for adjustment of an inventory's quantity value.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: adjustment  |  Fields: 10

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| name | String! | Unique identifier for this adjustment type |
| tolerance | Float! | Tolerance is the % - of the initial quantity that can be adjusted either up or down by the user during the process. Default value is 1.0, which equals 100%. |
| countLineInventoryUse | Boolean! | — |
| system | Boolean! | — |
| usable | Boolean! | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| lastInventories | [Inventory!]! | — |

Area

"Area" is defined as a physical, geographical, or logical grouping within a site. This grouping is determined by the site and can include various work centers such as process cells, production units, production lines, and storage zones. A larger section within the facility, often dedicated to a particular function or process.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: area  |  Fields: 14

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| active | Boolean! | — |
| process | Boolean! | These are areas where specific manufacturing processes take place, such as mixing, heating, or assembly. |
| productionCell | Boolean! | These are smaller, more specific areas within a process area where particular tasks or operations are performed. An Example maybe "CNC Department" within this Area you may have different Groups of CNCs such as Lathe, Mill, etc. which can be further broken down by child areas and then workcenter Groups as necessary. |
| productionLine | Boolean! | These are linear arrangements of equipment and workstations where products move through various stages of production. |
| storage | Boolean! | These areas are designated for storing raw materials, work-in-progress items, or finished goods. |
| description | String | — |
| externalId | String | — |
| name | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| storageZones | [StorageZone!]! | — |
| countParameters | [CountParameters!]! | — |

HandlingUnit

According to GS1, a **handling unit** is a physical unit that consists of packaging materials (such as load carriers or packaging material) and the goods contained within. It is always a combination of materials and packaging materials, designed to facilitate the handling, storage, and transportation of goods.  Handling units are uniquely identified using standards like the Serial Shipping Container Code (SSCC), which allows for efficient tracking and management throughout the supply chain.  [GS1 Logistic Label Guideline](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3)

Kind: Reference  |  Module: inventoryTracking  |  Model ID: handlingUnit  |  Fields: 27

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | — |
| serialNumber | String! | Handling Unit number, is a combination of the assigned sequence plus a prefix value. ~~The prefix value is determined by the mixed boolean.~~ Handling Units typically will recieve 1 of 2 prefixes - either designating that the contents are all the same, or they are mixed. |
| level | Int! | Handling Unit Level defines the number of levels away from the base inventory object this HU is. For example, if this HU is a case, containing 3 inventory objects, the inventory objects are level 0 and the HU would be level 1. If the HU is a pallet, with cases, which have inventory inside the cases, then the Pallet HU is level 3. Typically HU levels go up to 9. The default HU level value is 1. |
| mixed | Boolean! | The mixed flag is used to specify that the contents of this HU are mixed. Meaning not all the same product, or not all the same product.process. The state of this boolean will determine the code for the handling unit - Prefix value. |
| barcode | String | The barcode field is a full string typically for a 2D barcode of this handling unit. This field must be UNIQUE for all ACTIVE handling units. |
| note | String | — |
| gln | String | GLN (Global Location Number) for identifying locations |
| gtin | String | The **Global Trade Item Number (GTIN)** is a unique identifier used to identify trade items, including handling units, in the supply chain. Here are the key details about GTIN: ### Components of GTIN 1. **Company Prefix**: Assigned by GS1, this prefix identifies the company that created the GTIN [[1]](https://www.saplogisticsexpert.com/all-about-handling-units-and-bar-codes-in-sap-s4-hana/). 2. **Item Reference**: A unique number assigned by the company to identify the specific trade item [[1]](https://www.saplogisticsexpert.com/all-about-handling-units-and-bar-codes-in-sap-s4-hana/). 3. **Check Digit**: A digit calculated to ensure the GTIN is correctly composed [[1]](https://www.saplogisticsexpert.com/all-about-handling-units-and-bar-codes-in-sap-s4-hana/). ### Types of GTIN GTINs can vary in length depending on the application: - **GTIN-8**: Used for small items, typically scanned at point-of-sale. - **GTIN-12**: Commonly used in the United States, often seen on retail products. - **GTIN-13**: The most common type, used globally for retail products. - **GTIN-14**: Used for groupings of trade items not intended for point-of-sale scanning. ### Uses of GTIN - **Identification**: GTINs uniquely identify trade items, ensuring accurate tracking and management [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). - **Barcoding**: GTINs are encoded into barcodes like EAN/UPC, ITF-14, and GS1-128 [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). - **Supply Chain Efficiency**: GTINs enhance supply chain operations by providing a standardized method for identifying and tracking items [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). ### Benefits of GTIN - **Global Standardization**: GTINs are recognized worldwide, facilitating international trade [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). - **Accuracy**: Reduces errors in inventory management and product identification [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). - **Compliance**: Helps meet regulatory requirements in various industries [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). ### Example of GTIN A GTIN-13 might look like this: - **Company Prefix**: 1234567 - **Item Reference**: 890123 - **Check Digit**: 5 ### Implementation GTINs are typically encoded into barcodes and included on labels for handling units. These barcodes can be scanned to track the items electronically, ensuring real-time visibility and management [[2]](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3). Would you like to know more about how GTINs are used in specific industries or any other aspect of supply chain management? References [2] [GS1 Logistic Label Guideline](https://www.gs1.org/standards/gs1-logistic-label-guideline/1-3) |
| sscc | String | The **Serial Shipping Container Code (SSCC)** is a globally unique identifier used to track logistic units, such as cartons, pallets, or containers, throughout the supply chain. Here are the key details about SSCC: ### Components of SSCC 1. **Application Identifier (AI)**: Two digits that identify the type of data encoded into the SSCC barcode [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). 2. **Extension Digit**: A number between 0-9 that differentiates multiple SSCCs assigned by the same manufacturer [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). 3. **GS1 Company Prefix**: A 7-9 digit number that identifies the entity responsible for assigning the SSCC [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). 4. **Serial Reference**: A unique number assigned to each shipping container [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). 5. **Check Digit**: A digit that mathematically validates the accuracy of the SSCC [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). ### Uses of SSCC - **Tracking**: SSCCs are used to track logistic units from the point of origin to the destination, ensuring efficient and accurate shipment management [[2]](https://documents.gs1us.org/adobe/assets/deliver/urn:aaid:aem:494e625b-e1d8-4bbd-a1be-5918879cfc3d/An-Introduction-to-the-Serial-Shipping-Container-Code-SSCC.pdf). - **Identification**: Each SSCC acts like a license plate, uniquely identifying each logistic unit [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). - **Visibility**: SSCCs enhance supply chain visibility, allowing companies to monitor the status and location of shipments [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). ### Benefits of SSCC - **Efficiency**: Improves supply chain efficiency by enabling precise tracking and management of shipments [[2]](https://documents.gs1us.org/adobe/assets/deliver/urn:aaid:aem:494e625b-e1d8-4bbd-a1be-5918879cfc3d/An-Introduction-to-the-Serial-Shipping-Container-Code-SSCC.pdf). - **Accuracy**: Reduces errors in shipment handling and inventory management [[2]](https://documents.gs1us.org/adobe/assets/deliver/urn:aaid:aem:494e625b-e1d8-4bbd-a1be-5918879cfc3d/An-Introduction-to-the-Serial-Shipping-Container-Code-SSCC.pdf). - **Compliance**: Helps meet regulatory requirements in industries like food and pharmaceuticals [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). ### Example of SSCC An SSCC with a 7-digit GS1 Company Prefix can assign up to 10 billion unique SSCCs [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). Here’s a breakdown of how an SSCC is structured: - **AI**: 00 - **Extension Digit**: 3 - **GS1 Company Prefix**: 1234567 - **Serial Reference**: 890123456 - **Check Digit**: 5 ### Implementation SSCCs are typically encoded into a GS1-128 barcode and included on a GS1 Logistics Label [[1]](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes). These barcodes can be scanned to track the logistic units electronically, ensuring real-time visibility and management. Would you like to know more about how SSCCs are used in specific industries or any other aspect of supply chain management? References [1] [Serialized Shipping Container Codes (SSCC) | GS1 US](https://www.gs1us.org/upcs-barcodes-prefixes/serialized-shipping-container-codes) [2] [An Introduction to the Serial Shipping Container Code (SSCC) - GS1US](https://documents.gs1us.org/adobe/assets/deliver/urn:aaid:aem:494e625b-e1d8-4bbd-a1be-5918879cfc3d/An-Introduction-to-the-Serial-Shipping-Container-Code-SSCC.pdf) |
| externalId | String | This is to link this HU to another external system or app - this value must be unique for all active HU's. |
| customData | JSON | — |
| integrationData | JSON | — |
| length | Measure | — |
| width | Measure | — |
| height | Measure | — |
| volume | Measure | The volume occupied by the handling unit |
| weight | Measure | The tare weight of this handling unit when empty. |
| parentHandlingUnitId | ID | — |
| parentHandlingUnit | HandlingUnit | — |
| packagingConfigurationId | ID | — |
| inventories | [Inventory!]! | — |
| childHandlingUnits | [HandlingUnit!]! | — |
| loadId | ID | — |
| load | Load | — |
| handlingUnitTypeId | ID | — |
| handlingUnitType | HandlingUnitType | — |

HandlingUnitType

Reference model in the inventoryTracking module (module group inventoryManagement).

Kind: Reference  |  Module: inventoryTracking  |  Model ID: handlingUnitType  |  Fields: 4

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| description | String | — |
| name | String | — |
| handlingUnits | [HandlingUnit!]! | — |

Inventory

Inventory Model - Data changes are tracked for 25 Years.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: inventory  |  Fields: 45

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | — |
| serialNumber | String! | A unique number identifying this inventory - maybe a Lot or Serial. This will be used to set the Id field - using a system trigger. |
| quantity | Measure! | — |
| shippable | Boolean! | — |
| barcode | String | The full contents of a barcode - sometimes an RFID tag, or a 2D matrix. Making the need to parse that information on the UI unnecessary. This field must be unique for all active inventory. |
| trackingNumber | String | useful for any type of additional tracking information - perhaps from a carrier. Is not unique. |
| note | String | a user defined note field |
| grossWeight | Measure | gross weight of this inventory - quantity*product.weight + packaging.tareWeight |
| netWeight | Measure | net weight should be product.weight*quantity |
| expiresAt | DateTime | expiration date - determined by the shelf life days on the product in fuuz - this is based upon the receipt date typically - although, you can manually set this value in the system as well. |
| lastCountAt | DateTime | — |
| manufacturedAt | DateTime | the manufactured or born on date of the product. |
| workOrderId | String | — |
| externalId | String | externalId of another systems lot/serial number that should be referenced in Fuuz. If inventory is already labeled by another system, you should set this field, and the number of the inventory in Fuuz to be the same - also suggest encoding the barcode field as well, with any data that was generated outside of fuuz to prevent errors reading barcodes. |
| extProductId | String | This field can be used to provide a product code, in the event not all product codes exist in the wms schema. |
| extProcessId | String | This field can be used to provide a process code, in the event not all product codes exist in the wms schema. |
| customData | JSON | — |
| integrationData | JSON | — |
| handlingUnitId | ID | — |
| handlingUnit | HandlingUnit | — |
| inventoryStatusId | ID! | — |
| inventoryStatus | InventoryStatus! | — |
| lastTransactionTypeId | ID | — |
| lastTransactionType | TransactionType! | This is the ID of the last transaction performed to this inventory - historical transactions for this inventory will be updated in the data changes table. |
| lastAdjustmentId | ID | — |
| lastAdjustment | Adjustment | The last Adjustment made to this inventory |
| lotId | ID | — |
| lot | Lot | — |
| processId | ID | — |
| process | Process | — |
| productId | ID | — |
| product | Product | product code- optional - as there maybe times when the product is not setup in Fuuz yet - in those cases, set the extProduct field instead of this one. |
| receiptLineId | ID | — |
| receiptLine | ReceiptLine | — |
| sourceInventoryTraces | [InventoryTrace!]! | List of inventory Trace records, where this inventoryID was the source or origin (downstream) for the depletion / usage transaction. |
| destinationInventoryTraces | [InventoryTrace!]! | List of inventory Trace records, where this inventoryID was the destination (upstream) for the depletion / usage transation. |
| shipmentLineId | ID | — |
| shipmentLine | ShipmentLine | — |
| storageUnitId | ID | — |
| storageUnit | StorageUnit | — |
| countLineInventories | [CountLineInventory!]! | — |
| inventoryPickSteps | [InventoryPickStep!]! | — |
| mrbCaseId | ID | — |
| mrbHoldStatus | String | — |

InventoryPickStep

Reference model in the inventoryTracking module (module group inventoryManagement).

Kind: Reference  |  Module: inventoryTracking  |  Model ID: inventoryPickStep  |  Fields: 9

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| picked | Boolean | — |
| packedAt | DateTime | — |
| pickedAt | DateTime | — |
| allocatedQuantity | Measure | — |
| pickStepId | ID! | — |
| pickStep | PickStep! | — |
| inventoryId | ID | — |
| inventory | Inventory | — |

InventoryStatus

Inventory status controls the lifecycle of all inventory in the system.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: inventoryStatus  |  Fields: 12

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| inventoryUse | Boolean! | This flag determines if this status can be used on inventory (non-master). |
| locationUse | Boolean! | This flag determines if this status can be used as a "default" status on a location. |
| lotUse | Boolean! | — |
| system | Boolean! | — |
| usable | Boolean! | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| inventories | [Inventory!]! | — |
| lots | [Lot!]! | — |

InventoryTrace

Many:Many table for inventory transactions where depletions, splits or merges occur. Data change capture is not required, since this table represents the history itself that needs to be stored for a long time.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: inventoryTrace  |  Fields: 11

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| timestamp | DateTime! | Represents the point in time that the inventory transfer took place. |
| sourceInventoryId | ID! | — |
| sourceInventory | Inventory! | The inventory that has some quantity consumed by or transferred/subtracted from the transaction. (origin) |
| sourceQuantitySubtracted | Measure! | The amount of inventory removed from source |
| destinationInventoryId | ID! | — |
| destinationInventory | Inventory! | The inventory that some quantity is being transferred/added to |
| destinationQuantityAdded | Measure! | The amount of inventory added to destination |
| externalId | String | A unique ID to an external system |
| customData | JSON | — |
| integrationData | JSON | — |

LabelDesign

This model is designed to store legacy label designs in ZPL, IPL or other languages which can then be used in your custom flows rather than re-creating the labels from scratch in our label designer.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: labelDesign  |  Fields: 16

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. This field is system generated. |
| name | String! | Friendly name of the label design, must be unique and is required. |
| dpmm | Int! | The units of the label coordinate system are "dots", which are the smallest point which the printer is able to print. The physical size of a "dot" is different for different printer models, but the most common print density is 8 dpmm (8 dots per millimeter), or 203 dpi (203 dots per inch). Other print densities available include 6 dpmm (150 dpi), 12 dpmm (300 dpi) and 24 dpmm (600 dpi). |
| active | Boolean! | Specifies whether this label design format is active or not, inactive label designs should not be used in Flows. The default value for this field in any new record is true. |
| code | String! | Code is a system generated combination of name & revision for this model. The table triggers will update this field anytime the revision is updated. |
| description | String | Description of the label format/design. |
| format | String | Format is a free form text field where you will place the enter string of the label code format. This should include all the varables you're using for each element of the label as well. |
| revision | String! | Label Revision - This is required and defaults to 0, unless the user provides a specific value. This is combined with the name during the intiial 'create' or update mutation in order to generate the 'code' for this record.. Code = nameRevision to help with making this simpler to identify in a flow pattern. |
| variableData | String | — |
| lastUpdatedAt | DateTime | — |
| customData | JSON | — |
| integrationData | JSON | — |
| length | Measure | Length of the label |
| width | Measure | Width of the label |
| imageId | ID | — |
| image | Image | — |

Lot

Lots are logical groupings of inventory - usually from the same batch, heat or lot / production process.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: lot  |  Fields: 26

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String | The lot number is unique and required - user generated or from an external system. |
| countryOfOrigin | String | Where the lot was manufactured |
| externalId | String | — |
| hazardClassification | String | — |
| manufacturerId | String | Reference field for the manufacturer of this lot. |
| productionLineId | String | Reference field, to indicate the production line this lot was made from. |
| qcHoldReason | String | — |
| storageRequirements | String | — |
| supplierLotId | String | Supplier lot ID must be unique |
| bestBeforeAt | DateTime | — |
| expiresAt | DateTime | — |
| manufacturedAt | DateTime | — |
| receivedAt | DateTime | — |
| releasedAt | DateTime | When the lot was released. |
| customData | JSON | — |
| integrationData | JSON | — |
| inventoryStatusId | ID! | — |
| inventoryStatus | InventoryStatus! | — |
| productId | ID! | — |
| product | Product! | — |
| masterLotId | ID | — |
| masterLot | Lot | — |
| inventories | [Inventory!]! | — |
| subLots | [Lot!]! | — |
| receiptLines | [ReceiptLine!]! | — |

Process

Process Codes used to define the stage of inventory.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: process  |  Fields: 7

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | The descriptor of the inventory process stage. |
| system | Boolean! | — |
| usable | Boolean! | — |
| description | String | — |
| customData | JSON | — |
| inventories | [Inventory!]! | — |

Product

Product is a common data model should be used across Fuuz apps that require product, item, sku level information.

Kind: Reference  |  Module: inventoryTracking  |  Model ID: product  |  Fields: 33

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | True if this product is in use False if the record would otherwise be deleted Defaults to True |
| code | String! | A human-readable and unique shorthand name for the product |
| shelfLifeDays | Int! | Shelf life in calendar days for this product. Default value is 0; no expiration. |
| length | Measure | — |
| width | Measure | — |
| height | Measure | — |
| weight | Measure | — |
| name | String | Human-readable product name using full words Doesn't strictly have to be unique |
| description | String | Full description of the product. Usually about a paragraph, depending on the product. |
| number | String | Product/Model number |
| revision | String | Product/Model number revision |
| gtin | String | A GTIN (Global Trade Item Number) is a universally recognized identifier for a product. It is globally unique and customer-facing, allowing retailers and online marketplaces to easily identify what is being sold. |
| sku | String | An SKU (Stock Keeping Unit) is a company-specific identifier used for internal inventory management |
| upc | String | UPC (Universal Product Code) is a specific type of GTIN used in North America |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| defaultQuantityUnitId | ID! | — |
| defaultQuantityUnit | Unit! | The default unit to assign the quantity of inventory of this product |
| barcodeDesignId | ID | — |
| barcodeDesign | DocumentDesign | A reference to the document which specifies the barcode format to be used on this product |
| productCategoryId | ID | — |
| productCategory | ProductCategory | — |
| inventories | [Inventory!]! | — |
| orderLines | [OrderLine!]! | — |
| lots | [Lot!]! | — |
| receiptLines | [ReceiptLine!]! | — |
| forceHold | Boolean! | — |
| productPreferredStorageUnits | [ProductPreferredStorageUnit!]! | — |
| countLines | [CountLine!]! | — |
| countParameters | [CountParameters!]! | — |
| allowPickOverage | Boolean | — |

ProductCategory

Reference model in the inventoryTracking module (module group inventoryManagement).

Kind: Reference  |  Module: inventoryTracking  |  Model ID: productCategory  |  Fields: 8

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| usable | Boolean! | — |
| description | String | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| products | [Product!]! | — |

TransactionType

The unique name/code that identifies a transaction type

Kind: Reference  |  Module: inventoryTracking  |  Model ID: transactionType  |  Fields: 8

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| name | String! | — |
| usable | Boolean! | — |
| description | String | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| lastInventories | [Inventory!]! | — |

### 4.3 Cycle Counting

Count

Cycle counts for optimizing inventory movements. Cycle parameters are the filters used to query the system for inventory that should be counted - all parameters are "OR" statements. Any inventory matching any of the parameters will be included in the count.

Kind: Reference  |  Module: cycleCounting  |  Model ID: count  |  Fields: 14

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | Int! | — |
| blind | Boolean! | If Blind is True, the users will not be shown the expected quantities or expected weight information. If Blind is false, users will be provided the information during the count. |
| externalId | String | — |
| scheduledStartAt | DateTime | — |
| actualStartAt | DateTime | — |
| completedAt | DateTime | — |
| customData | JSON | — |
| integrationData | JSON | — |
| countLines | [CountLine!]! | — |
| countParameters | [CountParameters!]! | — |
| countStatusId | ID | — |
| countStatus | CountStatus | — |
| employee | String | — |

CountLine

Count lines are the aggregated expectations for the count based upon the dimensions/parameters provided when the count was defined. Each line represents a single product, in a single location with the expected quantity and weight to be counted.

Kind: Reference  |  Module: cycleCounting  |  Model ID: countLine  |  Fields: 12

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | Int! | — |
| expectedQuantity | Float! | Expected Quantity is calculated sum at the time the count is 'started'. This defaults to 0 upon initial creation. |
| expectedWeight | Float! | expected weight to be counted for this product in this location. |
| approvedAt | DateTime | Approved At date / time |
| countId | ID! | — |
| count | Count! | — |
| productId | ID! | — |
| product | Product! | — |
| expectedStorageUnitId | ID | — |
| expectedStorageUnit | StorageUnit | — |
| countLineInventories | [CountLineInventory!]! | — |

CountLineInventory

An individual line item for a specific inventory in a larger collection of line items called a "Count" (aka Cycle Count). A record in this table represents the measurement of a single inventory record that corresponds with a single count.

Kind: Reference  |  Module: cycleCounting  |  Model ID: countLineInventory  |  Fields: 17

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| note | String | — |
| adjustedAt | DateTime | Date time of any adjustment made to this entry. |
| quantity | Measure! | counted Quantity; should default to the inventory.quantity - unless the user enters a different quantity during the counting process. |
| weight | Measure! | the weight, of the inventory if doing count by weight. |
| countLineId | ID! | — |
| countLine | CountLine! | — |
| counterUser | User! | — |
| inventoryId | ID! | — |
| inventory | Inventory! | — |
| adjustingUserId | ID | — |
| adjustingUser | User | — |
| adjustmentId | ID | — |
| adjustment | Adjustment | — |
| counterUserId | ID | — |
| storageUnitId | ID | — |
| storageUnit | StorageUnit | — |

CountParameters

Count parameters define the filter criteria, that is used when the count is 'started' these parameters are used to determine what the expected products, locations and quantities are for the count.  If no results are found, the cound should be set to 'invalid' status.

Kind: Reference  |  Module: cycleCounting  |  Model ID: countParameters  |  Fields: 18

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | Int! | — |
| createdFromAt | Date | Created From Date (_gte) start of day selected |
| createdToAt | Date | (_lte) end of day selected |
| updatedFromAt | Date | Latest Updated At starts (_gte) beginning of day selected |
| updatedToAt | Date | last updated date/time is (_lte) end of day selected |
| countId | ID! | — |
| count | Count! | — |
| areaId | ID | — |
| area | Area | — |
| productId | ID | — |
| product | Product | — |
| productCategoryId | ID | — |
| productCategory | ProductCategory | — |
| storageUnitId | ID | — |
| storageUnit | StorageUnit | — |
| storageZoneId | ID | — |
| storageZone | StorageZone | — |

CountStatus

Reference model in the cycleCounting module (module group inventoryManagement).

Kind: Reference  |  Module: cycleCounting  |  Model ID: countStatus  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean | — |
| usable | Boolean | — |
| description | String | — |
| name | String | — |

### 4.4 Picking

NonConformance

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: nonConformance  |  Fields: 11

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean | — |
| note | String | — |
| affectedPickStepId | ID | — |
| affectedPickStep | PickStep | — |
| causePickStepId | ID | — |
| causePickStep | PickStep | — |
| handlingUnitId | ID | — |
| handlingUnit | HandlingUnit | — |
| inventoryId | ID | — |
| inventory | Inventory | — |

PickStatus

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: pickStatus  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| usable | Boolean | — |
| description | String | — |
| name | String | — |
| picks | [Pick!]! | — |

PickStep

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: pickStep  |  Fields: 16

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| droppedOff | Boolean | — |
| quantity | Measure | — |
| dropoffStorageUnitId | ID | — |
| dropoffStorageUnit | StorageUnit | — |
| orderLineReleaseId | ID | — |
| orderLineRelease | OrderLineRelease | — |
| pickId | ID | — |
| pick | Pick | — |
| pickStepStatusId | ID | — |
| pickStepStatus | PickStepStatus | — |
| pickupStorageUnitId | ID | — |
| pickupStorageUnit | StorageUnit | — |
| affectedNonConformances | [NonConformance!]! | — |
| causeNonConformances | [NonConformance!]! | — |
| inventoryPickSteps | [InventoryPickStep!]! | — |

PickStepStatus

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: pickStepStatus  |  Fields: 7

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| usable | Boolean | — |
| description | String | — |
| name | String | — |
| userId | ID | — |
| user | User | — |
| pickSteps | [PickStep!]! | — |

Unpick

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: unpick  |  Fields: 7

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| note | String | — |
| unpickTime | DateTime! | — |
| unpickReasonId | ID | — |
| unpickReason | UnpickReason | — |
| userId | ID | — |
| user | User | — |

UnpickReason

Reference model in the picking module (module group orderFulfillment).

Kind: Reference  |  Module: picking  |  Model ID: unpickReason  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| holdOnReturn | Boolean | — |
| description | String | — |
| name | String! | — |
| unpicks | [Unpick!]! | — |

### 4.5 Shipping & Carrier Management

StandardCarrierAlphaCode

Reference model in the shipping module (module group materialsManagement).

Kind: Reference  |  Module: shipping  |  Model ID: standardCarrierAlphaCode  |  Fields: 9

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | — |
| isParcel | Boolean | — |
| carrier | String | — |
| code | String | — |
| description | String | — |
| name | String | — |
| serviceType | String | — |
| loads | [Load!]! | — |

### 4.6 Order & Load Management

BusinessPartner

Reference model in the orderProcessing module (module group orderFulfillment).

Kind: Reference  |  Module: orderProcessing  |  Model ID: businessPartner  |  Fields: 25

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | Short Abbreviated code for business partner - must be unique. |
| creditLimit | Float | — |
| active | Boolean! | — |
| allowTransfer | Boolean! | — |
| customer | Boolean! | — |
| supplier | Boolean! | — |
| dunsNumber | String | — |
| externalId | String | — |
| internalNote | String | — |
| name | String | Longer / Legal name of the business partner. |
| packagingNote | String | — |
| paymentTerms | String | — |
| shippingNote | String | — |
| taxId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| firmHorizon | Duration | Firm Horizon - is the period of time, from now, where all newly created releases - with a dueAt inside the horizon will be set to Firm. Example DueAt is 1 week from now, Horizon is 2 weeks, the release type will be Firm. |
| forecastHorizon | Duration | The longest horizon, this starts at the end of the planned horizon. |
| plannedHorizon | Duration | Planned Horizon - is the period of time, from the END of the Firm Horizon, where all newly created releases - with a dueAt inside the horizon will be set to Plan. Example the Firm horizon is 2 weeks, the plan horizon is 3 weeks. A release with a DueAt of 4 weeks from today is entered, it will be a plan release type - unless you manually override it in fuuz or via the external system. |
| responsibleUserId | ID | — |
| responsibleUser | User | — |
| businessPartnerAddresses | [BusinessPartnerAddress!]! | — |
| orders | [Order!]! | — |
| destinationOrders | [Order!]! | — |

BusinessPartnerAddress

An addressed location for a business partner. May contain additional notes such as for packaging and shipping specifications. Acts as an endpoint for shipping and receiving.

Kind: Reference  |  Module: orderProcessing  |  Model ID: businessPartnerAddress  |  Fields: 18

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| active | Boolean! | — |
| externalId | String | — |
| internalNote | String | — |
| packagingNote | String | — |
| partnerLocationId | String | — |
| shippingNote | String | — |
| textAddress | String! | Unstructured address. |
| customData | JSON | — |
| integrationData | JSON | — |
| address | Address | Structured address object |
| businessPartnerId | ID! | — |
| businessPartner | BusinessPartner! | — |
| orders | [Order!]! | — |
| orderLines | [OrderLine!]! | — |
| destinationOrders | [Order!]! | — |
| destinationOrderLines | [OrderLine!]! | — |

Load

Load is a single Shipment, or a collection of shipments. A load is already required for shipments.

Kind: Reference  |  Module: orderProcessing  |  Model ID: load  |  Fields: 29

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | — |
| active | Boolean! | — |
| appointmentNumber | String | — |
| expediteCode | String | — |
| externalId | String | — |
| note | String | — |
| sealNumber | String | — |
| trailerNumber | String | An identifier for the physical trailer, typically given by the carrier (UPS, FedEx, etc.) in addition to other tracking-related information. |
| appointmentAt | DateTime | — |
| arrivalAt | DateTime | — |
| bolPrintedAt | DateTime | Date / Time when the Bill of Lading document was printed, this can be reset |
| departAt | DateTime | — |
| packPrintedAt | DateTime | Date / Time when the packing document was printed. This can be reset. |
| pickPrintedAt | DateTime | Date / Time when the picking document was printed. This can be reset. |
| scheduledDepartAt | DateTime | — |
| chargeData | JSON | — |
| customData | JSON | — |
| integrationData | JSON | — |
| dockStorageUnitId | ID | — |
| dockStorageUnit | StorageUnit | — |
| loadStatusId | ID | — |
| loadStatus | LoadStatus | — |
| pick | Pick | — |
| scacId | ID | — |
| scac | StandardCarrierAlphaCode | — |
| shipments | [Shipment!]! | — |
| orders | [Order!]! | — |
| handlingUnits | [HandlingUnit!]! | — |

Order

Data change set for 90 days by default, changes to the orders should only be initiated from the external ERP system or via other mechnisms and typically not done manually via Fuuz - however, if you do make manual changes, those changes will be tracked for 90 days unless this setting is modified.  Deleting an Order, will delete the line and release - providing no other transactions have been performed.

Kind: Reference  |  Module: orderProcessing  |  Model ID: order  |  Fields: 24

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | — |
| active | Boolean! | — |
| extStatus | String | String status mapped from an external system. |
| externalId | String | External Order Key or ID |
| customData | JSON | — |
| integrationData | JSON | — |
| orderStatusId | ID! | — |
| orderStatus | OrderStatus | — |
| orderTypeId | ID! | — |
| orderType | OrderType | — |
| businessPartnerId | ID | — |
| businessPartner | BusinessPartner | — |
| businessPartnerAddressId | ID | — |
| businessPartnerAddress | BusinessPartnerAddress | — |
| destinationBusinessPartnerId | ID | — |
| destinationBusinessPartner | BusinessPartner | — |
| destinationBusinessPartnerAddressId | ID | — |
| destinationBusinessPartnerAddress | BusinessPartnerAddress | — |
| orderLines | [OrderLine!]! | — |
| receipts | [Receipt!]! | — |
| loadId | ID | — |
| load | Load | — |
| specialInstructions | String | — |

OrderLine

Reference model in the orderProcessing module (module group orderFulfillment).

Kind: Reference  |  Module: orderProcessing  |  Model ID: orderLine  |  Fields: 17

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | — |
| unitPrice | Float! | — |
| active | Boolean! | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| orderId | ID! | — |
| order | Order! | — |
| productId | ID! | — |
| product | Product! | Direct link to the product for which this Order Line applies. Note, that if creating an order type of sale - you may want to use this which specifically indicates the productId that must be shipped; alternatively, the Catalog Item can provide a method to suggest any 1 of {} the list of CatalogItemProducts maybe used to fulfil this order line. It is not recommended to use both fields - if you do, the shipping logic will not consider the Catalog Item, it will only consider the productId as validation. |
| businessPartnerAddressId | ID | — |
| businessPartnerAddress | BusinessPartnerAddress | This address is specifically for the source location - If purchasing, or creating a transfer order - this address is the origin. |
| destinationBusinessPartnerAddressId | ID | — |
| destinationBusinessPartnerAddress | BusinessPartnerAddress | This address is the deliver to address- for a sale, purchase or transfer order. |
| orderLineReleases | [OrderLineRelease!]! | — |
| receiptLines | [ReceiptLine!]! | — |

OrderLineRelease

Order Line Releases are the individual ship/firm or plan schedules related to a single order number.

Kind: Reference  |  Module: orderProcessing  |  Model ID: orderLineRelease  |  Fields: 23

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | — |
| quantity | Measure! | Represents a quantity of a single product for a single order |
| quantityLeft | Measure! | The quantity that remains to be fulfilled. When this value is zero, that means the release has been fulfilled completely. This value is stored separately from the quantity because this value is used to track partial fulfillment over time, whereas the quantity represents the total amount released upon creation. |
| active | Boolean! | — |
| allowPartial | Boolean! | — |
| allowOverage | Boolean! | — |
| orderLineId | ID! | — |
| orderLine | OrderLine! | — |
| orderLineReleaseTypeId | ID! | — |
| orderLineReleaseType | OrderLineReleaseType! | — |
| orderLineReleaseStatusId | ID! | — |
| orderLineReleaseStatus | OrderLineReleaseStatus! | — |
| dueAt | Date! | — |
| shipAt | Date! | Ship At date is generally determined by the Due At minus any transit time, this Ship At data is helpful in planning for shipping/picking activities. |
| actualFulfilmentAt | Date | The actual date, of the shipment or receipt based on order type. Should be compared to the shipAt for sales orders, or dueAt for purchase orders. |
| associatedBarcode | String | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| receiptLineOrderLineReleases | [ReceiptLineOrderLineRelease!]! | — |
| shipmentLines | [ShipmentLine!]! | — |
| pickSteps | [PickStep!]! | — |

OrderLineReleaseStatus

Any Order Line Release Status marked as system will be included in base packages and should not be modified or removed.

Kind: Reference  |  Module: orderProcessing  |  Model ID: orderLineReleaseStatus  |  Fields: 8

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| description | String | — |
| system | Boolean! | — |
| usable | Boolean! | — |
| customData | JSON | — |
| integrationData | JSON | — |
| orderLineReleases | [OrderLineRelease!]! | — |

OrderStatus

Any Order Status marked as system - will be included in base packages and should not be modified or removed.

Kind: Reference  |  Module: orderProcessing  |  Model ID: orderStatus  |  Fields: 11

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String! | — |
| orderLineReleaseUse | Boolean! | — |
| orderUse | Boolean! | — |
| system | Boolean! | — |
| usable | Boolean! | — |
| description | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| orders | [Order!]! | — |
| orderLineReleases | [OrderLineRelease!]! | — |

OrderType

Reference model in the orderProcessing module (module group orderFulfillment).

Kind: Reference  |  Module: orderProcessing  |  Model ID: orderType  |  Fields: 4

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| name | String! | — |
| usable | Boolean! | — |
| orders | [Order!]! | — |

Pick

Reference model in the orderProcessing module (module group orderFulfillment).

Kind: Reference  |  Module: orderProcessing  |  Model ID: pick  |  Fields: 14

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| actualStartAt | String | — |
| completedAt | String | — |
| number | String | — |
| scheduledStartAt | String | — |
| dropoffStorageUnitId | ID | — |
| dropoffStorageUnit | StorageUnit | — |
| loadId | ID | — |
| load | Load | — |
| pickStatusId | ID | — |
| pickStatus | PickStatus | — |
| userId | ID | — |
| user | User | — |
| pickSteps | [PickStep!]! | — |

Shipment

Shipment represents a single End Point - Customer - Supplier to which the goods contained within the shipment lines will be delivered to.  A single shipment record links to 1 Order.

Kind: Reference  |  Module: orderProcessing  |  Model ID: shipment  |  Fields: 24

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| number | String! | — |
| sequence | Int! | The load sequence - meaning, if the load must be handled in FIFO basis, the shipments for the load must be organized that way as well. using this sequence field. This field is required and defaults to 1000, lower numbers are presumed to be loaded ahead of larger numbers. This correlates to the a stop sequence, but in reverse order. |
| active | Boolean! | If false, this shipment has either not been initialized yet, or has been cancelled and is now here for record keeping only. |
| asnNumber | String | **ASN (Advance Shipment Notice) Number / Identifier** The ASN number must be **unique across all loads in the system**. In many cases this number is the same as the internal load number; however, when external systems are used to manually create ASNs, those externally assigned identifiers can also be carried into Fuuz as the ASN number. - **In X12 856 EDI (Ship Notice/Manifest):**The **BSN02 (Shipment Identification)** field is the document instance’s unique identifier, and is typically what trading partners refer to as the ASN number. - **In EDIFACT DESADV (Dispatch Advice):**The equivalent identifier is found in the **BGM segment, data element 1004 (Document/Message Number)**, which represents the sender-assigned unique ASN number. This ASN number provides the shipment-level control reference that links physical goods, packaging identifiers, and transactional acknowledgements across systems. |
| trackingNumber | String | — |
| externalId | String | — |
| note | String | — |
| bolPrintedAt | DateTime | Date / Time when the Bill of Lading document was printed, this can be reset |
| chargeData | JSON | — |
| customData | JSON | — |
| integrationData | JSON | — |
| businessPartnerAddressId | ID! | — |
| businessPartnerAddress | BusinessPartnerAddress! | — |
| shipmentTypeId | ID! | — |
| shipmentType | ShipmentType! | Describes the mode of transportation a given shipment will take |
| shipmentLines | [ShipmentLine!]! | — |
| proNumber | String | — |
| freightTerms | String | — |
| bolNumber | String | — |
| shipAt | DateTime | — |
| loadId | ID | — |
| load | Load | — |
| commercialInvoiceNumber | String | — |

ShipmentLine

Represents a quantity of a single product for a single order during it's journey from load creation to shipment completion.

Kind: Reference  |  Module: orderProcessing  |  Model ID: shipmentLine  |  Fields: 13

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | — |
| number | String! | The identifier of this line item in the corresponding shipment. It is gauranteed to be unique per shipment, but not globally unique. |
| quantity | Measure! | The quantity of inventory that has physically been loaded onto the shipment. This value will start at 0 when the shipment details are created, then it will add up until the amount reaches the sum of the quantities of the corresponding order line releases. |
| shipmentId | ID! | — |
| shipment | Shipment! | — |
| externalId | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| inventories | [Inventory!]! | — |
| orderLineReleaseId | ID | — |
| orderLineRelease | OrderLineRelease | — |
| packedQuantity | Measure | — |

ShipmentLineOrderLineRelease

Defines how much and to which shipment line(s) a given order line release is allocated.

Kind: Reference  |  Module: orderProcessing  |  Model ID: shipmentLineOrderLineRelease  |  Fields: 7

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| quantityAllocated | Measure! | The quantity from the order line release that was allocated to the corresponding shipment line If a shipment is short shipped this will be updated to the amount that was actually shipped. the difference between the OriginalAllocatedQuantity and the allocatedQuantity is how much this line is short by. This is because the quantityAlocated is how the system calculates the quantityLeft for sales orders. |
| shipmentLineId | ID! | — |
| shipmentLine | ShipmentLine! | — |
| orderLineReleaseId | ID! | — |
| orderLineRelease | OrderLineRelease! | — |
| quantityOriginallyAllocated | Measure | — |

ShipmentType

A system model that describes what kind of transaction this shipment is, whether a sale to a customer, a transfer to another facility, or a return to supplier.

Kind: Reference  |  Module: orderProcessing  |  Model ID: shipmentType  |  Fields: 4

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| name | String! | The human-readable label for this shipment type |
| description | String! | A longer explanation of what this shipment type is and when it's used |
| shipments | [Shipment!]! | — |

### 4.7 Order Fulfillment & Business Partners

LoadStatus

Reference model in the logistics module (module group materialsManagement).

Kind: Reference  |  Module: logistics  |  Model ID: loadStatus  |  Fields: 6

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean | — |
| usable | Boolean | — |
| description | String | — |
| name | String | — |
| loads | [Load!]! | — |

OrderLineReleaseType

Reference model in the logistics module (module group materialsManagement).

Kind: Reference  |  Module: logistics  |  Model ID: orderLineReleaseType  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| name | String! | — |
| usable | Boolean! | — |
| description | String | — |
| orderLineReleases | [OrderLineRelease!]! | — |

### 4.8 Site Management

ProductPreferredStorageUnit

Used to determine which storage units each product is compatible with, and with what priority, for both filtering and auto-assigning features

Kind: Reference  |  Module: siteManagement  |  Model ID: productPreferredStorageUnit  |  Fields: 6

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| priority | Int! | The priority of preference for a storage unit for some particular product. Each product may have any number of preferred storage units to pick from in ascending priority order. |
| productId | ID! | — |
| product | Product! | — |
| storageUnitId | ID! | — |
| storageUnit | StorageUnit! | — |

StorageUnit

Reference model in the siteManagement module (module group siteManagement).

Kind: Reference  |  Module: siteManagement  |  Model ID: storageUnit  |  Fields: 37

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| active | Boolean! | — |
| crossDock | Boolean! | — |
| packing | Boolean! | — |
| picking | Boolean! | — |
| qualityControl | Boolean! | — |
| receiving | Boolean! | — |
| returns | Boolean! | — |
| shipping | Boolean! | — |
| storage | Boolean! | — |
| altitude | String | — |
| code | String! | — |
| description | String | — |
| externalId | String | — |
| latitude | String | — |
| longitude | String | — |
| name | String | — |
| customData | JSON | — |
| integrationData | JSON | — |
| height | Measure | Height of the bin / location space. |
| length | Measure | length of the bin/location space. |
| width | Measure | width of the bin/location space. |
| defaultInventoryStatusId | ID | — |
| defaultInventoryStatus | InventoryStatus | — |
| storageUnitStatusId | ID | — |
| storageUnitStatus | StorageUnitStatus | — |
| storageZoneId | ID | — |
| storageZone | StorageZone | — |
| products | [Product!]! | — |
| receiptLines | [ReceiptLine!]! | — |
| putAwayReceiptLines | [ReceiptLine!]! | — |
| dockReceipts | [Receipt!]! | — |
| inventories | [Inventory!]! | — |
| expectedCountLines | [CountLine!]! | — |
| countParameters | [CountParameters!]! | — |
| countLineInventories | [CountLineInventory!]! | — |
| picks | [Pick!]! | — |

StorageUnitStatus

Reference model in the siteManagement module (module group siteManagement).

Kind: Reference  |  Module: siteManagement  |  Model ID: storageUnitStatus  |  Fields: 5

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| code | String | — |
| active | Boolean | — |
| usable | Boolean | — |
| description | String | — |

StorageZone

Reference model in the siteManagement module (module group siteManagement).

Kind: Reference  |  Module: siteManagement  |  Model ID: storageZone  |  Fields: 19

| Field | Type | Description |
|---|---|---|
| id | ID! | The primary unique ID of the record. Every model should have an id field. |
| crossDocking | Boolean! | - **Description**: Areas where incoming products are immediately prepared for outbound shipping - **Characteristics**:Minimal or no storage time Located near both receiving and shipping docks Focused on speed of throughput - **Best for**: Pre-allocated orders, promotional items, seasonal products - **Layout considerations**: Staging areas with clear demarcation for order consolidation |
| fastMoving | Boolean! | - **Description**: Areas dedicated to high-velocity products that have frequent picks - **Characteristics**:Located near packing stations to minimize travel distance Optimized for easy access and quick retrieval Often feature forward pick locations - **Best for**: Top 20% of SKUs that generate 80% of picking activity - **Layout considerations**: Usually at waist or chest height to minimize bending/reaching |
| hazardous | Boolean! | - **Description**: Segregated areas for dangerous or regulated substances - **Characteristics**:Special safety equipment and protocols Often have restricted access May need specialized ventilation or containment - **Best for**: Chemicals, flammables, corrosives, regulated substances - **Layout considerations**: Generally separated from main picking areas, with special handling requirements |
| highValue | Boolean! | - **Description**: Secured areas for expensive or theft-prone items - **Characteristics**:Limited access controls (key cards, biometrics) Additional surveillance May require two-person verification for picking - **Best for**: Electronics, luxury goods, controlled substances, high-value components - **Layout considerations**: Caged areas, locked cabinets, separate security protocols |
| kitting | Boolean! | - **Description**: Areas where multiple items are combined into kits or assembled before shipping - **Characteristics**:Work tables or assembly stations Multiple item pick access Often involves specialized packaging - **Best for**: Product bundles, gift sets, items requiring assembly - **Layout considerations**: Need workspace and access to component parts |
| overflow | Boolean! | - **Description**: Flexible spaces that can be repurposed based on seasonal needs - **Characteristics**:Less permanent storage solutions May be used differently throughout the year Can serve as buffer capacity - **Best for**: Holiday merchandise, seasonal products, promotional items - **Layout considerations**: Modular storage that can be reconfigured |
| oversized | Boolean! | - **Description**: Areas designed for large or heavy products - **Characteristics**:Wider aisles to accommodate larger handling equipment Specialized storage like floor stacking or cantilever racking Often requires mechanical assistance for picking - **Best for**: Furniture, large appliances, long materials, palletized items - **Layout considerations**: Floor-level accessibility, reinforced shelving or floor storage |
| refrigerated | Boolean! | - **Description**: Temperature-controlled areas for perishable items - **Characteristics**:Maintain specific temperature ranges May have different picking equipment requirements Often require expedited picking processes to maintain cold chain - **Best for**: Food products, pharmaceuticals, chemicals requiring temperature control - **Layout considerations**: Need to minimize door openings and time spent retrieving products |
| returnProcessing | Boolean! | - **Description**: Dedicated areas for processing returned merchandise - **Characteristics**:Inspection stations Sorting capabilities Repackaging equipment - **Best for**: Customer returns, damaged goods processing - **Layout considerations**: Often near receiving but separate from regular inventory |
| slowMoving | Boolean! | - **Description**: Areas for products with lower picking frequency - **Characteristics**:Can be located farther from main picking aisles May use more dense storage methods Often located in upper or lower storage positions - **Best for**: Items that are ordered less frequently but still need to be accessible - **Layout considerations**: Can utilize harder-to-access locations since visits are infrequent |
| usable | Boolean! | — |
| code | String! | — |
| description | String | — |
| externalId | String | — |
| areaId | ID | — |
| area | Area | — |
| storageUnits | [StorageUnit!]! | — |
| countParameters | [CountParameters!]! | — |

### 5. Core Components — Data Flows

The package ships 49 data flows, grouped below by functional area. Each flow section lists its type, module, and node count, a purpose description (drawn from the flow’s package description where authored), and the sequence of nodes that make up the flow.

### 5.1 Receiving

Confirm Line

Copied from Confirm Line Web v0.0.1

Flow Type: System  |  Module: orderReceiving  |  Nodes: 8  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Validate | validate | Schema/business-rule validation. |
| Set Context | setContext | Sets/replaces the flow context. |
| Query: Get PO | query | GraphQL query — reads records. |
| Mutate: Update Release Order Lines | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Mutate: Update Order Status | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |

Confirm Line Web

A Screen-type flow composed of: 1 request, 2 validate, 1 setContext, 1 executeFlow, 1 response, 1 searchTable.

Flow Type: Screen  |  Module: orderReceiving  |  Nodes: 7  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Set Context | setContext | Sets/replaces the flow context. |
| Execute Flow | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |
| Refresh Orders Table | searchTable | Executes/refreshes a table search. |
| Validate | validate | Schema/business-rule validation. |

Confirm Receipt Line

A System-type flow composed of: 1 request, 3 mergeContext, 1 query, 1 validate, 1 mutexLock, 3 mutexUnlock, 1 transform, 1 ifElse, 2 mutate, 3 executeFlow, 3 response, 1 tryCatch.

Flow Type: System  |  Module: orderReceiving  |  Nodes: 21  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Receipt Line | query | GraphQL query — reads records. |
| Validate | validate | Schema/business-rule validation. |
| Mutex Lock | mutexLock | Acquires a distributed lock to prevent concurrent execution. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Prepare Payloads | transform | JSONata data transform. |
| If Else | ifElse | Conditional branch (if/else). |
| Merge Context | mergeContext | Merges values into the flow context. |
| Mutate Create Handling Units | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Execute Flow Create Invenotry | executeFlow | Invokes another data flow. |
| Mutate Receipt Lines | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Response | response | Returns the result payload to the caller. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Try Catch | tryCatch | Error-handling boundary. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Execute Flow Update Releases | executeFlow | Invokes another data flow. |
| Execute Flow Update Receipts | executeFlow | Invokes another data flow. |

Create Receipt Line

A System-type flow composed of: 1 request, 1 query, 1 mergeContext, 1 switch, 5 response, 1 mutate, 1 mutexLock, 2 mutexUnlock, 1 tryCatch, 1 executeFlow.

Flow Type: System  |  Module: orderReceiving  |  Nodes: 15  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Query Release Info | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Route | switch | Multi-branch conditional router. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Mutate Create Receipt Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Mutex Lock | mutexLock | Acquires a distributed lock to prevent concurrent execution. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Try Catch | tryCatch | Error-handling boundary. |
| Response | response | Returns the result payload to the caller. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Response | response | Returns the result payload to the caller. |
| Execute Flow | executeFlow | Invokes another data flow. |

Update Receipt Status

A System-type flow composed of: 1 request, 1 response, 1 mergeContext, 1 validate, 1 query, 1 transform, 1 mutate.

Flow Type: System  |  Module: orderReceiving  |  Nodes: 7  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Validate | validate | Schema/business-rule validation. |
| Query Receipt Data | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Mutate Receipt | mutate | GraphQL mutation — creates, updates, or deletes records. |

### 5.2 Inventory & Product Management

Create Inventory

A System-type flow composed of: 1 request, 1 publish, 2 transform, 1 validate, 1 mutate, 1 response, 1 mergeContext, 1 query.

Flow Type: System  |  Module: inventory  |  Nodes: 9  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Publish | publish | Publishes an event/message to a topic. |
| Validate Basic Payload Type | transform | JSONata data transform. |
| Validate Inventory Array Input | validate | Schema/business-rule validation. |
| Mutate - Create Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Product | query | GraphQL query — reads records. |
| Join Product | transform | JSONata data transform. |

Merge Inventory

Take some quantity from the source inventory and transfer it to the destination inventory. If the source inventory is completely consumed, you may pass "true" into the "mergeAll" input to delete (deactivate) the source inventory after the merge. You may either pass in "true" for "mergeAll" OR a quantity for "mergeQty", but not both.

Flow Type: System  |  Module: inventoryTracking  |  Nodes: 58  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Source & Destination Inventory | query | GraphQL query — reads records. |
| Validate | validate | Schema/business-rule validation. |
| Merge Context Input | mergeContext | Merges values into the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Route Validate Inventory | switch | Multi-branch conditional router. |
| Response Source Not Found | response | Returns the result payload to the caller. |
| Response Destination Not Found | response | Returns the result payload to the caller. |
| Response Same Source and Destination | response | Returns the result payload to the caller. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Merge Context Add Unit Info | mergeContext | Merges values into the flow context. |
| Response Quantity is Negative | response | Returns the result payload to the caller. |
| Response Quantity is Zero | response | Returns the result payload to the caller. |
| Route Validate Units | switch | Multi-branch conditional router. |
| Response Inventory Unit Mismatch | response | Returns the result payload to the caller. |
| Response Quantity Unit Mismatch | response | Returns the result payload to the caller. |
| | log | Writes a log entry. |
| Route Validate Merge All & Quantity | switch | Multi-branch conditional router. |
| Response Merge All & Qty | response | Returns the result payload to the caller. |
| Response Quantity Too Big | response | Returns the result payload to the caller. |
| Route Validate Inventory Merge | switch | Multi-branch conditional router. |
| Response Source Inventory Inactive | response | Returns the result payload to the caller. |
| Response Destination Inventory Inactive | response | Returns the result payload to the caller. |
| Merge Context Check Source Destination Mismatches | mergeContext | Merges values into the flow context. |
| Query Source & Destination Inventory Id | query | GraphQL query — reads records. |
| JSONata Translate to Request | transform | JSONata data transform. |
| Merge Context Set mergeQty | mergeContext | Merges values into the flow context. |
| Merge Context Set mergeQty | mergeContext | Merges values into the flow context. |
| Route Merge All or Quantity | switch | Multi-branch conditional router. |
| Query Gather Unit Information | query | GraphQL query — reads records. |
| Inject Type Conversion Info Here --> | log | Writes a log entry. |
| Response Inventory Statuses Incompatible | response | Returns the result payload to the caller. |
| Response Handling Unit Check | response | Returns the result payload to the caller. |
| Response Lot Check | response | Returns the result payload to the caller. |
| Response Products Don't Match | response | Returns the result payload to the caller. |
| Response Shippable Doesn't Match | response | Returns the result payload to the caller. |
| Response Tracking Numbers Don't Match | response | Returns the result payload to the caller. |
| Response Past Expiration Date | response | Returns the result payload to the caller. |
| Response Work Orders Don't Match | response | Returns the result payload to the caller. |
| Response Processes Don't Match | response | Returns the result payload to the caller. |
| Response Shipment Lines Don't Match | response | Returns the result payload to the caller. |
| Response Inventory Weight Unit Mismatch | response | Returns the result payload to the caller. |
| Mutate Merge All Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Route Merge All or Quantity | switch | Multi-branch conditional router. |
| Response Inventory Weight Unit Mismatch | response | Returns the result payload to the caller. |
| Response Inventory Weight is Zero | response | Returns the result payload to the caller. |
| Mutate Merge Some Inventory With Weight | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Mutate Merge Some Inventory No Weight | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response Merge Successful | response | Returns the result payload to the caller. |
| Merge Context Set New Inventory | mergeContext | Merges values into the flow context. |
| Query Gather Unit Information (Conversions) | query | GraphQL query — reads records. |
| Query Source & Destination Inventory Id | query | GraphQL query — reads records. |
| JSONata Translate to Request | transform | JSONata data transform. |
| Route Validate Unit Types | switch | Multi-branch conditional router. |
| Response Inventory Unit Type Mismatch | response | Returns the result payload to the caller. |
| Response Quantity Unit Type Mismatch | response | Returns the result payload to the caller. |
| Response Different Inventory Statuses | response | Returns the result payload to the caller. |
| Response Differnt Expiration Date | response | Returns the result payload to the caller. |
| Route Validate Inventory Merge | switch | Multi-branch conditional router. |

Merge Inventory Post Submit Web

Only resets the form and focuses the source inventory input

Flow Type: Screen  |  Module: inventoryTracking  |  Nodes: 6  |  Active: true

| Node | Type | Role |
|---|---|---|
| Set Source Inventory Input Null | setFieldValue | Sets a specific screen field’s value. |
| Set Destination Inventory Input Null | setFieldValue | Sets a specific screen field’s value. |
| Set Merge Qty Input Null | setFieldValue | Sets a specific screen field’s value. |
| Focus Source Inventory Input | focusBlurField | Sets focus/blur on a screen field. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |

Move Inventory

Use this flow to change the location of inventory by inventoryId.

Flow Type: System  |  Module: inventoryTracking  |  Nodes: 21  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate Input | validate | Schema/business-rule validation. |
| Query Inventory | query | GraphQL query — reads records. |
| Merge Current Inventory & Storage Unit | mergeContext | Merges values into the flow context. |
| Mutate Move Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response Success | response | Returns the result payload to the caller. |
| Route - Handling Unit Check | switch | Multi-branch conditional router. |
| Response Error Storage Unit Inactive | response | Returns the result payload to the caller. |
| Response Error Inventory Inactive | response | Returns the result payload to the caller. |
| Response Error Storage Unit Unavailable | response | Returns the result payload to the caller. |
| Response Error Inventory Retired | response | Returns the result payload to the caller. |
| Set Context | setContext | Sets/replaces the flow context. |
| SU Empty? | ifElse | Conditional branch (if/else). |
| Route | switch | Multi-branch conditional router. |
| Response | response | Returns the result payload to the caller. |
| Merge Inventory | executeFlow | Invokes another data flow. |
| Prepare Payload | transform | JSONata data transform. |
| Merge Error? | ifElse | Conditional branch (if/else). |
| Response Error | response | Returns the result payload to the caller. |
| Response Success | response | Returns the result payload to the caller. |
| Response Error Default Inventory Status | response | Returns the result payload to the caller. |

Move Inventory Web

Used by the Move Inventory screen when the "Submit" button is pressed.

Flow Type: Screen  |  Module: inventoryTracking  |  Nodes: 77  |  Active: true

| Node | Type | Role |
|---|---|---|
| Set scannedInventoryBarcode= null | setFieldValue | Sets a specific screen field’s value. |
| Focus Scan Inventory Input | focusBlurField | Sets focus/blur on a screen field. |
| Execute Flow Move Inventory Backend | executeFlow | Invokes another data flow. |
| Snackbar - Report Move Results | snackbar | Non-blocking success/error/info message to the user. |
| Set locationCode = null | setFieldValue | Sets a specific screen field’s value. |
| Response | response | Returns the result payload to the caller. |
| If Successful | ifElse | Conditional branch (if/else). |
| Query Inventory & HUs | query | GraphQL query — reads records. |
| Validate | validate | Schema/business-rule validation. |
| Merge Input | mergeContext | Merges values into the flow context. |
| Route Query Results | switch | Multi-branch conditional router. |
| Merge Inventory & HUs | mergeContext | Merges values into the flow context. |
| Response Bad Input | response | Returns the result payload to the caller. |
| Confirm Remove Inventory From HU? | confirm | Blocking confirmation dialog shown to the user. |
| Response - Cancelled | response | Returns the result payload to the caller. |
| Snackbar Inventory Move Cancelled | snackbar | Non-blocking success/error/info message to the user. |
| If Remember Location | ifElse | Conditional branch (if/else). |
| Set location = null | setFieldValue | Sets a specific screen field’s value. |
| Route Location Input Type | switch | Multi-branch conditional router. |
| Alert Multiple Found | alert | Blocking informational dialog. |
| Alert Inventory Inactive | alert | Blocking informational dialog. |
| Alert Inventory Not Found | alert | Blocking informational dialog. |
| Alert Handling Unit Inactive | alert | Blocking informational dialog. |
| Merge Inventory IDs | mergeContext | Merges values into the flow context. |
| Merge Remove From Handing Unit | mergeContext | Merges values into the flow context. |
| Confirm Move Entire HU Stack? | confirm | Blocking confirmation dialog shown to the user. |
| Snackbar HU Move Cancelled | snackbar | Non-blocking success/error/info message to the user. |
| Response Move Failed | response | Returns the result payload to the caller. |
| Set Context Move Results | setContext | Sets/replaces the flow context. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Query Preferred Locations | query | GraphQL query — reads records. |
| Merge Preferred Locations | mergeContext | Merges values into the flow context. |
| Route | switch | Multi-branch conditional router. |
| Validate | validate | Schema/business-rule validation. |
| JSONata | transform | JSONata data transform. |
| Merge Input | mergeContext | Merges values into the flow context. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Query Inventory & HUs | query | GraphQL query — reads records. |
| Hide/Show Container | hideShowContainer | Shows or hides a screen container. |
| Hide/Show Container | hideShowContainer | Shows or hides a screen container. |
| Response | response | Returns the result payload to the caller. |
| Focus/Blur Field | focusBlurField | Sets focus/blur on a screen field. |
| Route Query Results | switch | Multi-branch conditional router. |
| Response Bad Input | response | Returns the result payload to the caller. |
| Confirm Remove Inventory From HU? | confirm | Blocking confirmation dialog shown to the user. |
| Response - Cancelled | response | Returns the result payload to the caller. |
| Snackbar Inventory Move Cancelled | snackbar | Non-blocking success/error/info message to the user. |
| Alert Multiple Found | alert | Blocking informational dialog. |
| Alert Inventory Inactive | alert | Blocking informational dialog. |
| Alert Inventory Not Found | alert | Blocking informational dialog. |
| Alert Handling Unit Inactive | alert | Blocking informational dialog. |
| Query Preferred Locations | query | GraphQL query — reads records. |
| Merge Inventory & HUs | mergeContext | Merges values into the flow context. |
| Validate | validate | Schema/business-rule validation. |
| JSONata | transform | JSONata data transform. |
| Query Preferred Locations | query | GraphQL query — reads records. |
| Confirm Move | confirm | Blocking confirmation dialog shown to the user. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Respons | response | Returns the result payload to the caller. |
| Validate | validate | Schema/business-rule validation. |
| JSONata | transform | JSONata data transform. |
| Query Inventory & HUs | query | GraphQL query — reads records. |
| Execute Flow Move Inventory Backend | executeFlow | Invokes another data flow. |
| Snackbar - Report Move Results | snackbar | Non-blocking success/error/info message to the user. |
| Set Context Move Results | setContext | Sets/replaces the flow context. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Hide/Show Container | hideShowContainer | Shows or hides a screen container. |
| Hide/Show Container | hideShowContainer | Shows or hides a screen container. |
| Focus/Blur Field | focusBlurField | Sets focus/blur on a screen field. |
| Response | response | Returns the result payload to the caller. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Route | switch | Multi-branch conditional router. |
| Snackbar - Report Move Results | snackbar | Non-blocking success/error/info message to the user. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |

Split Inventory

Use this flow to split inventory, creating a new inventory record with quantity subtracted from the original and added to the new.

Flow Type: System  |  Module: inventoryTracking  |  Nodes: 15  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate Input | validate | Schema/business-rule validation. |
| Mutate - Create Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query All Inventory Details | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Route | switch | Multi-branch conditional router. |
| Response Inventory Not Found | response | Returns the result payload to the caller. |
| Response Split Qty Too Big | response | Returns the result payload to the caller. |
| Response Split Qty Too Small | response | Returns the result payload to the caller. |
| Response Weight Units Different | response | Returns the result payload to the caller. |
| Mutate - Create Trace | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response Success | response | Returns the result payload to the caller. |
| Set Context | setContext | Sets/replaces the flow context. |
| Response Inventory Status Not Permitted | response | Returns the result payload to the caller. |

Split Inventory Web

Use this flow to split inventory, creating a new inventory record with quantity subtracted from the original and added to the new.

Flow Type: Screen  |  Module: inventoryTracking  |  Nodes: 10  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Set invNumber = null | setFieldValue | Sets a specific screen field’s value. |
| Focus Scan Input | focusBlurField | Sets focus/blur on a screen field. |
| Execute Backend Flow | executeFlow | Invokes another data flow. |
| Snackbar - Report Split Results | snackbar | Non-blocking success/error/info message to the user. |
| Set splitQty = null | setFieldValue | Sets a specific screen field’s value. |
| Response Success | response | Returns the result payload to the caller. |
| Execute Backend Flow | executeFlow | Invokes another data flow. |
| If Successful | ifElse | Conditional branch (if/else). |
| Response Fail | response | Returns the result payload to the caller. |

### 5.3 Cycle Counting

Cancel Count

To update the status of the count to a cancelled status.  Used in: The count details screen on the cancel button

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 12  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Query Count | query | GraphQL query — reads records. |
| If Else | ifElse | Conditional branch (if/else). |
| Mutate | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Load Form Data | loadFormData | Loads data to pre-populate a form. |

Complete Count

To update the status of the count to a complete status and apply all of the changes counted to the inventory in the count.  Used in: The count details screen on the complete button

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 10  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Query Count Inventory | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Mutate | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |

Count Count Line Inventory Webflow

To validate the entered count info for a count line inventory.  Used in: The execute count screen.

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 21  |  Active: true

| Node | Type | Role |
|---|---|---|
| If Else Has Been Counted | ifElse | Conditional branch (if/else). |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Mutate Update Line Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Set Field Value | setFieldValue | Sets a specific screen field’s value. |
| Load Form Data | loadFormData | Loads data to pre-populate a form. |
| Focus/Blur Field | focusBlurField | Sets focus/blur on a screen field. |
| If Else Is in Correct Location | ifElse | Conditional branch (if/else). |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Mutate Update StorageUnit | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Publish | publish | Publishes an event/message to a topic. |
| JSONata | transform | JSONata data transform. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Query Count Line | query | GraphQL query — reads records. |
| Route | switch | Multi-branch conditional router. |
| Alert Dialog | alert | Blocking informational dialog. |
| Enable/Disable Field | enableDisableField | Enables or disables a screen field. |
| Enable/Disable Field | enableDisableField | Enables or disables a screen field. |

Create Count Line Inventory

To create count line inventory for a count that already has cout lines created.  Used in: This is called from another flow when the create count is initated.

Flow Type: System  |  Module: cycleCounting  |  Nodes: 11  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Count | query | GraphQL query — reads records. |
| If Else Count is Started | ifElse | Conditional branch (if/else). |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Inventory | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Mutate Create Count Line Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| If Else Pagination | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |
| Validate | validate | Schema/business-rule validation. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |

Cycle Count Web Flow

This is to create a new count. This calls flows to create the inital count and count lines and calls another fow to create the count line inventory.  Used in: The count tables create button

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 14  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Create Line Inventory | executeFlow | Invokes another data flow. |
| Start Count | executeFlow | Invokes another data flow. |
| If Else | ifElse | Conditional branch (if/else). |
| Alert Dialog | alert | Blocking informational dialog. |
| Validate | validate | Schema/business-rule validation. |
| If Else | ifElse | Conditional branch (if/else). |
| Alert Dialog | alert | Blocking informational dialog. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Search Table | searchTable | Executes/refreshes a table search. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |

Exclude Count Line Inventory

This is used to delete a count line inventory from a count. This will not change the expected amount for the line. But it will effect the progress % when scanning.  Used in: The count details screen.

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 3  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Mutate | mutate | GraphQL mutation — creates, updates, or deletes records. |

Execute Cycle Count Scan

This is to load a count line details based on a serial number that is scanned. This will prompt you if the inventory is not in part of a count.  Used in: The execute cycle count screen.

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 17  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Inventory | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Route | switch | Multi-branch conditional router. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Prepare Payload | transform | JSONata data transform. |
| Mutate Add Line Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Alert Dialog | alert | Blocking informational dialog. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Mutate Update StorageUnit | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Load Form Confirm Panel | loadFormData | Loads data to pre-populate a form. |
| Load Form Progress | loadFormData | Loads data to pre-populate a form. |
| Response | response | Returns the result payload to the caller. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Enable/Disable Field | enableDisableField | Enables or disables a screen field. |
| Enable/Disable Field | enableDisableField | Enables or disables a screen field. |

Recount Count Line Inventory

This will reset the counted user id on the count line inventory. This will change the progress % on count and require someone to recount the inventory  Used in: The count details screen.

Flow Type: Screen  |  Module: cycleCounting  |  Nodes: 3  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Mutate | mutate | GraphQL mutation — creates, updates, or deletes records. |

Start Count and Create Count Lines

This is used to create the inital count and count lines.  Used in: This is used in another flow that creates the count lines and inventory.

Flow Type: System  |  Module: cycleCounting  |  Nodes: 12  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Count | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Inventory Agg | query | GraphQL query — reads records. |
| Mutate Create Count Lines | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| If Else Pagination | ifElse | Conditional branch (if/else). |
| Mutate Start Count | mutate | GraphQL mutation — creates, updates, or deletes records. |
| If Else Count is New | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |
| Validate | validate | Schema/business-rule validation. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |

### 5.4 Picking

Create Pick Step

A System-type flow composed of: 1 request, 1 validate, 1 mutexLock, 1 mergeContext, 2 query, 2 mutexUnlock, 1 switch, 2 tryCatch, 6 response, 6 transform, 1 mutate, 3 log, 2 ifElse.

Flow Type: System  |  Module: picking  |  Nodes: 29  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Mutex Lock | mutexLock | Acquires a distributed lock to prevent concurrent execution. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Inventory | query | GraphQL query — reads records. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Route | switch | Multi-branch conditional router. |
| Try Catch | tryCatch | Error-handling boundary. |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| Mutate Create Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Prepare Payload | transform | JSONata data transform. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Error Log | log | Writes a log entry. |
| Response - Error | response | Returns the result payload to the caller. |
| Initial Log | log | Writes a log entry. |
| Enough Inventory? | ifElse | Conditional branch (if/else). |
| Response - Error | response | Returns the result payload to the caller. |
| Response - Error | response | Returns the result payload to the caller. |
| Some Inventory? | ifElse | Conditional branch (if/else). |
| Return Available | transform | JSONata data transform. |
| Try Catch | tryCatch | Error-handling boundary. |
| Response | response | Returns the result payload to the caller. |
| Error Log | log | Writes a log entry. |
| Query Inventory | query | GraphQL query — reads records. |
| Return Available | transform | JSONata data transform. |
| Prepare Payload | transform | JSONata data transform. |
| Prepare Payload | transform | JSONata data transform. |

Create Pick Web Flow

A Screen-type flow composed of: 2 query, 10 mergeContext, 1 setContext, 3 switch, 5 formDialog, 8 response, 4 executeFlow, 4 tryCatch, 1 request, 2 confirm, 1 alert, 2 mutate, 5 snackbar, 2 ifElse, 1 transform.

Flow Type: Screen  |  Module: picking  |  Nodes: 51  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Order(s) | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Set Context | setContext | Sets/replaces the flow context. |
| Business Partner | switch | Multi-branch conditional router. |
| Merge Partner | mergeContext | Merges values into the flow context. |
| No Business Partner | formDialog | Dialog collecting additional user input. |
| Multiple Partners | formDialog | Dialog collecting additional user input. |
| Merge Partner | mergeContext | Merges values into the flow context. |
| Merge Partner | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Initial Log | executeFlow | Invokes another data flow. |
| Try Catch | tryCatch | Error-handling boundary. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Error Log | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |
| Partner Address | switch | Multi-branch conditional router. |
| Merge Address | mergeContext | Merges values into the flow context. |
| Multiple Address | formDialog | Dialog collecting additional user input. |
| No Partner Address | formDialog | Dialog collecting additional user input. |
| Merge Address | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Merge Address | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Create Load and Shipment | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |
| Route | switch | Multi-branch conditional router. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Response | response | Returns the result payload to the caller. |
| Alert Dialog | alert | Blocking informational dialog. |
| Create Pick | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Try Catch | tryCatch | Error-handling boundary. |
| Error Log | executeFlow | Invokes another data flow. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Continue? | ifElse | Conditional branch (if/else). |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Try Catch | tryCatch | Error-handling boundary. |
| Continue? | ifElse | Conditional branch (if/else). |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Create Pick Step | transform | JSONata data transform. |
| Try Catch | tryCatch | Error-handling boundary. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Query Pick Steps | query | GraphQL query — reads records. |
| Release Pick Steps | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |

Execute Pick Web Flow

A Screen-type flow composed of: 1 request, 8 switch, 14 response, 22 transform, 1 setFieldValue, 8 query, 17 snackbar, 12 mergeContext, 8 ifElse, 2 setContext, 1 mutexLock, 1 tryCatch, 4 mutexUnlock, 7 confirm, 9 mutate, 6 echo, 2 alert, 2 delay, 1 formDialog, 6 executeFlow.

Flow Type: Screen  |  Module: picking  |  Nodes: 132  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Route | switch | Multi-branch conditional router. |
| Response | response | Returns the result payload to the caller. |
| Load Pick to state | transform | JSONata data transform. |
| Refresh Screen State | transform | JSONata data transform. |
| Add Inventory to state | transform | JSONata data transform. |
| Clear Scanner Input | setFieldValue | Sets a specific screen field’s value. |
| Query Storage Unit | query | GraphQL query — reads records. |
| Snackbar: Invalid Storage Unit | snackbar | Non-blocking success/error/info message to the user. |
| Query Inventory | query | GraphQL query — reads records. |
| Response (failure) | response | Returns the result payload to the caller. |
| Snackbar: Serial not found | snackbar | Non-blocking success/error/info message to the user. |
| Merge: Scanned Inventory | mergeContext | Merges values into the flow context. |
| Step Complete? | ifElse | Conditional branch (if/else). |
| Snackbar: Already Scanned | snackbar | Non-blocking success/error/info message to the user. |
| Response (failure) | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Snackbar: Incorrect Product | snackbar | Non-blocking success/error/info message to the user. |
| Set Context | setContext | Sets/replaces the flow context. |
| Mutex Lock | mutexLock | Acquires a distributed lock to prevent concurrent execution. |
| Try Catch ( all errors) | tryCatch | Error-handling boundary. |
| Snackbar Last Error | snackbar | Non-blocking success/error/info message to the user. |
| Snackbar: Success | snackbar | Non-blocking success/error/info message to the user. |
| Reset Screen State | transform | JSONata data transform. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Response (success) | response | Returns the result payload to the caller. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Set Form Mode | transform | JSONata data transform. |
| Complete Step? | confirm | Blocking confirmation dialog shown to the user. |
| Response (failure) | response | Returns the result payload to the caller. |
| Pick Complete? | ifElse | Conditional branch (if/else). |
| Mutate Pick | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Snackbar: Pick Complete | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Drop Current Stop and Reset Mode | transform | JSONata data transform. |
| Snackbar: Step Complete | snackbar | Non-blocking success/error/info message to the user. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Q: User by Id | query | GraphQL query — reads records. |
| Q:PickSteps | query | GraphQL query — reads records. |
| Mutate Pick Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge: Scan Type | mergeContext | Merges values into the flow context. |
| Mutate Pick Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Scan Pick | echo | Debug echo of the current payload/context. |
| Route | switch | Multi-branch conditional router. |
| Query Drop Zone Storage Unit | query | GraphQL query — reads records. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Response | response | Returns the result payload to the caller. |
| Initial Validation | switch | Multi-branch conditional router. |
| Snackbar: No Pick Steps | snackbar | Non-blocking success/error/info message to the user. |
| Snackbar: No Inventory | snackbar | Non-blocking success/error/info message to the user. |
| Error Mountain | echo | Debug echo of the current payload/context. |
| Location Scan | echo | Debug echo of the current payload/context. |
| Route | switch | Multi-branch conditional router. |
| Validate Inventory | switch | Multi-branch conditional router. |
| Snackbar: Bad Status | snackbar | Non-blocking success/error/info message to the user. |
| Dropoff | echo | Debug echo of the current payload/context. |
| Complete Step | echo | Debug echo of the current payload/context. |
| Mode Change Logic | switch | Multi-branch conditional router. |
| Set Dropoff Mode | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| Set Dropoff Mode | transform | JSONata data transform. |
| Drop Current Stop | transform | JSONata data transform. |
| Merge Context - Quantities | mergeContext | Merges values into the flow context. |
| Confirm Use Incorrect Serial | confirm | Blocking confirmation dialog shown to the user. |
| Q:PickSteps | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Drop Current Stop | transform | JSONata data transform. |
| Set Dropoff Mode | transform | JSONata data transform. |
| Q:InventoryPickStep | query | GraphQL query — reads records. |
| Snackbar: No Pick | snackbar | Non-blocking success/error/info message to the user. |
| Confirm Incorrect Storage Unit | confirm | Blocking confirmation dialog shown to the user. |
| Q:PickSteps | query | GraphQL query — reads records. |
| Mutate Actual Start | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Snackbar: No Pick Steps | snackbar | Non-blocking success/error/info message to the user. |
| M: Create NoncConformanceReport | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Snackbar: Wrong Location | alert | Blocking informational dialog. |
| Merge Context - Quantities | mergeContext | Merges values into the flow context. |
| Mutate create pickstep | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Add Inventory Pick Step to Context | transform | JSONata data transform. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Delay | delay | Introduces a timed delay. |
| If Else Inventory | ifElse | Conditional branch (if/else). |
| Validate Handling Unit | switch | Multi-branch conditional router. |
| Snackbar: Serial not found | snackbar | Non-blocking success/error/info message to the user. |
| Snackbar: Already Scanned | snackbar | Non-blocking success/error/info message to the user. |
| Response (failure) | response | Returns the result payload to the caller. |
| Snackbar: Incorrect Product | snackbar | Non-blocking success/error/info message to the user. |
| Mutex Unlock | mutexUnlock | Releases a distributed lock. |
| Snackbar: Wrong Location | alert | Blocking informational dialog. |
| Merge Context - Quantities | mergeContext | Merges values into the flow context. |
| Prepare Payload | transform | JSONata data transform. |
| Prepare Payload | transform | JSONata data transform. |
| M: Create NoncConformanceReport | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context - Quantities | mergeContext | Merges values into the flow context. |
| Mutate create pickstep | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Add Inventory Pick Step to Context | transform | JSONata data transform. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Confirm Use Incorrect Serial | confirm | Blocking confirmation dialog shown to the user. |
| Delay | delay | Introduces a timed delay. |
| Product Scan | echo | Debug echo of the current payload/context. |
| Allow Overage? | ifElse | Conditional branch (if/else). |
| Select Quantity | formDialog | Dialog collecting additional user input. |
| Prepare Payload | transform | JSONata data transform. |
| Flow - Split Inventory | executeFlow | Invokes another data flow. |
| Prepare Payload | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| Update Context | transform | JSONata data transform. |
| Prepare Split Payload | transform | JSONata data transform. |
| Merge Context For Designer Testing | mergeContext | Merges values into the flow context. |
| Split? (Inv > Pick Qty) | ifElse | Conditional branch (if/else). |
| Mutate Update InventoryPickStep, PickStep, Pick | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Overage Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Route | switch | Multi-branch conditional router. |
| Under Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Response | response | Returns the result payload to the caller. |
| Set Context | setContext | Sets/replaces the flow context. |
| Prepare Payload | transform | JSONata data transform. |
| Split? (Inv > Pick Qty) | ifElse | Conditional branch (if/else). |
| Update Order Status | executeFlow | Invokes another data flow. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Update Order Status | executeFlow | Invokes another data flow. |
| Update Load Status | executeFlow | Invokes another data flow. |
| Update Load Status | executeFlow | Invokes another data flow. |
| Pick Complete? | ifElse | Conditional branch (if/else). |
| Prepare Payload | transform | JSONata data transform. |
| PickStepComplete? | ifElse | Conditional branch (if/else). |
| Execute Flow | executeFlow | Invokes another data flow. |
| Set Screen Context | transform | JSONata data transform. |

NonConformancePublichMessage

A System-type flow composed of: 1 dataChanges, 1 publish.

Flow Type: System  |  Module: picking  |  Nodes: 2  |  Active: true

| Node | Type | Role |
|---|---|---|
| Data Changes | dataChanges | Triggers on a data-change event. |
| Publish | publish | Publishes an event/message to a topic. |

Pick Details Web Flow

A Screen-type flow composed of: 3 formDialog, 1 setContext, 1 switch, 10 response, 7 query, 5 transform, 7 mutate, 1 executeFlow, 1 request, 3 mergeContext, 4 ifElse, 2 alert, 1 generateDocument, 2 confirm, 1 snackbar.

Flow Type: Screen  |  Module: picking  |  Nodes: 49  |  Active: true

| Node | Type | Role |
|---|---|---|
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Set Context | setContext | Sets/replaces the flow context. |
| Route | switch | Multi-branch conditional router. |
| Response | response | Returns the result payload to the caller. |
| Query Load | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Update Load | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Initial Log | executeFlow | Invokes another data flow. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Query Pick Details | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Prepare Payload | transform | JSONata data transform. |
| loadId Provided? | ifElse | Conditional branch (if/else). |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Query Shipment | query | GraphQL query — reads records. |
| Update Shipment | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Alert Dialog | alert | Blocking informational dialog. |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| Render Document | generateDocument | Generates a printable document (e.g., label, BOL). |
| Query Pick Details | query | GraphQL query — reads records. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Prepare Payload | transform | JSONata data transform. |
| Update Pick | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Query Pick Steps | query | GraphQL query — reads records. |
| Delete Inv Pick Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Delete Pick Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Update Pick Step | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Query Order | query | GraphQL query — reads records. |
| Create Pick Step | transform | JSONata data transform. |
| Alert Dialog | alert | Blocking informational dialog. |
| Has Error? | ifElse | Conditional branch (if/else). |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| At Least 1 Success? | ifElse | Conditional branch (if/else). |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Query Pick Steps | query | GraphQL query — reads records. |
| Release Pick Steps | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Missing Pick Steps? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |

### 5.5 Packing

Pack Inventory

A Screen-type flow composed of: 1 request, 3 executeFlow, 1 setContext, 4 ifElse, 4 mergeContext, 2 query, 15 transform, 1 switch, 3 response, 1 mutate, 2 tryCatch, 1 snackbar, 2 alert.

Flow Type: Screen  |  Module: packing  |  Nodes: 40  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Create Handling Unit | executeFlow | Invokes another data flow. |
| Set Context | setContext | Sets/replaces the flow context. |
| Pack HU Provided? | ifElse | Conditional branch (if/else). |
| Merge Handling Unit | mergeContext | Merges values into the flow context. |
| Query Serial | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Route | switch | Multi-branch conditional router. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Merge Ship Line | mergeContext | Merges values into the flow context. |
| Has Ship Line? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |
| Mutate Update Line + Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Prepare Payload | transform | JSONata data transform. |
| Update Load Status | executeFlow | Invokes another data flow. |
| Query Load | query | GraphQL query — reads records. |
| Has LoadId? | ifElse | Conditional branch (if/else). |
| Merge LoadId | mergeContext | Merges values into the flow context. |
| Try Catch | tryCatch | Error-handling boundary. |
| Try Catch | tryCatch | Error-handling boundary. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Initial Log | executeFlow | Invokes another data flow. |
| Serial Not Found | transform | JSONata data transform. |
| Pick Not Complete | transform | JSONata data transform. |
| Serial Already Packed | transform | JSONata data transform. |
| Inventory is Not Active | transform | JSONata data transform. |
| Serial Inventory Status | transform | JSONata data transform. |
| No Shipment Lines | transform | JSONata data transform. |
| Pack Quantity Exceeded | transform | JSONata data transform. |
| No Shipment Line Found | transform | JSONata data transform. |
| Error Packing Inventory | transform | JSONata data transform. |
| Error Updating Status | transform | JSONata data transform. |
| Alert Dialog | alert | Blocking informational dialog. |
| Alert Dialog | alert | Blocking informational dialog. |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| Serial Not Shippable | transform | JSONata data transform. |
| Serial Provided? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |
| Serial Not in Load | transform | JSONata data transform. |

Packing Web Flow

A Screen-type flow composed of: 4 executeFlow, 1 setContext, 4 ifElse, 2 switch, 8 hideShowContainer, 8 response, 1 request, 2 transform, 3 query, 5 alert, 1 confirm, 2 mergeContext, 2 executeAction, 1 mutate, 1 formDialog.

Flow Type: Screen  |  Module: packing  |  Nodes: 45  |  Active: true

| Node | Type | Role |
|---|---|---|
| Initial Log | executeFlow | Invokes another data flow. |
| Set Context | setContext | Sets/replaces the flow context. |
| Pack By Order? | ifElse | Conditional branch (if/else). |
| Route | switch | Multi-branch conditional router. |
| Show Order | hideShowContainer | Shows or hides a screen container. |
| Hide Load | hideShowContainer | Shows or hides a screen container. |
| Response | response | Returns the result payload to the caller. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Show Load | hideShowContainer | Shows or hides a screen container. |
| Hide Order | hideShowContainer | Shows or hides a screen container. |
| Show Table | hideShowContainer | Shows or hides a screen container. |
| Hide Pack | hideShowContainer | Shows or hides a screen container. |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| Query Data | query | GraphQL query — reads records. |
| Route | switch | Multi-branch conditional router. |
| Alert Dialog | alert | Blocking informational dialog. |
| Alert Dialog | alert | Blocking informational dialog. |
| Alert Dialog | alert | Blocking informational dialog. |
| Confirm Dialog | confirm | Blocking confirmation dialog shown to the user. |
| Find Missing Pick Steps | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| Order Status | executeFlow | Invokes another data flow. |
| Load Status | executeFlow | Invokes another data flow. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Order Back Button | executeAction | Executes a screen action programmatically. |
| By Order? | ifElse | Conditional branch (if/else). |
| Load Back Button | executeAction | Executes a screen action programmatically. |
| Update HU | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query HU Details | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Found? | ifElse | Conditional branch (if/else). |
| Alert Dialog | alert | Blocking informational dialog. |
| Query HUs | query | GraphQL query — reads records. |
| Found? | ifElse | Conditional branch (if/else). |
| Alert Dialog | alert | Blocking informational dialog. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Show Table | hideShowContainer | Shows or hides a screen container. |
| Hide Pack | hideShowContainer | Shows or hides a screen container. |
| Pack Inventory | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |

Print Box Content Label

A System-type flow composed of: 1 request, 1 setContext, 1 log, 2 response, 2 query, 1 transform, 2 mergeContext, 1 generateDocument, 1 ifElse.

Flow Type: System  |  Module: packing  |  Nodes: 12  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Set Context | setContext | Sets/replaces the flow context. |
| Initial Log | log | Writes a log entry. |
| Response | response | Returns the result payload to the caller. |
| Query Header Info | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Merge Header Info | mergeContext | Merges values into the flow context. |
| Query Item Info | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Render Document | generateDocument | Generates a printable document (e.g., label, BOL). |
| HandlingUnitId? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |

Unpack Inventory

A Screen-type flow composed of: 1 request, 1 mergeContext, 1 query, 1 transform, 1 mutate, 1 response, 1 snackbar, 1 searchTable.

Flow Type: Screen  |  Module: packing  |  Nodes: 8  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Serial and Line | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Mutate Inventory and Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Search Table | searchTable | Executes/refreshes a table search. |

### 5.6 Shipping & Carrier Management

Add Edit SCAC Web Flow

A Screen-type flow composed of: 1 setContext, 1 request, 2 query, 2 formDialog, 1 mergeContext, 3 response, 2 ifElse, 2 snackbar, 1 alert, 2 mutate.

Flow Type: Screen  |  Module: shipping  |  Nodes: 17  |  Active: true

| Node | Type | Role |
|---|---|---|
| Set Context | setContext | Sets/replaces the flow context. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Query SCAC | query | GraphQL query — reads records. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Add? | ifElse | Conditional branch (if/else). |
| Exists? | ifElse | Conditional branch (if/else). |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Alert Dialog | alert | Blocking informational dialog. |
| Mutate SCAC | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query SCAC | query | GraphQL query — reads records. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Response | response | Returns the result payload to the caller. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Mutate SCAC | mutate | GraphQL mutation — creates, updates, or deletes records. |

Create Handling Unit Web Flow

A Screen-type flow composed of: 1 mutate, 3 transform, 2 query, 1 switch, 1 setContext, 1 request, 2 response, 1 tryCatch, 2 executeFlow, 1 formDialog, 1 mergeContext, 1 ifElse.

Flow Type: Screen  |  Module: shipping  |  Nodes: 17  |  Active: true

| Node | Type | Role |
|---|---|---|
| Create Handling Unit | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Prepare Payload | transform | JSONata data transform. |
| Query Order | query | GraphQL query — reads records. |
| Route | switch | Multi-branch conditional router. |
| Set Context | setContext | Sets/replaces the flow context. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| Query Load | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Try Catch | tryCatch | Error-handling boundary. |
| Initial Log | executeFlow | Invokes another data flow. |
| ErrorLog | executeFlow | Invokes another data flow. |
| Get HU Type | formDialog | Dialog collecting additional user input. |
| Merge Context | mergeContext | Merges values into the flow context. |
| HU Type? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |

Create Load and Shipments

A System-type flow composed of: 1 setContext, 4 mergeContext, 1 query, 7 mutate, 1 validate, 2 tryCatch, 1 switch, 2 response, 1 request, 1 log.

Flow Type: System  |  Module: shipping  |  Nodes: 21  |  Active: true

| Node | Type | Role |
|---|---|---|
| Set Context | setContext | Sets/replaces the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Order(s) | query | GraphQL query — reads records. |
| Mutate Create Load | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Validate | validate | Schema/business-rule validation. |
| Create Shipment | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Create Shipment Lines | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Load | mergeContext | Merges values into the flow context. |
| Merge Shipment | mergeContext | Merges values into the flow context. |
| Update Order | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Try Catch | tryCatch | Error-handling boundary. |
| Try Catch | tryCatch | Error-handling boundary. |
| Merge Shipment Lines | mergeContext | Merges values into the flow context. |
| Route | switch | Multi-branch conditional router. |
| Delete Shipments | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Delete Load | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response Undo Recovery | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Delete Shipment Lines | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Initial Log | log | Writes a log entry. |

Get BoL Number

A System-type flow composed of: 1 query, 2 mutate, 1 setContext, 2 mergeContext, 1 ifElse, 2 response, 1 request, 2 log, 1 tryCatch.

Flow Type: System  |  Module: shipping  |  Nodes: 13  |  Active: true

| Node | Type | Role |
|---|---|---|
| Query Shipment | query | GraphQL query — reads records. |
| Create Sequence | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Set Context | setContext | Sets/replaces the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| BoL Exist? | ifElse | Conditional branch (if/else). |
| Response | response | Returns the result payload to the caller. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Initial Log | log | Writes a log entry. |
| Try Catch | tryCatch | Error-handling boundary. |
| Error Log | log | Writes a log entry. |
| Update Shipment | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |

Get Commercial Invoice Number

A System-type flow composed of: 1 request, 2 log, 1 setContext, 1 query, 2 mergeContext, 1 ifElse, 1 tryCatch, 2 mutate, 2 response.

Flow Type: System  |  Module: shipping  |  Nodes: 13  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Initial Log | log | Writes a log entry. |
| Set Context | setContext | Sets/replaces the flow context. |
| Query Shipment | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| CI Exist? | ifElse | Conditional branch (if/else). |
| Try Catch | tryCatch | Error-handling boundary. |
| Create Sequence | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Update Shipment | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Error Log | log | Writes a log entry. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |

Print Shipping Documents Web Flow

A Screen-type flow composed of: 9 transform, 5 ifElse, 3 mergeContext, 1 formDialog, 2 response, 5 generateDocument, 2 query, 1 request, 1 setContext, 1 snackbar, 2 executeFlow, 1 executeDeviceFunction V2.

Flow Type: Screen  |  Module: shipping  |  Nodes: 33  |  Active: true

| Node | Type | Role |
|---|---|---|
| Options Payload | transform | JSONata data transform. |
| Doc Provided? | ifElse | Conditional branch (if/else). |
| Merge Context | mergeContext | Merges values into the flow context. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Response | response | Returns the result payload to the caller. |
| Prepare Payload | transform | JSONata data transform. |
| BoL? | ifElse | Conditional branch (if/else). |
| Packing List? | ifElse | Conditional branch (if/else). |
| Render BoL | generateDocument | Generates a printable document (e.g., label, BOL). |
| BoL | transform | JSONata data transform. |
| Query Details | query | GraphQL query — reads records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Set Context | setContext | Sets/replaces the flow context. |
| Query Details | query | GraphQL query — reads records. |
| Response | response | Returns the result payload to the caller. |
| Certificate of Conformance | transform | JSONata data transform. |
| Render Cert Conf | generateDocument | Generates a printable document (e.g., label, BOL). |
| Packing List | transform | JSONata data transform. |
| Render Packing List | generateDocument | Generates a printable document (e.g., label, BOL). |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Get BoL Number | executeFlow | Invokes another data flow. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Packing List | transform | JSONata data transform. |
| Merge PDF Copies | transform | JSONata data transform. |
| Execute Device Function V2 | executeDeviceFunction V2 | Invokes a connected device function (e.g., scanner, printer). |
| Commercial Invoice? | ifElse | Conditional branch (if/else). |
| Commercial Invoice | transform | JSONata data transform. |
| Commercial Invoice | generateDocument | Generates a printable document (e.g., label, BOL). |
| Get Invoice Number | executeFlow | Invokes another data flow. |
| Shipping Label? | ifElse | Conditional branch (if/else). |
| Shipping Label | transform | JSONata data transform. |
| Shipping Label | generateDocument | Generates a printable document (e.g., label, BOL). |

Ship Management Web Flow

A Screen-type flow composed of: 1 switch, 1 setContext, 3 mergeContext, 1 request, 6 response, 2 executeFlow, 4 ifElse, 4 hideShowContainer, 4 query, 4 mutate, 2 alert.

Flow Type: Screen  |  Module: shipping  |  Nodes: 32  |  Active: true

| Node | Type | Role |
|---|---|---|
| Route | switch | Multi-branch conditional router. |
| Set Context | setContext | Sets/replaces the flow context. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Initial Log | executeFlow | Invokes another data flow. |
| Pack By Order? | ifElse | Conditional branch (if/else). |
| Show Order | hideShowContainer | Shows or hides a screen container. |
| Hide Order | hideShowContainer | Shows or hides a screen container. |
| Hide Load | hideShowContainer | Shows or hides a screen container. |
| Show Load | hideShowContainer | Shows or hides a screen container. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query Inv | query | GraphQL query — reads records. |
| Packed? | ifElse | Conditional branch (if/else). |
| Update Order Status | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Update Inventory | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Update Shipment Status | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query Shipment | query | GraphQL query — reads records. |
| All Inv Status ok? | ifElse | Conditional branch (if/else). |
| Has Load? | ifElse | Conditional branch (if/else). |
| Order Status | query | GraphQL query — reads records. |
| Query Load | query | GraphQL query — reads records. |
| Alert Dialog | alert | Blocking informational dialog. |
| Alert Dialog | alert | Blocking informational dialog. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Update Load Status | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Print Ship Docs | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |

Update Load Status

A System-type flow composed of: 1 request, 1 validate, 1 query, 6 transform, 1 switch, 1 mutate, 1 mergeContext, 3 response, 1 ifElse, 1 tryCatch, 2 log.

Flow Type: System  |  Module: shipping  |  Nodes: 19  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate | validate | Schema/business-rule validation. |
| Query Load | query | GraphQL query — reads records. |
| Find Load Status | transform | JSONata data transform. |
| Route | switch | Multi-branch conditional router. |
| Prepare Packed Payload | transform | JSONata data transform. |
| Prepare Shipped Payload | transform | JSONata data transform. |
| Update Load Status | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Prepare New Payload | transform | JSONata data transform. |
| Prepare Shipped Payload | transform | JSONata data transform. |
| If Else | ifElse | Conditional branch (if/else). |
| Prepare Payload | transform | JSONata data transform. |
| Try Catch | tryCatch | Error-handling boundary. |
| Error Log | log | Writes a log entry. |
| Response | response | Returns the result payload to the caller. |
| Initial Log | log | Writes a log entry. |

### 5.7 Order & Load Management

Add New Shipment Line

A Screen-type flow composed of: 2 transform, 1 formDialog, 1 mutate, 1 snackbar, 2 response, 1 request, 1 executeFlow.

Flow Type: Screen  |  Module: orderTracking  |  Nodes: 9  |  Active: true

| Node | Type | Role |
|---|---|---|
| JSONata | transform | JSONata data transform. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Create Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Prepare Payload | transform | JSONata data transform. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Update Release | executeFlow | Invokes another data flow. |

Orders Web Flow

- This data flow is responsible for creating, updating, and deleting orders and order lines. The functionality was moved from the screen transform to the flow to provide greater control, simplify maintenance, and make future enhancements easier to implement.

Flow Type: Screen  |  Module: orderProcessing  |  Nodes: 69  |  Active: true

| Node | Type | Role |
|---|---|---|
| Route | switch | Multi-branch conditional router. |
| Route | switch | Multi-branch conditional router. |
| Query Order | query | GraphQL query — reads records. |
| Create a New Order | formDialog | Dialog collecting additional user input. |
| Set Context | setContext | Sets/replaces the flow context. |
| Update Order | formDialog | Dialog collecting additional user input. |
| Confirm Delete | confirm | Blocking confirmation dialog shown to the user. |
| Create Order | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Update Order | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Query Order | query | GraphQL query — reads records. |
| Order in correct Status | ifElse | Conditional branch (if/else). |
| Store Order | mergeContext | Merges values into the flow context. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Delete Order, Line and Release | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Route | switch | Multi-branch conditional router. |
| Create Order Line | formDialog | Dialog collecting additional user input. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Create Order Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Create Line Release | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Query OLR | query | GraphQL query — reads records. |
| Update OLR | formDialog | Dialog collecting additional user input. |
| Update OLR | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Store OLR | mergeContext | Merges values into the flow context. |
| Order in New Status? | ifElse | Conditional branch (if/else). |
| Query OLR | query | GraphQL query — reads records. |
| Store OLR | mergeContext | Merges values into the flow context. |
| Order in New Status? | ifElse | Conditional branch (if/else). |
| Confirm Delete | confirm | Blocking confirmation dialog shown to the user. |
| Delete OLR | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Cancel Message | snackbar | Non-blocking success/error/info message to the user. |
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Execute Flow | executeFlow | Invokes another data flow. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Q-Line | query | GraphQL query — reads records. |
| Line Exist? | ifElse | Conditional branch (if/else). |
| Create Line Release | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Store Orderline | mergeContext | Merges values into the flow context. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Response | response | Returns the result payload to the caller. |
| Can't Update | alert | Blocking informational dialog. |
| Can't Delete | alert | Blocking informational dialog. |
| Can't Delete | alert | Blocking informational dialog. |
| Update Order Line | formDialog | Dialog collecting additional user input. |
| Update Order Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query Order Line | query | GraphQL query — reads records. |
| Delete Order, Line and Release | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Success | snackbar | Non-blocking success/error/info message to the user. |
| Query Order Lines | query | GraphQL query — reads records. |
| Delete Order Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Query OLR | query | GraphQL query — reads records. |
| Store OLR | mergeContext | Merges values into the flow context. |
| Prepare Payload | transform | JSONata data transform. |
| Remove Order Line? | ifElse | Conditional branch (if/else). |
| Query Order Lines | query | GraphQL query — reads records. |
| Delete Order Line | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Prepare Payload | transform | JSONata data transform. |
| Remove Order Line? | ifElse | Conditional branch (if/else). |

Update Order Status

A System-type flow composed of: 1 request, 2 response, 1 mergeContext, 1 validate, 1 query, 2 transform, 1 mutate, 1 ifElse, 1 tryCatch, 2 log.

Flow Type: System  |  Module: orderTracking  |  Nodes: 13  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Validate | validate | Schema/business-rule validation. |
| Query Order Data | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Update Order | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Status Provided? | ifElse | Conditional branch (if/else). |
| Prepare Payload | transform | JSONata data transform. |
| Try Catch | tryCatch | Error-handling boundary. |
| Error Log | log | Writes a log entry. |
| Response | response | Returns the result payload to the caller. |
| Initial Log | log | Writes a log entry. |

Update Release Status

A System-type flow composed of: 1 request, 1 response, 1 mergeContext, 1 validate, 1 query, 1 transform, 1 mutate, 1 executeFlow.

Flow Type: System  |  Module: orderTracking  |  Nodes: 8  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Validate | validate | Schema/business-rule validation. |
| Query Release Data | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Mutate Update Releases | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Execute Flow Update Orders | executeFlow | Invokes another data flow. |

### 5.8 Order Fulfillment & Business Partners

Change Shipment Line Quantity

A Screen-type flow composed of: 1 request, 3 response, 1 mergeContext, 1 formDialog, 1 switch, 3 snackbar, 1 mutate, 1 searchTable, 1 executeFlow.

Flow Type: Screen  |  Module: logistics  |  Nodes: 13  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Route | switch | Multi-branch conditional router. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Mutate update line qty | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Snackbar | snackbar | Non-blocking success/error/info message to the user. |
| Response | response | Returns the result payload to the caller. |
| Search Table | searchTable | Executes/refreshes a table search. |
| Response | response | Returns the result payload to the caller. |
| Update Load Status | executeFlow | Invokes another data flow. |

### 5.9 Quality (Material Review Board)

MRB Adjust Inventory

A System-type flow composed of: 1 request, 1 validate, 3 mergeContext, 2 query, 2 switch, 7 response, 1 mutate, 1 ifElse.

Flow Type: System  |  Module: quality  |  Nodes: 18  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate Input | validate | Schema/business-rule validation. |
| Merge Input | mergeContext | Merges values into the flow context. |
| Query Inventory | query | GraphQL query — reads records. |
| Merge Inventory | mergeContext | Merges values into the flow context. |
| Route — Inventory Check | switch | Multi-branch conditional router. |
| Response — Inventory Not Found | response | Returns the result payload to the caller. |
| Response — Inventory Inactive | response | Returns the result payload to the caller. |
| Response — Inventory Retired | response | Returns the result payload to the caller. |
| Response — MRB Quality Hold | response | Returns the result payload to the caller. |
| Query Adjustment Type | query | GraphQL query — reads records. |
| Merge Adjustment Type | mergeContext | Merges values into the flow context. |
| Route — Adjustment Check | switch | Multi-branch conditional router. |
| Response — Adjustment Type Invalid | response | Returns the result payload to the caller. |
| Mutate — Adjust Inventory Quantity | mutate | GraphQL mutation — creates, updates, or deletes records. |
| MRB Scrap Adjustment? | ifElse | Conditional branch (if/else). |
| Response — Success | response | Returns the result payload to the caller. |
| Response — MRB Scrap Success | response | Returns the result payload to the caller. |

MRB Complete Return

A System-type flow composed of: 1 request, 1 validate, 2 mergeContext, 1 query, 2 switch, 4 response, 1 mutate.

Flow Type: System  |  Module: quality  |  Nodes: 12  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Validate Input | validate | Schema/business-rule validation. |
| Merge Input | mergeContext | Merges values into the flow context. |
| Query Shipment | query | GraphQL query — reads records. |
| Merge Shipment | mergeContext | Merges values into the flow context. |
| Route — Shipment Check | switch | Multi-branch conditional router. |
| Response — Shipment Not Found | response | Returns the result payload to the caller. |
| Response — Already Completed | response | Returns the result payload to the caller. |
| Mutate — Complete Return Shipment | mutate | GraphQL mutation — creates, updates, or deletes records. |
| MRB Driven Return? | switch | Multi-branch conditional router. |
| Response — Return Complete | response | Returns the result payload to the caller. |
| Response — MRB Return Complete | response | Returns the result payload to the caller. |

### 5.10 Configuration

Configuration Web Flow

A Screen-type flow composed of: 1 request, 1 executeFlow, 2 formDialog, 1 setContext, 9 transform, 2 query, 3 mergeContext, 2 mutate, 3 response, 1 ifElse, 1 switch.

Flow Type: Screen  |  Module: configuration  |  Nodes: 26  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Execute Flow | executeFlow | Invokes another data flow. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| Set Context | setContext | Sets/replaces the flow context. |
| Filter Options | transform | JSONata data transform. |
| Query App Config | query | GraphQL query — reads records. |
| Prepare Payload | transform | JSONata data transform. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Mutate AppConfig | mutate | GraphQL mutation — creates, updates, or deletes records. |
| Response | response | Returns the result payload to the caller. |
| Get Payload | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Update? | ifElse | Conditional branch (if/else). |
| AppConfig Data | transform | JSONata data transform. |
| AppConfig Data | transform | JSONata data transform. |
| Route | switch | Multi-branch conditional router. |
| Form Dialog | formDialog | Dialog collecting additional user input. |
| JSONata | transform | JSONata data transform. |
| Prepare Payload | transform | JSONata data transform. |
| Merge Context | mergeContext | Merges values into the flow context. |
| Query App Config | query | GraphQL query — reads records. |
| Mutate AppConfig | mutate | GraphQL mutation — creates, updates, or deletes records. |
| AppConfig Data | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| AppConfig Data | transform | JSONata data transform. |

### 5.11 Dashboard & Reporting

WMS Dashboard Search v3

A System-type flow composed of: 1 request, 1 tryCatch, 1 log, 2 response, 1 switch, 10 transform, 5 query, 4 mergeContext, 1 setContext.

Flow Type: System  |  Module: dashboard  |  Nodes: 26  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Try Catch | tryCatch | Error-handling boundary. |
| Log | log | Writes a log entry. |
| Response | response | Returns the result payload to the caller. |
| Route | switch | Multi-branch conditional router. |
| RCV — Read Filters | transform | JSONata data transform. |
| RCV — Set KPI Context | transform | JSONata data transform. |
| Response | response | Returns the result payload to the caller. |
| RCV — All KPIs | query | GraphQL query — reads records. |
| RCV - Context | mergeContext | Merges values into the flow context. |
| SHIP — Read Filters | transform | JSONata data transform. |
| SHIP - Context | mergeContext | Merges values into the flow context. |
| INV — Read Filters | transform | JSONata data transform. |
| CC — Read Filters | transform | JSONata data transform. |
| INV - Context | mergeContext | Merges values into the flow context. |
| CC - Context | mergeContext | Merges values into the flow context. |
| SHIP - All KPIs | query | GraphQL query — reads records. |
| INV - All KPIs | query | GraphQL query — reads records. |
| CC - All KPIs | query | GraphQL query — reads records. |
| SHIP — Set KPI Context | transform | JSONata data transform. |
| INV — Set KPI Context | transform | JSONata data transform. |
| CC — Set KPI Context | transform | JSONata data transform. |
| Overview - Read Filters | transform | JSONata data transform. |
| Overview - Context | setContext | Sets/replaces the flow context. |
| Overview— All KPIs | query | GraphQL query — reads records. |
| Overview— Set KPI Context | transform | JSONata data transform. |

### 5.12 Testing & Utility

DataModel Details

A System-type flow composed of: 1 request, 2 response, 1 tryCatch, 1 validate, 4 query, 3 transform, 2 setContext, 1 ifElse, 2 javascriptTransform, 1 collect.

Flow Type: System  |  Module: testing  |  Nodes: 18  |  Active: true

| Node | Type | Role |
|---|---|---|
| Request | request | Entry point — receives the invoking payload (from a screen action or another flow). |
| Response | response | Returns the result payload to the caller. |
| Error Response | response | Returns the result payload to the caller. |
| Try Catch | tryCatch | Error-handling boundary. |
| Validate Input | validate | Schema/business-rule validation. |
| Fetch Model Details | query | GraphQL query — reads records. |
| Format Response | transform | JSONata data transform. |
| Store Request | setContext | Sets/replaces the flow context. |
| If Markdown? | ifElse | Conditional branch (if/else). |
| Format as Markdown | javascriptTransform | JavaScript data transform. |
| Query - All System Models | query | GraphQL query — reads records. |
| Query - All Custom Models | query | GraphQL query — reads records. |
| Query - All Setup Custom Models | query | GraphQL query — reads records. |
| Collect | collect | Collects results from parallel branches. |
| Set Context | setContext | Sets/replaces the flow context. |
| JSONata - Execute Query | transform | JSONata data transform. |
| JSONata - Reshape Results | transform | JSONata data transform. |
| JavaScript | javascriptTransform | JavaScript data transform. |

### 6. Core Components — Screens

The package ships 43 screens, summarized below by functional area. Each row lists the screen’s module, whether it is a full screen or an embeddable widget, its primary layout template (where applicable), and the element types it is composed of.

### 6.1 Receiving

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Create Exception Modal | Widget | — | Action Button (7), Select (6), Text (5), Float (5), Container (3), Button Group (2), Form (1), Markdown (1) |
| Create Receipt Line Widget | Widget | — | Container (14), Text (12), Select (7), Action Button (5), Form (3), Icon (2), Number (2), Markdown (2), Switch (2), Float (1), Date (1), Button Group (1), Integer (1) |
| New Receipt Widget | Widget | — | Text (10), Container (4), Action Button (2), Form (1), Select (1) |
| Receipt Line Confirmation | Screen | table | Table Column (16), Action Button (6), Select (6), Container (4), Integer (2), Float (2), Form (1), Table (1), Checkbox (1), Text (1), Button Group (1) |
| Receiving | Screen | — | Table Column (46), Container (21), Scan (11), Table (10), Select (10), Action Button (8), Form (7), Resizable Panel (2), Dynamic Table Column (1), Text (1) |
| Recieving Details Modal | Widget | — | Container (2), Select (2), Action Button (2), Form (1), Text (1), Float (1), Date (1), Button Group (1), Icon (1), Integer (1) |

### 6.2 Inventory & Product Management

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Adjustment Table | Screen | table | Container (4), Action Button (4), Table Column (3), Float (2), Form (1), Table (1), Text (1), Checkbox (1), Button Group (1) |
| Basic Inventory Update Widget | Widget | — | Text (7), Container (6), Float (5), Markdown (4), JSON (4), Select (3), Measure (2), Action Button (2), Form (1), Custom Fields (1) |
| Create Inventory Widget | Widget | — | Select (6), Container (4), Integer (2), Action Button (2), Form (1), Icon (1), Switch (1), Text (1), Flow Button (1) |
| Inventory Table | Screen | table | Table Column (19), Container (9), Select (9), Float (6), Action Button (5), Text (3), Date Range (2), Switch (2), Date (2), Form (1), Table (1), Checkbox (1), Button Group (1), Cards (1), JSON (1), Resizable Panel (1) |
| Merge Inventory | Widget | — | Container (75), Text (21), Switch (11), Scan (10), Action Button (8), Form (4), Float (3), Select (3), Icon (2), Measure (2), JSON (2), Cards (1), Resizable Panel (1), Ratio Measure (1) |
| Move Inventory | Widget | — | Container (31), Text (13), Scan (10), Switch (8), Action Button (7), Select (3), Icon (3), Form (2), Cards (2), Options (1), Array Input (1), Checkbox (1), Screen Widget (1), Flow Button (1) |
| Process Table | Screen | table | Container (5), Table Column (5), Action Button (4), Text (3), Form (1), Table (1), Checkbox (1), Button Group (1) |
| Product Category Table | Screen | table | Container (4), Action Button (4), Table Column (3), Text (2), Form (1), Table (1), Checkbox (1), Button Group (1) |
| Product Details Form | Widget | — | Text (26), Container (11), Measure (9), Select (6), JSON (4), Rich Text (3), Form (2), Action Button (2), Integer (1), Markdown (1), Switch (1) |
| Product Table | Screen | table | Table Column (25), Container (8), Action Button (8), Float (8), Select (6), Text (3), Form (2), Table (2), Split Button (2), Button Group (1), Custom Fields Table Column (1), Resizable Panel (1), Switch (1) |
| Split Inventory | Widget | — | Container (40), Text (13), Action Button (6), Switch (5), Scan (3), Form (3), Flow Button (2), Float (1), Select (1), Cards (1), Icon (1), Resizable Panel (1), Measure (1) |

### 6.3 Cycle Counting

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Count Details | Screen | — | Table Column (18), Container (12), Text (10), Form (2), Table (2), Action Button (2), Flow Button (2), Progress Bar (1) |
| Count Location Selector | Screen | — | Container (11), Text (9), Menu Button (3), Form (2), Cards (2), Progress Bar (2), Scan (1) |
| Count Selector | Screen | — | Text (8), Container (6), Form (2), Cards (1) |
| Count Table | Screen | table | Table Column (8), Action Button (5), Container (4), Date Range (4), Integer (2), Form (1), Table (1), Checkbox (1), Select (1), Flow Button (1), Button Group (1) |
| Execute Cycle Count | Screen | — | Container (13), Form (5), Number (3), Select (2), Action Button (2), Flow Button (2), Text (1), Progress Bar (1), Measure (1), Scan (1) |

### 6.4 Picking

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Execute Pick | Screen | — | Container (43), Text (38), Action (26), Scan (11), Switch (8), Action Button (7), Icon (6), Flow (5), Form (3), Cards (3), Select (3), JSON (3), Button Group (3), Image (2), Options (1), Array Input (1), Checkbox (1), Screen Widget (1), Flow Button (1) |
| Pick Details | Widget | — | Text (90), Container (76), Action (31), TableColumn (30), Cards (11), Table (8), Button Group (7), Table Column (7), Form (3), Date (2), Tabs (1), Flow (1) |
| Pick Management | Screen | table, table, table | Table Column (33), Action Button (21), Container (19), Select (9), Table (6), Date Range (6), Form (4), Text (4), Button Group (3), Action (3), GraphQL Where Input (1), Dynamic Table Column (1) |
| Unpick Modal | Widget | — | Text (4), Container (3), Action (3), Form (1), Select (1), Rich Text (1), Scan (1) |

### 6.5 Packing

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Packing | Screen | — | Table Column (173), Container (58), Table (38), Action (37), Text (21), Form (11), Button Group (9), Select (9), Resizable Panel (2), Scan (2), Icon (1), GraphQL Where Input (1), Options (1) |

### 6.6 Shipping & Carrier Management

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| SCAC | Screen | — | Table Column (7), Container (5), Text (3), Form (2), Action (2), Switch (2), Resizable Panel (1), Table (1), Button Group (1) |
| Ship Management | Screen | — | Container (6), Table Column (6), Action (4), Resizable Panel (2), Form (2), Button Group (2), Text (2), Table (2) |

### 6.7 Order & Load Management

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Create Load Form | Widget | form | Text (73), Container (69), Icon (19), Table Column (15), Action Button (7), Cards (6), Float (6), Switch (6), Display (4), Form (2), Date & Time (2), Tab Bar (2), Table (2), Data Tree (2), Decimal (2), Select (2), Measure (1), Rich Text (1), Resizable Panel (1), Dynamic Column (1), Widget (1) |
| Load Management | Screen | table | Table Column (22), Text (13), Container (9), Action (6), Action Button (4), Select (3), Table (2), Form (1), Button Group (1), Cards (1), Date & Time (1), Menu (1) |
| Orders | Screen | table | Table Column (207), Action (68), Container (50), Table (25), Action Button (21), Select (11), Text (9), Switch (4), Button Group (3), Form (2), Checkbox (1), Resizable Panel (1), Tabs (1) |

### 6.8 Order Fulfillment & Business Partners

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Allocate Inventory to Lines | Widget | — | Table Column (6), Container (5), Action (4), Options (2), Accordion (1), Table (1), Form (1), Select (1) |
| Business Partner Address Detail Form | Widget | — | Text (14), Container (8), Markdown (7), JSON (4), Action Button (4), Form (1), Address (1) |
| Business Partner Detail Form | Widget | — | Text (17), Container (7), Markdown (6), Duration (6), JSON (4), Switch (3), Select (2), Float (2), Action Button (2), Form (1) |
| Business Partners Table | Widget | table | Table Column (10), Container (9), Action Button (8), Text (3), Checkbox (3), Form (2), Table (2), Button Group (1) |
| Order Line Create Widget | Widget | — | Text (14), Container (12), Markdown (7), Action Button (5), JSON (4), Select (3), Form (1), Address (1), Float (1), Action (1) |
| Select Order Releases | Widget | — | Table Column (8), Container (3), Action (2), Table (1), Form (1), Select (1) |
| Shipping Mobile | Screen | — | Container (8), Action (8), Table Column (5), Form (1), Select (1), Date & Time (1), Table (1) |

### 6.9 Site Management

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Site Management | Screen | table | Text (197), Container (196), Table Column (125), Switch (78), Action Button (35), JSON (30), Select (26), Checkbox (20), Form (19), Action (12), Options (11), Table (10), Measure (9), Tab Bar (4), Address (4), Date Range (2), Combobox (2), Button Group (2), Resizable Panel (2), Split Button (1) |

### 6.10 Configuration

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Configuration | Screen | — | Container (22), Text (19), Action (11), Options (6), Table Column (3), Form (1), Table (1), Select (1) |

### 6.11 Dashboard & Reporting

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| WMS Dashboard | Screen | — | Table Column (74), Container (43), Text (29), Table (12), Chart (9), Form (3), Date Range (1), Action (1) |

### 6.12 Testing & Utility

| Screen | Type | Template | Key Elements |
|---|---|---|---|
| Test | Screen | — | Form (1), Container (1), Select (1) |

### 7. Document Designs

The package ships the following printable document designs, generated by shipping and receiving flows:

| Document Design | Description |
|---|---|
| Bill Of Lading | — |
| Box Content Label | — |
| Commercial Invoice | — |
| Packing List | — |
| Pick Sheet | — |
| Shipping Label | — |

### 8. Application Configuration & Reference Data

### 8.1 Application Configurations

| Configuration | Description | Key Settings |
|---|---|---|
| Company Information | — | Name, Address1, Address2, City, StateProvince, PostalCode, Country, Phone, Email, Website, LogoUrl |
| Move Inventory Options | This will decide if when moving inventory to a storage unit, will the serial number be moved OR will the inventory be merged with any existing inventory in the storage unit. | method, formTitle, options |
| Pack Options | Options for how to peform packing. | method, formTitle, options |
| Pick Options | Options to define how the pick method is executed. | method, formTitle, options |

### 8.2 Status & Type Reference Data

The following reference data sets ship pre-populated and drive status/type selection and business logic throughout the package:

#### 8.2.1 Order Status

| Value | Description |
|---|---|
| Cancelled | This Order has been cancelled and can not be operated against. |
| Fulfilled | The Order has been completed. |
| New | The Order has just been created and is not ready to be operated against. |
| Packed | Occurs when all items within the order are packed. |
| Partial Fulfillment | An Order has been operated against but not completed. |
| Partial Pick | Picking is in process but not complete. |
| Pending Fulfillment | Indicates that the Order is ready to be operated against. |
| Picked | Picking has been completed. |
| Shipped | Occurs when the order has been shipped. |

#### 8.2.2 Order Type

| Value | Description |
|---|---|
| Purchase | — |
| Sale | — |

#### 8.2.3 Order Line Release Status

| Value | Description |
|---|---|
| Canceled | Indicates that the release has been canceled and can not be operated against. |
| Fulfilled | Indicates that the release has had its entire quantity operated against. |
| Partially Fulfilled | Indicates that the release has been operated against but not completed. |
| Pending Fulfillment | The starting status of a release. Indicates that the release is ready to be operated against. |

#### 8.2.4 Order Line Release Type

| Value | Description |
|---|---|
| Firm | This one’s not just committed — it’s already picking out furniture. Locked, loaded, and ready to ship. You can bet your forklift on it. |
| Forecasted | It says it’s coming... probably. Like a weather report for your warehouse — not guaranteed, but worth keeping an eye on. |
| Planned | It’s penciled in — there’s a date on the calendar, but don’t carve it in stone just yet. Think of it as a strong ‘maybe’ with a time attached. |

#### 8.2.5 Load Status

| Value | Description |
|---|---|
| New | This is the status that loads are first created in. These loads need to still be packed |
| Packed | Loads in this status have been fully packed and are ready to ship |
| Picked | The load has been completed picked. |
| Shipped | Loads in this have already been shipped. |
| Short Shipped | Loads in this status have been shipped but at least one shipment line was shipped with out meeting the expected quantity |

#### 8.2.6 Pick Status

| Value | Description |
|---|---|
| Cancelled | A pick that has been cancelled. |
| Completed | A pick in which all steps have been completed. |
| New | A pick that has been created, but not yet configured. |
| Ready | A pick that had been configured, but not yet started.. |
| Started | A pick with at leaste one pick step that has been picked. |

#### 8.2.7 Pick Step Status

| Value | Description |
|---|---|
| Completed | A pick step that has been fully completed and dropped off. |
| New | A pick step that has been configured, but not yet started. |
| Picked | A pick step that has been completely picked, but not yet dropped off. |
| Ready | A pick step that has been released and can be picked. |
| Started | A pick step that has been started, but not completely picked. |

#### 8.2.8 Unpick Reason

| Value | Description |
|---|---|
| Damaged | Inventory was damaged before or after picking |
| Error | Picked incorrect inventory |
| Other | Other unpick reason |
| Quantity Descrepancy | Picked incorrect quantity |

#### 8.2.9 Inventory Status

| Value | Description |
|---|---|
| Damaged | — |
| Hold | — |
| Lost | — |
| Ok | — |
| Packed | — |
| Retired | — |
| Shipped | — |

#### 8.2.10 Handling Unit Type

| Value | Description |
|---|---|
| Bag | — |
| Box | — |
| Pallet | — |
| Tote | — |

#### 8.2.11 Transaction Type

| Value | Description |
|---|---|
| Inventory Merge | Indicates that the a qty from one inventory record has been moved to an already exsisting inventory record. |
| Inventory Move | Indcates that the inventory was move from one storage unit to another. |
| Inventory Split | Indicates that an inventory has had quantity split of from it and a new inventory record created. |

#### 8.2.12 Shipment Type

| Value | Description |
|---|---|
| Return | A return shipment to the source that it came from, sometimes because of a defect in the product from a supplier, others because the job work on a customer's item is complete and ready to send back |
| Sale | A sale to a customer, direct trade |

#### 8.2.13 Standard Carrier Alpha Codes (SCAC)

The package ships 128 pre-populated SCAC records covering major North American carriers, used by the Shipping and Load Management screens for carrier selection.

### 9. Status & Lifecycle Reference

### 9.1 Order Lifecycle

An Order progresses through status values as its lines are released, picked, packed, and shipped:

- New → the order has just been created and is not yet ready to be operated against.

- Pending Fulfillment → the order is ready to be operated against (releases, picks, etc.).

- Partial Pick / Picked → picking is in process or has been completed for the order.

- Packed → all items within the order are packed.

- Partial Fulfillment / Fulfilled → the order has been operated against but not completed, or has been completed.

- Shipped → the order has been shipped.

- Cancelled → the order has been cancelled and can no longer be operated against.

### 9.2 Receipt & Receiving Lifecycle

A Receipt represents one Bill of Lading/Packing Slip/ASN, with one or more Receipt Lines. Receipt Lines are confirmed individually; discrepancies between expected and received quantity or product are captured as a ReceiptException rather than blocking the receipt.

### 9.3 Pick & Pick Step Lifecycle

A Pick is broken into PickStep records, each progressing independently:

- Pick: New → Ready → Started → Completed (or Cancelled).

- PickStep: New → Ready → Started → Picked → Completed.

When picked inventory cannot be fulfilled as planned, an Unpick record is created with a reason (Damaged, Error, Other, Quantity Discrepancy), and a NonConformancePublishMessage flow raises the condition for downstream handling.

### 9.4 Load Lifecycle

- New → the load has been created and its shipments still need to be picked/packed.

- Picked → the load has been completely picked.

- Packed → the load has been fully packed and is ready to ship.

- Shipped → the load has already been shipped.

- Short Shipped → the load has shipped but at least one shipment line did not meet the expected quantity.

### 9.5 Cycle Count Lifecycle

A Count is created from CountParameters (the filter criteria used to select inventory in scope) via Start Count and Create Count Lines, which produces CountLine records (expected quantities) and CountLineInventory records (individual counted line items). Counts can be excluded, recounted, or completed (applying the counted changes to inventory) or cancelled.

### 10. Sequences

The package ships the following auto-numbering sequences:

| Sequence | Format | Increment |
|---|---|---|
| BoL Number | BoL-000000 | 1 |
| CommercialInvoice | INV000000 | 1 |
| Count | — | 1 |
| CountLine | — | 1 |
| CountParameter | — | 1 |
| Count Up from One | — | 1 |
| HandlingUnit | HU00000000 | 1 |
| Inventory | S000000000000 | 1 |
| Load | L00000000 | 1 |
| Order | OR000000000000 | 1 |
| OrderLine | OL00000000 | 1 |
| OrderLineRelease | OLR00000000 | 1 |
| Pick | P00000000 | 1 |
| PreferredProductStroageUnitPriority | — | 1 |
| Receipt | R | 1 |
| ReceiptException | — | 1 |
| ReceiptLine | — | 1 |
| Shipment | SH00000000 | 1 |
| ShipmentLine | SL00000000 | 1 |

### 11. Package Dependencies & Prerequisites

The package declares no external package dependencies in its manifest. Prerequisites for operating the package:

- Platform version 2026.7.0 or higher.

- Business Partner and Site/Storage reference data (Areas, Storage Zones, Storage Units) should be configured before receiving or shipping transactions are run.

- Application Configuration records (Company Information, Pick/Pack/Move Inventory Options) should be reviewed and set for the target tenant.

- Document Designs (Bill of Lading, Shipping Label, etc.) should be reviewed against the target company’s branding and compliance requirements before go-live.

### 12. Glossary

| Term | Definition |
|---|---|
| ASN | Advance Shipping Notice — notification of inbound goods, referenced by a Receipt. |
| BOL | Bill of Lading — a legal shipping document describing goods and terms of transport. |
| Cycle Count | An ongoing inventory audit process that counts a subset of inventory on a rolling basis rather than a full physical count. |
| Handling Unit | A physical packaging unit (bag, box, pallet, tote) containing one or more inventory items. |
| Load | A single shipment or collection of shipments transported together. |
| Lot | A logical grouping of inventory from the same batch/heat/production run. |
| MRB | Material Review Board — the process for handling non-conforming inventory and returns. |
| Order Line Release | A specific ship/firm/planned schedule for a quantity of an order line. |
| Pick Step | An individual pick action against a specific storage location, part of a larger Pick. |
| SCAC | Standard Carrier Alpha Code — a unique code identifying a transportation carrier. |
| Storage Zone / Storage Unit | The physical warehouse location hierarchy used to place and locate inventory. |
