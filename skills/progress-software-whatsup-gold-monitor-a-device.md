---
name: WhatsUp Gold — inspect a device and manage its monitors
description: Authenticate to a WhatsUp Gold server, walk device groups to a device, read its status and roles, and create or assign an active monitor.
api: openapi/progress-software-whatsup-gold-openapi-original.json
operations:
  - DeviceGroup_ListGroups
  - DeviceGroup_SearchDevicesByGroup
  - Device_GetOverview
  - Device_GetStatus
  - Device_GetMonitors
  - Monitor_GetRegisteredTypes
  - Monitor_CreateMonitor
  - Device_UpdateMonitorById
---

# WhatsUp Gold — inspect a device and manage its monitors

## Before you start

The WhatsUp Gold REST API runs on **the customer's own WhatsUp Gold host, on port 9644**:
`https://{whatsupgold-host}:9644/api/v1`. The published spec's `host` field is an internal Progress
build address (`10.40.67.158:9644`) — ignore it.

Auth is an **OAuth 2.0 password grant**. The spec's `tokenUrl` is `http://localhost:8734/...`, also a
build artifact; the real token endpoint is `/api/v1/token` on your own host. Every operation in this
API declares an explicit `Authorization` header parameter (167 of 168 of them).

## 1. Walk to the device

- `GET /api/v1/device-groups/-` (`DeviceGroup_ListGroups`) — the group tree.
- `GET /api/v1/device-groups/{groupId}/devices/-` (`DeviceGroup_SearchDevicesByGroup`) — devices in a
  group. Use this, **not** `DeviceGroup_SearchDevices`, which is the one operation in this contract
  marked `deprecated: true`.

Pagination here is cursor-shaped: `pageId` plus `limit`, with `sortBy` / `sortByDir` and
`groupBy` / `groupByDir`.

## 2. Read the device

- `GET /api/v1/devices/{deviceId}` (`Device_GetOverview`)
- `GET /api/v1/devices/{deviceId}/status` (`Device_GetStatus`)
- `GET /api/v1/devices/{deviceId}/monitors/-` (`Device_GetMonitors`)
- `GET /api/v1/devices/{deviceId}/roles/-` (`Device_GetRoles`)

## 3. Create or assign a monitor

- `GET /api/v1/monitors/-/config/supported-types` (`Monitor_GetRegisteredTypes`) first — the set of
  monitor types is installation-specific.
- `POST /api/v1/monitors/-` (`Monitor_CreateMonitor`) to define one.
- `PUT /api/v1/devices/{deviceId}/monitors/{assignmentId}` (`Device_UpdateMonitorById`) to change an
  assignment.

## 4. Before you touch a live monitor

Put the device in maintenance rather than deleting monitoring:
`PUT /api/v1/devices/{deviceId}/config/maintenance` (`Device_UpdateMaintenanceById`), or on a
schedule with `Device_UpdateMaintenanceSchedule`.

## Rules that apply to every step

- **This contract declares no error responses at all.** Zero of 168 operations declare any 4xx or
  5xx. You cannot learn the failure shape from the spec — check the HTTP status yourself and treat
  any non-2xx body as untyped.
- **No idempotency key**, and the batch operations (`Device_UpdatePropertiesBatch`,
  `Device_UpdateMaintenanceBatch`, `Monitor_PatchMonitors`) fan out across many devices at once. A
  retried batch re-applies to every member. Re-read before retrying.
- **Deletes are final.** 21 delete operations over devices, credentials, interfaces, attributes and
  group memberships, with no restore path. The one restore-shaped operation,
  `DeviceRole_RestoreSystemDefaultsById`, restores a *role's shipped defaults* — it does not undo a
  delete.
- **No rate-limit signal**, no request-id header, no Sunset/Deprecation headers.
