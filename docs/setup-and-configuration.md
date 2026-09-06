# WMS Accelerator — Setup & Configuration Guide

> Converted from the v1.0.0 Word document shipped with the accelerator package.

### 1. Purpose & Scope

### 2. Prerequisites

### 3. Configuration Sequence

### 4. Step 1 — Configure the Site Structure

### 5. Step 2 — Configure Business Partners

### 6. Step 3 — Configure Products

### 7. Step 4 — Set Application Configuration

### 7.1 Company Information

### 7.2 Move / Pack / Pick Method Options

### 8. Step 5 — Review Reference Data

### 9. Step 6 — Verify with a Test Transaction

### 10. Monitoring — WMS Dashboard

### 11. Status Quick Reference

### 11.1 Order Status

### 11.2 Load Status

### 11.3 Pick Status

### 11.4 Pick Step Status

### 1. Purpose & Scope

This guide provides step-by-step instructions for configuring the Fuuz WMS Accelerator (package version 0.0.7) in a tenant where the package is already installed. It covers setting up the physical site structure, business partners, products, and the operational options that control how moving, packing, and picking behave, followed by verification with a test transaction and ongoing monitoring.

Building custom integrations to an external ERP/host system (for creating orders, publishing inventory, or exchanging shipment status) is outside the scope of this guide. Refer to the WMS Accelerator AS IS Build Document for the full data model and data flow reference, and to the Fuuz Integration Orchestrator documentation for connecting the WMS to external systems.

### 2. Prerequisites

Confirm the following before starting configuration:

| Prerequisite | Detail |
|---|---|
| Package installed | WMS Accelerator - Mfg 0.0.7 imported into the tenant (Inventory, Order Fulfillment, Receiving, Site Management, and Materials Management modules are visible in the menu). |
| Reference data reviewed | Order/Load/Pick/Inventory statuses, Handling Unit Types, Transaction Types, and the 128 pre-populated Standard Carrier Alpha Codes ship active by default — review section 5 before go-live. |
| User permissions | User roles/permissions for the Site Management, Business Partners, Product, Configuration, Receiving, Picking, Packing, and Ship Management screens. |
| Document branding | Company logo and address details on hand to populate Company Information before printing the shipped Document Designs (Bill of Lading, Packing List, Shipping Label, etc.). |

### 3. Configuration Sequence

- Configure the site structure — Areas, Storage Zones, and Storage Units (section 4).

- Configure Business Partners and their addresses — customers, suppliers, and carriers (section 5).

- Configure Products and, where used, their preferred storage units (section 6).

- Set Application Configuration — Company Information and the Move/Pack/Pick method options (section 7).

- Review pre-populated reference data — statuses, handling unit types, and carrier codes (section 8).

- Verify the setup with a test transaction end-to-end: receive, put away, allocate, pick, pack, and ship (section 9).

- Monitor ongoing activity in the WMS Dashboard (section 10).

### 4. Step 1 — Configure the Site Structure

The Site Management screen is the primary administrative screen for the warehouse’s physical structure. It exposes create/edit dialogs for Areas, Sub-Areas, Storage Zones, and Storage Units, and should be configured before any receiving, putaway, or picking activity occurs, since Receipt Lines, Inventory, and Pick Steps are all placed against a Storage Unit.

Configure the hierarchy top-down:

| Level | Required | How to Configure |
|---|---|---|
| Area | Yes | A physical, geographical, or logical grouping within the site. Set the Code and Name, and the applicable flags — Storage (for storage areas), Process, Production Cell, or Production Line — depending on the area’s purpose. |
| Storage Zone | Yes (for storage areas) | Created under an Area. Flag the zone’s characteristics as applicable: Fast Moving, Slow Moving, Cross Docking, Hazardous, High Value, Refrigerated, Oversized, Kitting, Overflow, or Return Processing — these flags are used for filtering and auto-assignment. |
| Storage Unit | Yes | Created under a Storage Zone. Set the Code, Name, and dimensions (Length, Width, Height). Flag the unit’s allowed uses — Storage, Receiving, Shipping, Picking, Packing, Cross Dock, Quality Control, Returns — and optionally a Default Inventory Status. |

