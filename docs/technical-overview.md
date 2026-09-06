# WMS Accelerator — Technical Overview

> Converted from the technical overview deck shipped with the accelerator package.

## FUUZ WMS ACCELERATOR

- Technical Overview — Warehouse Operations, Data Model & Fulfillment Flow
- FUUZ

## What It Is — and What It Is Not

- Clear boundaries: the accelerator owns warehouse operations end to end, from receiving through shipping.
- WHAT IT IS
- • A complete operations backbone — receiving, inventory, cycle counting, picking, packing, and shipping in one accelerator.
- • Configuration-driven — Move, Pack, and Pick behavior are set through Application Configuration, not code.
- • Master-data anchored — Business Partners, Products, and the Area / Zone / Storage Unit hierarchy ground every transaction.
- • Status-tracked end to end — Order, Load, Pick, and Pick Step each carry an explicit, auditable lifecycle.
- • Document-ready — ships six printable warehouse and shipping document designs out of the box.
- WHAT IT IS NOT
- • Not a rate-shopping TMS — carriers are assigned from pre-populated SCAC reference data, not live carrier-rate integration.
- • Not an ERP — Orders and Business Partners are typically created from or synchronized with an external ERP/host system.
- • Not a fixed process — Pick, Pack, and Move methods, and the site structure, are configured per warehouse.
- • Not an integration layer by itself — connecting to an external ERP/host uses the Fuuz Integration Orchestrator alongside it.
- • Not locked to one industry — the accelerator models a generic warehouse operation.

## Core Design: The Inventory Record

- Every unit of inventory is a tracked record — location, status, and history are never implicit.
- • Tied to a physical location — Inventory always references a Product, Lot, quantity, and InventoryStatus, placed at a specific Storage Unit.
- • Every change is traceable — merges, splits, and moves are written to InventoryTrace rather than silently overwriting the record.
- • Configurable selection — the Pick Options method (FIFO, FEFO, Manufactured, or None) determines which inventory record is offered first when more than one can fulfill a pick.
- Inventory
- id INV000000042
- productId FG-10245
- lotId LOT-2026-08
- quantity 480
- inventoryStatusId ok
- storageUnitId A1-Z2-SU014
- handlingUnitId HU-88213

## Data Model: The Order-to-Cash Backbone

- 52 reference models ship in the package — six form the transactional spine that the rest support.
- Order
- A Purchase or Sale header whose lines generate Order Line Releases for allocation.
- Receipt
- One record per inbound BOL/ASN; Receipt Lines confirm inventory as it arrives.
- Inventory
- The tracked unit of product, lot, quantity, status, and storage location.
- Pick
- Broken into Pick Steps against specific storage locations to fulfill a release.
- Shipment
- The end point — customer, supplier, or transfer — that a Load’s goods are bound for.
- Load
- A single shipment or collection of shipments transported and tracked together.

## The Packaged Flows

- 49 data flows implement warehouse operations, grouped by functional area.
- Cycle Counting
- 9
- Shipping & Carrier Management
- 8
- Inventory & Product Management
- 7
- Receiving
- 5
- Picking
- 5
- Order & Load Management
- 4
- Packing
- 4
- Quality (Material Review Board)
- 2

## Transaction Lifecycle & Status Machine

- Order, Load, and Pick each progress through an explicit, seeded set of status values.
- Order
- New
- →
- Pending Fulfillment
- →
- Partial Pick
- →
- Picked
- →
- Packed
- →
- Shipped
- Load
- New
- →
- Picked
- →
- Packed
- →
- Shipped
- Pick
- New
- →
- Ready
- →
- Started
- →
- Completed
- Pick Step
- New
- →
- Ready
- →
- Started
- →
- Picked
- →
- Completed
- Exception paths: Cancelled (Order/Pick), Short Shipped (Load), and Unpick (Damaged, Error, Other, Quantity Discrepancy) branch off the happy path at any point.

## Configuration Options: Move, Pack & Pick

- Three tenant-wide switches control core operator behavior — set once, applied everywhere.
- Move Inventory
- Default: move
- Move
- Merge
- Pack
- Default: order
- Order
- Load
- Pick
- Default: fifo
- FEFO
- FIFO
- Manufactured
- None
- These are tenant-wide settings, not per-order options — changing the Pick method changes which inventory is offered first for every pick going forward.

## End-to-End Flow: Receive to Ship

- Your operation plugs into the same six stages the package ships screens and flows for.
- 1
- Receiving
- Confirm Receipt Lines against a Business Partner; Inventory is created in the assigned Storage Unit.
- 2
- Putaway & Inventory
- Move or merge inventory to its final location; every change is written to InventoryTrace.
- 3
- Order & Allocation
- Order Line Releases are generated and allocated to a Pick.
- 4
- Picking
- Pick Steps are executed per the configured Pick method (FIFO/FEFO/Manufactured/None).
- 5
- Packing
- Picked inventory is packed into Handling Units per the configured Pack method (Order/Load).
- 6
- Shipping
- A carrier (SCAC) is assigned, the Load status progresses, and shipping documents are generated.

## Observability: Screens & Dashboard

- 43 screens cover every functional area; the WMS Dashboard gives a single live view across all of them.
- WMS Dashboard
- Live operational view fed by the WMS Dashboard Search v3 flow.
- Receiving
- Receipt intake, line confirmation, and exception handling.
- Pick / Ship Management
- Execute picks, manage loads, and assign carriers.
- Site & Business Partners
- Configure the physical warehouse and its trading partners.

## Complete Visibility, Out of the Box

- What ships in the package, at a glance.
- 52
- Data Models
- 49
- Data Flows
- 43
- Screens
- 6
- Document Designs
- 128
- Carrier Codes
- 19
- Sequences

## Why This Architecture Holds Up

- Design choices that keep the warehouse honest as volume and complexity grow.
- Status-driven at every level
- Order, Load, Pick, and Pick Step each carry an explicit lifecycle rather than being inferred from other data — nothing is “probably done.”
- Centralized master data
- Products, Business Partners, and the Area/Zone/Storage Unit hierarchy anchor every transaction, preventing divergence across screens and flows.
- Full audit trail
- Inventory merges, splits, and moves are written to InventoryTrace rather than overwriting history.
- Configuration over code
- Move, Pack, and Pick behavior are tenant-level settings, changeable without touching a data flow.

## Why Teams Choose the WMS Accelerator

- What partners and prospective customers get on day one.
- 1
- Ready on day one
- Receiving through shipping work against seeded reference data — statuses, carrier codes, and handling unit types ship pre-populated.
- 2
- Configuration, not development
- Move, Pack, and Pick behavior and the site structure are set through screens, not custom flows.
- 3
- Nothing gets lost
- Every transaction is tracked through an explicit status and a traceable inventory history.
- 4
- Pairs with the Integration Orchestrator
- Connect to an external ERP/host system without changing the warehouse floor process.

## From First Site to Full Network

- The accelerator is installed once per environment. Each new site follows the same three-step onboarding.
- 1
- Site & Partners
- Configure Areas, Storage Zones, Storage Units, and Business Partners.
- 2
- Products & Configuration
- Load products; set Move/Pack/Pick methods; review reference data.
- 3
- Go-Live
- Verify with a test transaction end to end, then hand off to daily operations via the WMS Dashboard.
- FUUZ
