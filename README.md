# Fuuz WMS Accelerator — modular

Warehouse management: inventory, receiving, picking, shipping and location control.

**`WMS Accelerator - Mfg@0.0.7`** · platform `2026.7.0` · spec `2.0.0`

This is the **modular** publication: the package unpacked into its components so
each module can be read, reviewed and diffed. The monolithic package is in
[`package/`](package/) and is what you install.

## Modules (15)

| Module | | Components |
|---|---|---|
| `configuration` | Configuration | 1 dataFlows · 1 screens |
| `cycleCounting` | Cycle Counting | 9 dataFlows · 5 screens |
| `dashboard` | Dashboard | 1 dataFlows · 1 screens |
| `inventory` | Inventory | 1 dataFlows · 3 screens |
| `inventoryTracking` | Inventory Tracking | 6 dataFlows · 8 screens |
| `logistics` | Logistics | 1 dataFlows · 7 screens |
| `orderProcessing` | Order Processing | 1 dataFlows · 3 screens |
| `orderReceiving` | Order Receiving | 5 dataFlows · 6 screens |
| `orderTracking` | Order Tracking | 3 dataFlows |
| `packing` | Packing | 4 dataFlows · 1 screens |
| `picking` | Picking | 5 dataFlows · 4 screens |
| `quality` | Quality | 2 dataFlows |
| `shipping` | Shipping | 8 dataFlows · 2 screens |
| `siteManagement` | Site Management | 1 screens |
| `testing` | Testing | 1 dataFlows · 1 screens |

## Components without a module

Only `dataFlows` and `screens` carry `moduleId`. These types carry none, so they
are filed by type rather than guessed at:

| Path | Count |
|---|---:|
| `dataModels/` | 52 |
| `documentDesigns/` | 6 |
| `data/` | 17 |

## Layout

```
manifest.json                 name, version, platformVersion
definition.json               package definition + selections
modules/<module>/<type>/*.json
dataModels/ documentDesigns/ data/
package/                      the installable .fuuz
docs/                         guides and technical overview
```

## Installing

Install `package/WMS Accelerator - Mfg@0.0.7 1.fuuz` through the Fuuz platform's package
installer. The exploded tree is for reading and reviewing; it is not the install
artifact.