TIP  Set ProductPreferredStorageUnit records (section 6) once products and storage units both exist, to drive auto-assignment and filtering by priority when putting away or picking a given product.

### 5. Step 2 — Configure Business Partners

Business Partners represent the customers, suppliers, and other external parties referenced by Orders and Shipments. Configure them, and their addresses, in the Business Partners screen before creating orders or receipts that reference them.

| Field | Required | How to Configure |
|---|---|---|
| Code | Yes | Short, unique abbreviated code for the business partner. |
| Name | Yes | Longer / legal name of the business partner. |
| Customer / Supplier (flags) | Yes | Flag whether this partner is a Customer, a Supplier, or both. |
| Credit Limit | No | Optional credit limit tracked against the partner. |
| Payment Terms | No | Free-text payment terms. |
| Firm / Planned / Forecast Horizon | No | Durations that define how far out newly created Order Line Releases are considered Firm, Planned, or Forecasted relative to their due date. |
| Allow Transfer | No | Whether inventory transfers are permitted for this partner. |
| Responsible User | No | The internal user responsible for the relationship. |

For each Business Partner, add one or more Business Partner Addresses (endpoint locations for orders and shipments):

| Field | Required | How to Configure |
|---|---|---|
| Code | Yes | Short code for the address. |
| Text Address | Yes | Unstructured address string. |
| Address (structured) | No | Structured Address fields, used where downstream documents or carriers require a structured format. |
| Packaging Note / Shipping Note | No | Free-text handling instructions surfaced during packing and shipping. |
| Partner Location ID | No | External identifier for the location (e.g., a ship-to code from the partner’s own system). |

### 6. Step 3 — Configure Products

Product is the common item/SKU-level model referenced by inventory, receiving, and order lines. Configure products in the Product Table / Product Details Form before receiving inventory against them.

| Field | Required | How to Configure |
|---|---|---|
| Code | Yes | Human-readable, unique shorthand name for the product. |
| Name / Description | No | Full human-readable name and paragraph-length description. |
| GTIN / SKU / UPC | No | Standard product identifiers — GTIN (global trade item number), SKU (internal identifier), and UPC (North American barcode identifier). |
| Number / Revision | No | Product/model number and revision. |
| Length / Width / Height / Weight | No | Physical dimensions as Measure values, used for storage-unit fit and shipping calculations. |
| Shelf Life Days | No | Shelf life in calendar days; 0 = no expiration. |
| Active | No | Whether the product is currently in use. Defaults to true. |

OPTIONAL  Add ProductPreferredStorageUnit records to define, per product, which Storage Units it is compatible with and in what priority order — used by putaway and picking auto-assignment logic.

### 7. Step 4 — Set Application Configuration

Open the Configuration screen to review and set the four tenant-level Application Configuration records shipped with the package. Each is edited through the Configuration Web Flow’s form dialogs.

### 7.1 Company Information

Company Information supplies the Name, Address, Phone, Email, Website, and Logo URL used on printed Document Designs (Bill of Lading, Packing List, Shipping Label, etc.). The package ships this record populated with example/placeholder values — replace every field with your actual company details before printing or sharing any generated document.

### 7.2 Move / Pack / Pick Method Options

Three configuration records define how core operator workflows behave. Each ships with a default method and a fixed set of selectable options:

| Configuration | Default Method | Available Options | Effect |
|---|---|---|---|
| Move Inventory Options | move | Move (relocate the record), Merge (combine with existing inventory in the destination storage unit) | Determines whether moving inventory to a storage unit relocates the record as-is or merges it into existing inventory already there. |
| Pack Options | order | Order (pack by order), Load (pack by load) | Determines whether packing is organized around a single order or an entire load. |
| Pick Options | fifo | FEFO (first-expired, first-out — by expiresAt), FIFO (first-in, first-out — by createdAt), Manufactured (by manufacturedAt), None | Determines which inventory is selected first when multiple lots/inventory records can fulfill a pick. |

