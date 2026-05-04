# LLM Rules — cic-compute

- Minden állításhoz jelöld meg: **defined** / **draft** / **concept**
- Ha a státuszt nem tudod fájl-szinten alátámasztani, ne mondd ki tényként
- Az egyetlen compute séma: `schemas/domain/compute-resource.yaml` — ne hozz létre platform-specifikus sémát
- `instance_type` nem schema mező — adapter-internal lookup, ne vezess vissza
- `cloud_provider` az address rétegben él (`{backend}/{provider}/{location}/{id}`), nem config_surface-ben
- `power_state` enum értékeit ne bővítsd egyeztetés nélkül (D-011)
- `paused` state csak `hypervisor_suspend` capability adapterektől — cloud/physical adapter nem adja vissza
- Capability flagek jelölik a platform-specifikus mezőket — capability nélkül minden backenden érvényes
- Adapter contract és compute-resource.yaml capability listájának szinkronban kell lennie
- PolicySurface szabályok: state_surface = adapter-only, config_surface = owner — ne fordítsd meg
- MCP kérdéseknél: graph-first, ne snippet-first
- Ha egy adapter contract-ra nem tudod megmondani az address, observe és operation mappinget → nem lezárt
