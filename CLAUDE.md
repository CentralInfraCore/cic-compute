# cic-compute — Claude kontextus

## Mi ez a rendszer

A `cic-compute` a CentralInfraCore **compute domain schema repo** — a cic-primitives leszármazottja.
VirtualMachine, PhysicalMachine és CloudInstance helyett egyetlen egységes:
`schemas/domain/compute-resource.yaml` (D-010).

Nem futtat compute workloadokat. Hordozható domain- és adapter-sémákat definiál.
Implementációt a runtime repók (cic-relay és adapter implementációk) hordoznak.

Részletes architektúra: `ai/SYSTEM_CONTEXT.md`
Tervezési döntések: `ai/DECISIONS.md`
Kötelező szabályok: `ai/MAINTENANCE_CONTRACT.md`

---

## Boot sequence — minden session elején

1. `mcp__cic-graph__kb_status` — KB elérhető és friss?
2. `ai/DECISIONS.md` — D-010/D-011 ismerete kötelező mielőtt bármit mondasz
3. `ai/SYSTEM_CONTEXT.md` — teljes compute domain kontextus
4. `ai/MAINTENANCE_CONTRACT.md` — mit szabad, mit nem

Amíg ez nincs meg, ne tegyél tényállításokat a compute séma állapotáról.

---

## Háromszintű státusz — minden állításhoz kötelező

| Státusz | Jelentés |
|---|---|
| **defined** | YAML séma létezik, `make validate` zöld |
| **draft** | Design megvan, séma még nincs |
| **concept** | Megbeszélt, formálisan nem rögzítve |

---

## Aktuális séma állapot

| Elem | Státusz | Megjegyzés |
|---|---|---|
| `compute-resource.yaml` | **defined** | unified, platform-agnosztikus |
| `hypervisor-adapter.yaml` | **defined** | KVM/VMware/Proxmox contract |
| `ipmi-adapter.yaml` | **defined** | IPMI/Redfish contract |
| `cloud-provider-adapter.yaml` | **defined** | cloud provider contract (aws/gcp/azure/hcloud/do/ovh/oci) |
| `policy_surface` bekötés | **defined** | state=adapter-only, config=owner |
| Access atom integráció | **defined** | primitives v0.1.1-ből örökli |

---

## Kritikus döntések (röviden)

**D-010 — unified ComputeResource**
Nincs VM/Physical/Cloud szétválasztás. `instance_type` adapter-internal.
Address: `{backend}/{provider}/{location}/{id}`

**D-011 — power_state enum**
`running/starting/stopping/stopped/paused/terminated/error/unknown`
`paused` csak hypervisor_suspend capability adapterektől.

**Access atom (cic-primitives D-011)**
`state_surface` modify: adapter-only — `config_surface` modify: owner.

---

## Kompozíciós lánc

```
base-repo (upstream sablon)
    │  remote: base → git merge base@0.5.0
    └──► cic-primitives (primitives/@v0.1.1)
              │  remote: base → git merge primitives/@v0.1.1
              └──► cic-compute (ez a repo)
```

---

## Mérce

```bash
make validate          # ha nem zöld, semmi sem kész
make release VERSION=  # signed artifact (Vault szükséges)
```

Lezárási kritérium minden adapter contract-ra:
1. Az address mező egyezik a compute-resource.yaml address kulcsával?
2. Az observe output mezők mappolnak a state_surface-re?
3. Az operations capability listája szinkronban van a binding_surface-szel?