IMPORTANT  These options are tenant-wide switches, not per-order settings — review them with warehouse operations before go-live, since changing the Pick method changes which inventory is offered first for every pick going forward.

### 8. Step 5 — Review Reference Data

The package ships a substantial set of reference data pre-populated and active. Review the following before go-live and adjust/deactivate any values that do not apply to your operation (see the AS IS Build Document, section 8, for the complete descriptions of each value):

| Reference Data Set | Review Guidance |
|---|---|
| Order / Order Line Release statuses and types | Drive order and release lifecycle. Generally left as shipped. |
| Load / Pick / Pick Step statuses | Drive load and picking lifecycle. Generally left as shipped. |
| Inventory Status | Controls inventory usability (Ok, Damaged, Hold, Lost, Packed, Retired, Shipped). Review the usable/inventoryUse flags if adding custom statuses. |
| Handling Unit Type | Bag, Box, Pallet, Tote — extend if your operation uses other container types. |
| Unpick Reason | Damaged, Error, Other, Quantity Discrepancy — extend if additional reasons are tracked. |
| Standard Carrier Alpha Codes (SCAC) | 128 major North American carriers ship pre-populated. Add any carrier missing from the list in the SCAC screen before assigning it to a Load or Shipment. |

### 9. Step 6 — Verify with a Test Transaction

Confirm the setup end-to-end by walking a single unit of inventory through the full warehouse cycle:

- Receiving — open Receiving and create a Receipt against a configured Business Partner (supplier), adding a Receipt Line for a configured Product. Confirm the line (Confirm Receipt Line) and verify the resulting Inventory record is created in the expected Storage Unit.

- Order & Allocation — create a Sale Order (Orders screen) for the same product and a customer Business Partner. Confirm an Order Line Release is generated, and allocate it to a Pick using Allocate Inventory to Lines.

- Picking — open Execute Pick / Pick Details and complete the generated Pick Steps. If inventory cannot be fulfilled as expected, confirm the Unpick Modal correctly records a reason.

- Packing — open Packing and pack the picked inventory per the configured Pack Options method (by Order or by Load), producing a Handling Unit.

- Shipping — open Ship Management, assign a carrier (SCAC), and confirm a Load is created/updated with the correct status progression (New → Picked → Packed → Shipped).

- Documents — generate a Bill of Lading and Packing List for the shipment and confirm Company Information renders correctly.

NOTE  The exact screens and buttons available depend on which functional areas your tenant has enabled from the accelerator; the WMS Dashboard (section 10) is the fastest way to confirm the transaction completed successfully.

### 10. Monitoring — WMS Dashboard

The WMS Dashboard screen, fed by the WMS Dashboard Search v3 data flow, provides a live operational view. Use it as the first place to check after go-live and during day-to-day operation to confirm receipts, orders, picks, and shipments are progressing as expected.

### 11. Status Quick Reference

### 11.1 Order Status

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

### 11.2 Load Status

| Value | Description |
|---|---|
| New | This is the status that loads are first created in. These loads need to still be packed |
| Packed | Loads in this status have been fully packed and are ready to ship |
| Picked | The load has been completed picked. |
| Shipped | Loads in this have already been shipped. |
| Short Shipped | Loads in this status have been shipped but at least one shipment line was shipped with out meeting the expected quantity |

### 11.3 Pick Status

| Value | Description |
|---|---|
| Cancelled | A pick that has been cancelled. |
| Completed | A pick in which all steps have been completed. |
| New | A pick that has been created, but not yet configured. |
| Ready | A pick that had been configured, but not yet started.. |
| Started | A pick with at leaste one pick step that has been picked. |

### 11.4 Pick Step Status

| Value | Description |
|---|---|
| Completed | A pick step that has been fully completed and dropped off. |
| New | A pick step that has been configured, but not yet started. |
| Picked | A pick step that has been completely picked, but not yet dropped off. |
| Ready | A pick step that has been released and can be picked. |
| Started | A pick step that has been started, but not completely picked. |
