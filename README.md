# obsidian-sync-updater

Public update repository for an Obsidian plugin. It hosts the plugin's
update manifest — a single small JSON file that the plugin fetches to
learn the state of the latest release rollout.

## Repository layout

| Branch | Contents |
| --- | --- |
| `main` | this README |
| `latest` | the current manifest, `update.json` |

The `latest` branch always carries exactly one commit: it is force-moved
on every release step, so the repository intentionally keeps no history —
only the current state matters. The manifest is consumed as raw content
from `latest` at the path `update.json`.

This repository contains no plugin code and no user data — only the
manifest.

## Manifest format

`update.json` is a single JSON object (UTF-8):

```json
{
  "manifestVersion": 1,
  "updateId": "<uuid>",
  "rollout": "full | staged | halted",
  "releasedAt": <unix epoch ms>,
  "bundles": [
    {
      "bundleId": "<uuid>",
      "pendingDeltas": [0, 1],
      "metadataMissing": false
    }
  ]
}
```

| Field | Type | Meaning |
| --- | --- | --- |
| `manifestVersion` | number | schema version; readers ignore manifests with an unknown version |
| `updateId` | string | identifies the current update |
| `releasedAt` | number | release timestamp, Unix epoch milliseconds |
| `rollout` | string | overall state of the rollout: `full` — everything delivered; `staged` — partially delivered, see `bundles`; `halted` — delivery stopped, see `bundles` |
| `bundles` | array | present only when action is required; omitted on a fully delivered rollout |

Each `bundles` entry describes one bundle that still needs work:

| Field | Type | Meaning |
| --- | --- | --- |
| `bundleId` | string | identifies the bundle |
| `pendingDeltas` | number[] | parts of the bundle still missing, ascending, no repeats |
| `metadataMissing` | boolean | `true` when the bundle layout is unknown and it must be fetched in full |

## Updates

Manifest updates are pushed by the release automation (write access is
limited to it). Reading is anonymous and requires no token.
