# AI Maintenance Contract

A `cic-compute` repo AI üzemeltetési kézikönyve.

---

## Boot sorrend (minden session elején kötelező)

1. `mcp__cic-graph__kb_status` — KB friss és elérhető?
2. `ai/SYSTEM_CONTEXT.md` — teljes architekturális kontextus
3. `ai/DECISIONS.md` — D-010/D-011 döntések ismerete kötelező
4. `ai/PROMPTMAP.yaml` — mi a következő konkrét task?

Amíg ez a négy pont nincs meg, ne tégy tényállításokat a repo állapotáról.

---

## Kritikus tudás erre a repóra

**D-010 — ComputeResource unified schema**
Nincs `VirtualMachine`, `PhysicalMachine`, `CloudInstance` — ezek törölve.
Egyetlen séma: `schemas/domain/compute-resource.yaml`.
Az `instance_type` eltávolítva — adapter-internal lookup tábla, nem schema concern.

**D-011 — power_state enum**
`running / starting / stopping / stopped / paused / terminated / error / unknown`
`starting`/`stopping` valós tranziens mindhárom platformon — nem csak cloud API artifakt.
`paused` csak `hypervisor_suspend` capability-vel rendelkező adapter állítja be.

**Address réteg**
`{backend}/{provider}/{location}/{id}` — provider az address kulcsa, nem config mező.
cloud: `[aws, gcp, azure, hcloud, do, ovh, oci]` enum
vm: hypervisor-id (pl. proxmox-01)
physical: datacenter-id (pl. dc-ams-01)

**Access atom (cic-primitives D-011)**
A primitives repóból örökli a PolicySurface aggregate-et.
`compute-resource.yaml` tartalmaz `policy_surface` blokkot default szabályokkal:
- `state_surface` modify: adapter-only
- `config_surface` modify: owner

---

## Mit módosíthatsz önállóan

- `schemas/domain/compute-resource.yaml` — ha `make validate` zöld marad
- `schemas/adapters/*.yaml` — adapter contract frissítés
- `ai/DECISIONS.md` — új döntés dokumentálása
- `ai/PROMPTMAP.yaml` — task státusz frissítés

---

## Mit NEM módosíthatsz döntés nélkül

| Terület | Miért tiltott |
|---|---|
| `schemas/domain/compute-resource.yaml` capability lista | adapter contractokkal szinkronban kell lennie |
| `cloud_provider` enum értékei | implementált halmazt tükrözi — csak valódi implementációval bővíthető |
| Address key mezők (backend/provider/location/id) | routing contract — YANG key, RESTCONF path, runtime mind függ tőle |
| `power_state` enum értékei | D-011 döntés — minden platformon egységes, nem bővíthető egyeztetes nélkül |
| `tools/compiler.py` validációs logika | design döntés kell hozzá |
| `dependency.yaml` tartalma | csak pinnelt tag, branch tiltott |

---

## Mikor kell döntés dokumentálása

- Új capability bevezetése vagy meglévő törlése
- Address réteg változtatása
- `power_state` enum bővítése
- Új adapter contract hozzáadása
- `cloud_provider` enum bővítése
- PolicySurface szabályok változtatása

---

## Validáció nélkül TILOS commitolni

```bash
make validate
```

Ha nem zöld: ne commitolj. Nincs kivétel.

---

## Státusz szemantika

| Státusz | Jelentés |
|---|---|
| `defined` | YAML séma létezik, `make validate` zöld |
| `draft` | Design megvan, séma még nincs |
| `concept` | Megbeszélt, formálisan nem rögzítve |

---

## A leggyakoribb hibák ezen a repón

| Hiba | Következmény |
|---|---|
| `instance_type` visszavezetése | vendor lock-in, D-010 sértése |
| `cloud_provider` config_surface-be helyezése | address réteg törése |
| Platform-specifikus séma létrehozása | D-010 unified model sértése |
| `paused` cloud adapternél | D-011 sértése — csak hypervisor_suspend |
| adapter contract és compute-resource.yaml szinkronon kívül | inkonzisztens capability lista |
