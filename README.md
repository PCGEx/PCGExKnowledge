# PCGExKnowledge

The PCGEx node knowledge base as raw, queryable JSON: one entry per documented node across [PCGEx (PCG Extended Toolkit)](https://github.com/Nebukam/PCGExtendedToolkit) and its sibling plugins, plus pointers into the concept documentation. It is rendered from the same verified card store that produces the [PCGEx GitBook](https://pcgex.gitbook.io/pcgex), so every purpose statement, pin contract, settings note and trap in here was authored against plugin source and ships with the hash of the source it describes.

The primary consumer is the [PCGExAssistant](https://github.com/PCGEx/PCGExAssistant) companion plugin, which vendors this repository as its `Knowledge/` submodule and serves it to AI agents inside the Unreal editor. The data is published here, separately, so you can build your own tooling on it: an MCP server, a search index, a cheat sheet generator, whatever you need. Nothing in this repository is hand-edited; corrections happen in the card store and arrive with the next render.

## Files

| File | Contents |
| --- | --- |
| `manifest.json` | Format version, generation date, and the per-plugin file list with node and concept counts. Iterate `plugins[].file` rather than hardcoding filenames. |
| `search.json` | The flat search surface, all plugins in one array. Rows are `kind: "entry"` or `kind: "concept"`. Load only this to answer "which node or concept fits". |
| `concepts.json` | Concept-page pointers: title, description, section outline, and the live documentation URL. Extracted from the books mechanically. |
| `<Plugin>.json` | Full entries for one plugin, keyed by id, with the plugin's alias map and enum table. Load lazily when a search hit needs detail. |
| `schema/` | JSON Schemas for each of the above. |

## The id scheme

An entry's id is its C++ class name, `UPCGExBevelPathSettings` for **Path : Bevel**. That choice is deliberate: the same name is what Unreal reflection reports for the installed plugin and what the engine's PCG tooling needs to place a node in a graph, so the id joins documentation, live editor state, and actuation without a mapping table.

Not every entry is a placeable PCG node, which is why the container says `entries`: the corpus also documents assets (collections), shared settings surfaces, factories and a handful of Blueprint nodes. The `classification` field carries the real taxonomy (`node`, `provider`, `instanced_factory`, `factory_data`, `shared_struct`, `asset`, `blueprint_node`), and only `node` and `provider` belong in a PCG graph. `blueprint_node` marks the collection accessor helpers that live in Blueprint graphs; never try to place one in a PCG graph.

**One resolution path.** To resolve any id: check the `entries` map of each plugin file, then the `aliases` map. Every id the bundle emits resolves that way, wherever it appears:

- `aliases` maps variant classes (several nodes documented as one family, such as the score-based edge refinements) and covered helper classes to their primary entry.
- `inherits` on an entry lists documented ancestor ids, nearest first. Settings inherited from a shared base are not repeated on every node; fetch the base's own entry instead.
- `see` on a setting points at the entry documenting a struct-typed settings block. A struct with no entry of its own carries its `fields` inline instead.

References may cross files: sibling plugins inherit from bases documented in `PCGExtendedToolkit.json`.

## Conventions

- **Prose is verbatim from the documentation cards**, light markdown included. A `**Bold Name**` inside any text field is a reference to another node by display name, the same convention the books use for cross-linking.
- **Defaults are raw C++ initializer text** (`EPCGExBevelMode::Radius`, `FVector(0, 0, -100)`). Enum defaults resolve through the plugin file's `enums` table, which carries display names and per-value tooltips.
- **`settings` entries merge two layers**: mechanical facts from the source index (type, tooltip, default, edit condition, PCG overridability) and authored semantics where the tooltip was not enough (`semantics`, `trap`, `interacts_with`). A setting with no `semantics` is one whose tooltip suffices.
- **`source`** on every entry names the header the card was authored against and its content hash at authoring time. `stale: true` means the source has changed since; the manifest counts these per plugin.
- **`url`** fields point at the corresponding page of the live documentation.

## Versioning

`bundle_schema` in the manifest versions the format, semver. Content freshness is the manifest's `generated` date plus each plugin's `version` (its `.uplugin` VersionName at render time). Consumers pinning a state should pin a commit; the PCGExAssistant submodule does exactly that, which is how its shipped knowledge matches its shipped plugin versions.

## Provenance

Rendered by `render-bundle.js` in the PCGEx documentation automation pipeline, from a card store of one authored, source-verified JSON card per documentable header and a mechanical index rebuilt from plugin source. The card contract, staleness rules and authoring process are documented in that repository.

## License

MIT. See [LICENSE](LICENSE).
