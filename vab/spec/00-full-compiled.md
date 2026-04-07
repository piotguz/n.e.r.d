# N.E.R.D. – Nominally Engineered Rocket Disaster
## Specification: KSP-like Vehicle Assembly Building (VAB) Editor

## 1. Purpose

Build a **rocket vehicle editor** inspired by **Kerbal Space Program 1 VAB/SPH editing principles**, while simplifying the content domain to a smaller, cleaner, more parametric system.

The editor must preserve the **core KSP1 user experience**:
- drag-and-drop assembly
- parent-child part hierarchy
- surface and stack attachment
- snapping
- symmetry placement
- part translation/rotation
- root-part based craft structure
- immediate visual feedback during assembly
- editor gizmos for move/rotate
- procedural part adjustment before and after placement

At the same time, the part catalog is intentionally reduced to a compact set of reusable, highly parametric parts:
- engines
- fuel tanks
- nose cones
- fins
- RCS thrusters / RCS blocks

This editor is intended as the primary tool for building rockets and spacecraft from modular parts defined in data files.

---

## 2. Product Goals

### 2.1 Main goal
Provide a vehicle assembly editor that feels familiar to KSP1 users, but is easier to implement, easier to extend, and strongly driven by parametric part definitions.

### 2.2 Design goals
- Preserve KSP1 editing logic, interaction patterns, and mental model.
- Reduce content complexity by limiting available part families.
- Support procedural dimensions and appearance.
- Ensure stable and intuitive snapping and symmetry.
- Make part configuration data-driven and mod-friendly.
- Keep the editor responsive even with large craft.
- Separate visual representation from physical/gameplay properties.

### 2.3 Non-goals
- No complex internal crew systems.
- No robotics/mechanical hinges in first version.
- No procedural fairings in first version unless modeled as special nose-cone subtype.
- No staging logic editor in this scope.
- No resource transfer UI in this scope.
- No wings with arbitrary airfoil editing in first version.

---

## 3. High-Level Scope

The editor must allow the player to:
1. create a new craft from a root part
2. browse and select parts from categorized part palette
3. drag parts into the assembly scene
4. attach parts using stack nodes or surface attach rules
5. move, rotate, and reconfigure placed parts
6. apply symmetry during placement and optionally during editing
7. modify parametric values such as:
   - diameter
   - length
   - texture set
   - normal/bump map
   - color / paint scheme
   - engine variant parameters (where allowed)
8. save and load craft definitions
9. validate assembly constraints and provide feedback
10. preview mass/cost/basic metrics live during editing
11. export the complete craft into a standalone package consumable by external applications, especially the future flight module, including craft manifest YAML and any required custom assets
12. manage each ship project as a local Git repository with branch-based iteration workflows directly supported by the editor

---

## 4. Editor Principles to Preserve from KSP1

The editor should keep the following KSP1 principles as first-class requirements.

### 4.1 Rooted hierarchical assembly
- Every craft has exactly one **root part**.
- All other parts are attached directly or indirectly to the root.
- The craft is represented as a tree for structural attachment purposes.
- A detached subtree can be picked up and reattached elsewhere.

### 4.2 Attachment-first building flow
- Parts are added by attaching them to valid attachment locations.
- New parts preview live before placement.
- The editor highlights valid targets and snap points.
- Invalid placements are clearly rejected.

### 4.3 Stack and surface attachment
The editor must support:
- **stack attachment**: node-to-node axial attachment
- **surface attachment**: attaching a part to the outer surface of another part

Examples:
- tank on top of engine via stack node
- fin on side of tank via surface attach
- RCS block on side of tank or nose section via surface attach

### 4.4 Gizmo-based editing
As in KSP1, placed parts or selected subassemblies must support:
- move gizmo
- rotate gizmo
- optional re-root action
- delete / detach action

Technical requirement:
- gizmos must allow post-snap editing of already placed parts
- translation must support constrained movement along relevant axes
- rotation must support angular snapping and local/host-relative modes
- gizmo behavior must remain consistent across all part families

### 4.5 Symmetry-assisted building
- Symmetry modes must be available during placement.
- Common counts: 1x, 2x, 3x, 4x, 6x, 8x.
- Radial symmetry around a chosen craft axis must be supported.
- Mirrored fin placement should be supported where appropriate.
- Symmetry counterparts must remain linked unless user explicitly breaks symmetry.

Technical requirement:
- symmetry placement should be implemented as a script/system that mirrors a part's placement around the parent part's center axis
- symmetry edits must propagate to all linked counterparts for placement, movement, rotation, and supported parameter changes

### 4.6 Snap-centric control
- Snapping should be default behavior.
- Fine placement may exist, but snap should dominate for usability.
- Rotational snapping must be configurable.
- Surface attach should support angular snap around cylindrical bodies.

### 4.7 Immediate visual feedback
- Show ghost part preview before placement.
- Show attach node alignment indicators.
- Show collision / invalid placement feedback.
- Show symmetry preview copies before confirming placement.

### 4.8 Procedural part tweaking without breaking editor clarity
Unlike KSP1’s mostly discrete parts, this editor uses procedural dimensions. However, the procedural system must not make the editor feel like CAD software. It should remain game-oriented and simple.

### 4.9 Unified part interaction model (KSP1 parity)
A critical requirement is that **all parts—regardless of type—must be interacted with in a consistent, KSP1-like way**.

This means:
- Every part supports the same core interaction verbs:
  - select
  - grab (detach or pick up with subtree)
  - attach (stack or surface)
  - move (via gizmo or drag when applicable)
  - rotate (via gizmo or placement rotation)
  - reconfigure (via inspector)
  - delete
- No part introduces a unique or special interaction model that breaks consistency.
- Differences between parts must come only from:
  - attachment rules (e.g., stack-only vs surface)
  - available parameters
  - constraints
  - visual/functional behavior

#### Key principle
The editor interaction model must be **type-agnostic**:
- engines, tanks, fins, RCS, and nose cones all behave identically from a user interaction perspective
- the user does not need to learn new controls for different part families

#### Implications for implementation
- All parts inherit from a common interaction interface/component
- Editor tools (move, rotate, symmetry, snapping) operate on generic part instances
- Surface vs stack differences are handled by attachment system, not UI differences
- Inspector UI is dynamically generated from parameter schema, not hardcoded per part

#### Anti-patterns (must be avoided)
- Special-case controls for specific part types (e.g., fins edited differently than other parts)
- Unique drag behaviors per part category
- Hidden interaction modes that only apply to certain parts

This ensures the editor retains the **intuitive, muscle-memory-driven workflow of KSP1**, which is one of its strongest design qualities.

---

## 5. Simplified Part Catalog

Only the following part families are in scope.

### 5.1 Engines
#### Functional role
Provide thrust.

#### Editor requirements
- stack attach by default
- optional surface attach only if explicitly allowed by subtype
- configurable size compatibility with tank diameters
- configurable nozzle visual length and bell variant

#### Parametric properties
- diameter class or exact attach diameter
- engine length / model scale within allowed bounds
- thrust class
- vacuum/sea-level performance parameters
- gimbal enabled/disabled
- texture set
- base color / accent color
- normal map / bump map selection

#### Subtypes (optional)
- sea-level engine
- vacuum engine
- compact lander engine
- multi-nozzle cluster engine

### 5.2 Fuel Tanks
#### Functional role
Main structural and propellant storage parts.

#### Editor requirements
- stack attach top and bottom
- optional radial/surface attach if enabled for small utility tanks
- serve as common hosts for fins, RCS, and other surface-attached parts

#### Parametric properties
- diameter
- length
- end-cap style
- wall/segment style
- texture set
- color regions (body, stripe, top, bottom)
- normal/bump map
- optional material style (painted, bare metal, insulated)

#### Constraints
- dimensions must remain within subtype min/max values
- mass/resource volume scales with dimensions
- stack nodes update automatically with size changes

### 5.3 Nose Cones
#### Functional role
Aerodynamic front-end part.

#### Editor requirements
- stack attach to top node of compatible part
- must visually blend with compatible tank diameters

#### Parametric properties
- base diameter
- length
- profile style (conic, ogive, rounded, sharp)
- texture set
- color
- normal/bump map

### 5.4 Fins
#### Functional role
Stability and control surfaces.

#### Editor requirements
- surface attach by default
- radial or mirrored symmetry common
- must align correctly to local surface normal and craft axis

#### Parametric properties
- root chord
- tip chord
- span
- sweep
- thickness class
- cant angle
- texture set
- color
- normal/bump map

#### Constraints
- must not intersect host body excessively
- must support snap positions around cylindrical surfaces

### 5.5 RCS Thrusters / RCS Blocks
#### Functional role
Translation and attitude control.

#### Editor requirements
- surface attach by default
- placement orientation matters
- must support radial symmetry
- should preview thrust direction arrows in editor

#### Parametric properties
- block type (1-way, 2-way, 4-way, 5-way)
- size class
- offset from surface within safe range
- texture set
- color
- normal/bump map

---

## 6. Parametric Part System

## 6.1 Philosophy
Parts are not fully freeform. They are **bounded procedural assets**.
Each part definition provides:
- allowed dimensions
- valid attachment nodes
- generated or scaled meshes
- visual material options
- physical scaling rules
- editor rules

This gives flexibility without allowing uncontrolled shape editing.

## 6.2 Common parametric attributes
The system should support a shared schema for parameters:
- `diameter`
- `length`
- `variant`
- `textureSet`
- `normalMap`
- `colorPrimary`
- `colorSecondary`
- `colorTertiary`
- `surfaceFinish`
- `massOverride` (debug/dev only, not standard player option)

## 6.3 Constraints model
Each procedural parameter must support:
- minimum value
- maximum value
- increment/step for snapping
- optional preset values
- optional dependency rules

Example:
- tank diameter may be restricted to 0.625, 1.25, 2.5, 3.75, 5.0m presets
- or may support continuous adjustment in 0.125m steps

## 6.4 Parameter change propagation
When a parameter changes after placement:
- visual mesh updates
- attachment nodes update
- children remain attached if still valid
- if a child becomes invalid, the editor must either:
  - prevent the change, or
  - detach the child with clear warning

The preferred behavior is **prevent invalid parametric edits by default**.

---

## 7. Editor Interaction Model

## 7.1 Main editor zones
The screen should contain:
- **assembly viewport**
- **part palette / category browser**
- **part inspector / properties panel**
- **craft metrics panel**
- **symmetry and snap controls**
- **top toolbar** for save/load/undo/redo/filter/search

## 7.2 Camera behavior
The camera should support:
- orbit around craft
- pan
- zoom
- focus selected part
- frame entire craft
- optional lock to craft origin

Camera behavior should feel close to KSP1 VAB, not like a CAD free camera.

## 7.3 Part selection
User can select:
- a single part
- a symmetry group
- a subtree/subassembly

Selection should highlight:
- selected part outline
- parent/child relation if needed
- attachment nodes

## 7.4 Drag from palette to placement preview
Flow:
1. user selects part from palette
2. ghost preview follows cursor
3. valid target nodes/surfaces highlight
4. symmetry copies preview
5. placement confirms on click
6. part becomes selected for further editing

## 7.5 Move tool
Allows translation of selected part or subtree.

Requirements:
- constrained axis movement
- snap-based movement where applicable
- preserve valid attachment logic
- for attached parts, movement may switch attachment target/node
- for surface-attached parts, movement across host surface must stay stable and intuitive

## 7.6 Rotate tool
Allows rotation of selected part.

Requirements:
- rotational snap increments
- local-axis and host-relative modes
- surface-attached part rotation around normal/craft axis
- symmetry-aware rotation updates

## 7.6.1 Gizmo requirements
The editor must provide explicit player tools for:
- Translate / Move
- Rotate

These gizmos must work after snapping/placement, not only during initial attachment preview.
They must be available for:
- individual parts
- symmetry groups where applicable
- selected subassemblies/subtrees

## 7.7 Reconfigure tool / inspector
Selecting a part opens editable parameters.
Changes should update live.
Examples:
- tank length slider/input
- tank diameter preset buttons
- texture dropdown
- color pickers
- fin geometry sliders
- engine variant selection

---

## 8. Attachment System

## 8.1 Attachment types
The system must support:
- `stack`
- `surface`

Optional future types:
- `internal`
- `coupled`
- `compound`

## 8.2 Stack nodes
Each stack-capable part defines one or more nodes.
Each node has:
- local position
- local orientation
- size/diameter compatibility
- gender/type if used
- allowed counterpart categories
- occupancy state

Required examples:
- tank top node
- tank bottom node
- engine top node
- nose cone bottom node

## 8.3 Surface attachment
Surface attach requires:
- raycast hit on valid attachable mesh region
- local anchor point
- local normal
- host-relative transform storage
- optional snap to angular sectors or guide lines

## 8.4 Attachment rules
Examples:
- nose cone attaches only to compatible top stack node
- engine attaches only to compatible bottom stack node unless subtype allows otherwise
- fins attach only to surface-attach-enabled hosts
- RCS attaches to surface-attach-enabled hosts

## 8.5 Compatibility rules
Compatibility may use:
- exact diameter match
- tolerance range
- adapter allowance if part supports procedural resizing

## 8.6 Tree updates
When a part is attached:
- it becomes child of target part
- any selected subtree moves as one unit
- craft hierarchy updates immediately

When detached:
- detached subtree becomes transient preview object until reattached or deleted

---

## 9. Snapping System

## 9.1 Core principle
Snapping should make building easy, fast, and predictable.

## 9.2 Stack snap
When a stack node approaches another compatible node:
- nodes magnetically align
- axial orientation auto-corrects
- ghost part snaps into final preview position

## 9.3 Surface snap
Surface attach should support:
- normal alignment to host surface
- angular snap around cylindrical bodies
- longitudinal snap to reference bands/markers if enabled
- mirrored/radial symmetry preview

## 9.4 Rotation snap
Support rotation steps such as:
- 1°
- 5°
- 15°
- 45°
- 90°

Default should favor game-friendly increments.

## 9.5 Parametric snap
Dimensional parameters should also snap.
Examples:
- tank length in 0.25m increments
- diameter in standard size classes or configured steps
- fin dimensions in bounded increments

---

## 10. Symmetry System

## 10.1 Supported modes
- off / single placement
- radial symmetry around central axis
- mirrored bilateral symmetry where meaningful

## 10.2 Symmetry behavior
- placing one part creates linked counterparts
- editing one counterpart propagates to all linked instances
- deleting one deletes entire group unless symmetry is broken
- moving/rotating one updates all counterparts

## 10.3 Symmetry axis rules
The editor should use a clear craft reference axis, typically longitudinal vehicle axis.
For cylindrical main bodies, radial placements are distributed evenly around this axis.

## 10.4 Supported use cases
- 4 fins around lower tank
- 4 or 8 RCS blocks around upper stage
- 2 mirrored side thrusters if design supports it

## 10.5 Breaking symmetry
User may explicitly break symmetry for a selected group.
After breaking:
- parts become independent
- properties no longer auto-sync

## 10.6 Symmetry implementation rule
The symmetry tool should mirror part placement around the parent part's center axis.

At minimum, supported counts must include:
- 2x
- 3x
- 4x
- 8x

The symmetry system must preserve:
- relative radial spacing
- orientation consistency
- attachment validity
- linked editing behavior until symmetry is explicitly broken

---

## 11. Texture, Materials, and Appearance System

## 11.1 Visual customization goals
The editor must allow parts to look visually distinct without introducing an unmanageable asset explosion.

## 11.2 Texture support
Each part or part family must support:
- albedo/base color texture
- normal map / bump texture
- optional metallic/roughness maps
- optional mask texture for paint regions

## 11.3 Texture sets
A part may expose multiple texture sets, for example:
- white painted
- black thermal
- bare metal
- orange tank insulation
- military/industrial gray

## 11.4 Color customization
Support at least:
- primary color
- secondary color
- accent/stripe color

Color changes should be applied through material parameters or masks, not necessarily unique textures.

## 11.5 Procedural texture scaling
When a tank or nose cone changes length/diameter:
- textures must scale or tile correctly
- seams must remain visually acceptable
- panel lines and normal maps should remain coherent

## 11.6 Bump/normal support
The specification must support normal/bump maps as first-class visual assets for:
- panel lines
- rivets
- insulation patterns
- structural ridges

## 11.7 Material consistency rules
When two stack-connected parts share compatible diameters and texture family, the visuals should blend reasonably well.

## 11.8 Custom user textures
The editor must support **custom per-craft textures**, especially for procedural fuel tanks, but also for any part type whose material definition allows external overrides.

This is a first-class modding and personalization feature, not just an editor-side convenience.

### Supported custom asset types
At minimum, the system should support user-supplied:
- albedo/base color textures
- normal maps / bump textures
- optional mask textures
- optional metallic/roughness textures if the renderer uses them

### Primary use cases
- custom painted tanks
- mission- or faction-specific liveries
- stripes, decals, warning markings, logos
- custom insulation patterns
- custom fin or nose-cone skins

### Rules for custom textures
- custom textures are assigned per craft instance, not by mutating the global base part definition
- base game/default textures remain available as fallbacks
- tank parts should have the richest support, because they are the most visible and most likely to be customized
- custom textures must remain compatible with procedural resizing rules
- the editor must validate whether a custom texture is suitable for the selected material slot and UV behavior

### Custom texture application model
A part instance may reference either:
- a built-in texture set from the part library, or
- a craft-local custom texture asset package

The material layer should therefore support a distinction between:
- `builtInTextureSet`
- `customTextureRefs`

### Tank-specific requirement
Because tanks are the dominant structural and visual element in most rockets, fuel tanks must support:
- custom cylindrical body textures
- optional separate cap/end textures
- custom normal maps
- repeat/tile mode or stretch mode depending on material definition
- preview of texture distortion when changing tank length/diameter

### Validation and fallback
If a custom texture is missing, invalid, or incompatible:
- the editor must warn the user
- export must either fail with a clear message or automatically fall back to a valid built-in texture, depending on export mode
- the flight module must also support safe fallback behavior

## 11.9 Craft-local asset ownership
Custom textures used by a craft must be treated as part of the craft package, not as implicit external dependencies.

That means:
- a craft using custom textures is not fully portable unless those textures are exported together with the craft
- portability requires packaging all non-standard referenced assets together with the craft manifest

This requirement directly affects serialization and package export design.

## 12. Part Data Model

## 12.1 Data-driven requirement
All parts must be defined through external data files, not hard-coded.

Each part definition should include:
- identifier
- family/category
- display name
- description
- procedural parameters
- attachment rules
- mesh generation/scaling rules
- physical parameters
- material/texture options
- editor metadata

## 12.2 Conceptual schema

```yaml
id: tank_basic_125
family: fuel_tank
displayName: Basic Fuel Tank
rootAttach: true
attachModes:
  - stack
  - surface
parameters:
  diameter:
    type: preset_float
    values: [1.25, 2.5, 3.75]
  length:
    type: stepped_float
    min: 0.5
    max: 8.0
    step: 0.25
  textureSet:
    type: enum
    values: [white, orange, bare_metal]
  colorPrimary:
    type: color
  colorSecondary:
    type: color
nodes:
  - id: top
    type: stack
    direction: up
    diameterFrom: parameter:diameter
  - id: bottom
    type: stack
    direction: down
    diameterFrom: parameter:diameter
surfaceAttach:
  enabled: true
mesh:
  generator: procedural_cylinder_tank
materials:
  supportsNormalMap: true
  supportsMasking: true
physics:
  dryMassScale: volume_based
  resourceVolumeScale: volume_based
editor:
  symmetryAllowed: true
  canBeRoot: true
```

## 12.3 Craft instance data
Craft save data must store:
- unique part instance id
- part definition id
- parent instance id
- attachment mode
- attachment transform / node binding
- parameter values
- symmetry group membership
- editor-only metadata if needed

---

## 12.4 Ship project repository model
Each ship project should be treated as a **self-contained local project**, not just as a single craft file.

This is a strategic requirement because the game is intended to support:
- YAML-based craft definitions
- custom textures and assets
- standalone flight-module interoperability
- iterative engineering workflows
- branching design experiments

Therefore, the editor should model each ship as a **local Git repository-backed project**.

### Project-level requirements
Each ship project must:
- exist as a directory on disk
- contain the craft manifest YAML
- contain any craft-local assets such as textures and metadata
- be initialized as a local Git repository
- remain usable without any cloud or remote dependency

### Why Git is the right model
Git is a particularly strong fit here because:
- YAML and asset manifests are file-based
- ship design is iterative and experimental
- users may want to try multiple competing designs
- branches naturally represent design variants
- local repositories allow offline use
- advanced users can integrate the same project into external tooling pipelines

### Minimum Git-aware editor capabilities
The editor should support at least the following repository-aware actions:
- initialize a new ship project as a local Git repository
- open an existing ship project repository
- detect repository status (clean/dirty)
- commit current changes with a message
- create a new branch
- switch between branches
- show current branch name in the UI
- warn about uncommitted changes before destructive operations or branch switching

### Branching use cases
Branch support should be treated as a first-class design workflow, for example:
- `main` = baseline rocket design
- `heavy-lift-variant` = enlarged tank stack and engine cluster experiment
- `reentry-test` = modified nose cone and fin layout
- `paint-scheme-b` = alternative textures and markings

This fits the spirit of the game extremely well: branches become engineering experiments.

### Scope boundary
The editor does not need to become a full Git client in v1.
It should support a focused subset only:
- init
- status
- commit
- branch create
- branch switch
- optional diff awareness for manifest/assets

Advanced operations such as merge conflict resolution, rebase, stash, remote sync, and pull requests are explicitly out of scope for v1.

### Design principle
Git support must remain **project-centric**, not intrusive.
Casual users should still be able to use the editor without deep Git knowledge, while advanced users gain strong versioning and experimentation support.

## 13. Craft File Requirements

### 13.1 Strategic serialization requirement
The craft format is not just an editor save format. It is a **shared contract format** between the VAB editor and the future standalone flight/simulation module.

This means:
- every ship built in the editor must be exportable to YAML
- ships using custom textures or custom assets must be exportable as a portable package
- the future flight module must be able to run as a separate standalone application
- the standalone flight module must recognize and load the same YAML craft format without depending on editor-specific runtime state
- the YAML format must therefore be treated as a stable external interface, not an internal implementation detail

### 13.2 Save/load/export support
The editor must support:
- new craft
- save craft
- load craft
- duplicate craft / save as
- version-tolerant loading when part definitions evolve
- export craft to standalone YAML format
- import craft from standalone YAML format
- export craft as a ZIP archive containing manifest YAML plus all required custom assets
- import craft from such a ZIP archive
- save the craft and related assets as part of a local project directory suitable for Git tracking

### 13.3 Repository-friendly file strategy
Because each ship project is expected to be a local Git repository, the file layout must be Git-friendly.

That means:
- text-based canonical manifest files where possible
- deterministic serialization order
- stable file naming conventions
- avoidance of unnecessary file churn
- asset references that remain relative and portable
- optional separation of large binary assets into predictable folders

This is not just a tooling convenience. It directly improves branch usability, diff readability, and long-term maintainability.

### 13.4 Two-level interchange model
The recommended approach is to distinguish between:
- **manifest format** = YAML craft definition
- **portable distribution format** = ZIP archive containing manifest + referenced custom assets

This means:
- YAML is the canonical semantic craft description
- ZIP is the portable packaging/container format when external files are required

This is the correct approach because YAML alone is ideal for readability and Git workflows, while ZIP is necessary for reliable portability of custom textures and other assets.

### 13.5 YAML design goals
The YAML craft format must be:
- human-readable
- diff-friendly for Git/version control
- deterministic in structure
- stable across editor and flight module boundaries
- explicit rather than implicit
- independent from engine/editor-only object references

### 13.6 Package design goals
The ZIP package format must be:
- self-contained
- deterministic in directory layout
- easy to validate before import
- robust against missing references
- suitable for mod sharing and standalone simulation loading

### 13.7 Craft structure
A craft YAML file should contain:
- metadata
- schema version
- craft name
- root part id
- ordered part instance list
- hierarchy relations
- attachment definitions
- per-instance parameters
- appearance settings
- symmetry definitions
- references to custom assets where applicable
- optional derived data cache if needed

### 13.8 Required separation of runtime concerns
The YAML must contain only portable craft-definition data.
It must not depend on editor-only concepts such as:
- viewport state
- temporary gizmo state
- selection state
- transient drag preview state
- engine-specific scene instance ids
- pointers/references that are only valid in memory

A standalone flight module must be able to reconstruct the vehicle only from:
- YAML craft file
- part definition library
- packaged custom assets if referenced
- shared schema rules

### 13.9 ZIP package layout
A recommended portable craft archive layout is:

```text
my_craft.nerdship.zip
├── manifest.yaml
├── assets/
│   ├── textures/
│   │   ├── tank_body_custom_albedo.png
│   │   ├── tank_body_custom_normal.png
│   │   └── fin_markings.png
│   ├── materials/
│   │   └── optional_material_overrides.yaml
│   └── metadata/
│       └── package_info.yaml
```

### 13.10 Asset reference rules
The manifest YAML should reference packaged assets using package-relative paths, for example:
- `assets/textures/tank_body_custom_albedo.png`
- `assets/textures/tank_body_custom_normal.png`

It must not rely on:
- absolute machine-local paths
- editor cache folders
- user profile directories
- temporary import paths

This is critical. Otherwise exported ships will not be portable.

### 13.11 Robustness
Loading should handle:
- missing parts
- deprecated part definitions
- invalid parameters due to changed constraints
- older schema versions
- missing or corrupted asset files inside the ZIP package

Preferred behavior:
- load with warnings and auto-fallback if possible

### 13.12 YAML schema evolution
Because the YAML is intended as a long-term shared contract, it must support explicit versioning.

Required:
- top-level schema version field
- backwards-compatibility policy where feasible
- migration layer for older craft files
- validation before import into editor or flight module

### 13.13 Cross-application contract requirement
Both the editor and the flight module must use the same semantic interpretation of:
- part ids
- attachment types
- parent-child hierarchy
- transforms/orientation
- parameter names and units
- texture/material references
- symmetry declarations
- physical property derivation rules where applicable

If this is not enforced, the editor and flight module will drift apart. That drift must be treated as a design failure.

### 13.14 Conceptual YAML example
```yaml
schemaVersion: 1
craftName: Heavy Launcher Mk1
rootPartId: p001
metadata:
  author: Piotr
  createdAt: 2026-04-07T10:00:00Z
parts:
  - instanceId: p001
    partDefinitionId: tank_basic
    parentInstanceId: null
    attachment:
      mode: root
    parameters:
      diameter: 2.5
      length: 6.0
      builtInTextureSet: null
      customTextureRefs:
        albedo: assets/textures/tank_body_custom_albedo.png
        normal: assets/textures/tank_body_custom_normal.png
      colorPrimary: "#E5E5E5"
      colorSecondary: "#FF6A00"
    symmetry:
      groupId: null
  - instanceId: p002
    partDefinitionId: engine_lv_main
    parentInstanceId: p001
    attachment:
      mode: stack
      parentNodeId: bottom
      childNodeId: top
    parameters:
      diameter: 2.5
      variant: sea_level
      builtInTextureSet: black_thermal
      customTextureRefs: {}
      colorPrimary: "#444444"
    symmetry:
      groupId: null
  - instanceId: p003
    partDefinitionId: fin_basic
    parentInstanceId: p001
    attachment:
      mode: surface
      hostAnchor:
        u: 0.25
        v: 0.80
      localRotation:
        pitch: 0
        yaw: 0
        roll: 0
    parameters:
      rootChord: 1.2
      tipChord: 0.6
      span: 0.8
      sweep: 20
      builtInTextureSet: null
      customTextureRefs:
        albedo: assets/textures/fin_markings.png
      colorPrimary: "#D9D9D9"
    symmetry:
      groupId: sym_fin_4
symmetryGroups:
  - id: sym_fin_4
    mode: radial
    count: 4
    axis: longitudinal
```

### 13.15 Packaging/export modes
The editor should support at least two export modes:
- **manifest-only export**: YAML only, valid when all referenced assets are built-in or already available in shared content libraries
- **portable package export**: ZIP containing manifest and all required custom assets

Portable package export should be the default for user-created ships with custom textures.

### 13.16 Design conclusion
This packaging requirement is strategically important and should be elevated to a first-class architecture rule:
- the editor is one application
- the flight module is another application
- YAML is the semantic contract between them
- ZIP is the portable container for craft + custom assets

That is the correct direction, because it lets you evolve the VAB editor and the flight simulation separately while preserving interoperability and mod portability.

## 14. Validation Rules

## 14.1 Placement validation
Prevent invalid placements such as:
- node mismatch
- overlapping forbidden collision volume
- unsupported surface attach target
- impossible symmetry placement

## 14.2 Parametric validation
Prevent invalid values such as:
- length outside allowed range
- diameter incompatible with attached children/parent
- fin geometry outside subtype constraints

## 14.3 Structural validation
Editor should provide non-blocking warnings for:
- unattached floating parts
- asymmetrical RCS placement when symmetry expected
- large intersections
- invalid thrust direction arrangement

## 14.4 User feedback
Validation errors must be clear and concrete.
Examples:
- “Engine diameter does not match selected tank node.”
- “Tank length change would invalidate 4 attached side components.”
- “This host surface does not allow fin attachment.”

---

## 15. Undo/Redo and Editing Safety

The editor must support:
- multi-step undo/redo
- safe deletion with confirmation for large subtrees if configured
- reversible symmetry operations
- reversible parameter changes
- reversible detach/reattach operations

This is essential because parametric editing increases risk of accidental disruptive changes.

---

## 15.1 Repository UX requirements
The editor should expose lightweight Git/project controls without overwhelming the main building workflow.

Minimum UX expectations:
- current project path visible
- current branch visible
- unsaved changes indicator distinct from uncommitted Git changes indicator
- simple commit dialog with message field
- branch creation dialog
- branch switch flow with safeguards if working tree is dirty

Recommended UX behavior:
- separate “Save” from “Commit” clearly
- allow users to save local work without forcing a commit
- provide human-readable explanations such as “You have uncommitted design changes on this branch”

The editor should not assume the user is a Git expert, but it should expose enough structure that branch-based experimentation feels natural.

## 16. UX Requirements

## 16.1 Responsiveness
- placement preview must feel immediate
- parameter changes should update in near real time
- hierarchy and symmetry updates should not cause visible lag for typical craft sizes

## 16.2 Discoverability
- KSP-like users should understand the editor without tutorial overload
- new procedural features must remain understandable through constrained widgets and previews

## 16.3 Minimal friction
- common builds should take very few clicks
- placing stacks of tanks and engines should be extremely fast
- symmetry and snapping should reduce micro-adjustments

## 16.4 Search and filtering
Part palette should support:
- category tabs
- text search
- favorites/recent parts
- optional filtering by diameter/family

---

## 17. Metrics and Live Readouts

The editor should show at least simplified live metrics:
- total mass
- dry mass
- resource mass
- total cost
- total engine count
- approximate thrust
- approximate thrust-to-weight ratio
- center of mass marker
- center of thrust marker
- center of drag marker (optional in first version)

### 17.1 Center of Mass (CoM)
The editor must calculate CoM in real time based on all placed parts and their current mass state.

Conceptually:
- CoM = sum(mass_i * position_i) / sum(mass_i)

Requirements:
- recalculate when parts are added, removed, resized, moved, or when parameters affecting mass change
- render CoM as a clearly visible glowing icon/marker in the assembly viewport
- optionally show numeric coordinates in debug/advanced mode

### 17.2 Center of Thrust (CoT)
The editor must calculate Center of Thrust as the effective resultant of all active engines.

Requirements:
- derive CoT from the vector sum of all active engine thrust sources
- render CoT as a visible marker/vector in the assembly viewport
- show alignment relationship between CoT and CoM
- warn the user when CoT is significantly misaligned with CoM, because this indicates likely instability and control problems

### 17.2.1 Instantaneous total thrust vector
In addition to CoT, the editor must compute and visualize the **instantaneous total thrust vector**.

Requirements:
- compute the vector sum of all engine thrust forces in their current orientation
- render as a directional arrow originating from CoT or representative origin
- update in real time when engines are added, removed, rotated, or reconfigured
- support preview of gimbal influence if enabled

This vector represents the actual direction of acceleration applied to the craft.

### 17.2.2 Effective control authority / torque tendency
The editor should provide an estimation of **effective control authority and torque tendency**.

Requirements:
- compute torque contributions from engines (offset thrust), gimbal, and RCS where applicable
- indicate rotational tendencies caused by misalignment between thrust vector and CoM
- visualize direction and magnitude qualitatively (e.g., arrows, indicators, or heatmap-style cues)
- optionally classify stability level (stable / marginal / unstable)

Design goal:
- give the player intuitive feedback about whether the rocket will spin, drift, or remain stable under thrust
- avoid requiring full physics simulation while still providing meaningful guidance

This extends CoT analysis into practical flight behavior prediction.

### 17.3 Design-feedback rule
Misalignment between CoT and CoM should be treated as a major design signal.
The UI should make this understandable at a glance, because this is one of the core failure modes that turns a nominal design into a disaster.

This preserves a key KSP1-style design feedback loop.

---

## 18. Technical Architecture Considerations

### 18.0 Engine requirement
N.E.R.D. should be built using **Unreal Engine 5 (UE5)** as the primary game engine.

Rationale:
- strong support for real-time 3D rendering and large scenes
- powerful material system for procedural textures and customization
- built-in tools for editor-like interactions (gizmos, viewport controls)
- Blueprint + C++ hybrid architecture suitable for rapid iteration and performance-critical systems
- ecosystem support for physics, UI, and asset pipelines

Implications:
- editor interaction systems should leverage UE5 editor-style paradigms where possible
- rendering systems (including blueprint views and cross-sections) should integrate with UE5 rendering pipeline
- asset system should align with UE5 asset management while still supporting external YAML/ZIP workflows

## 18.1 Recommended subsystem split

## 18.1 Recommended subsystem split
- **Part Definition System**
- **Procedural Mesh/Variant System**
- **Attachment & Hierarchy System**
- **Symmetry System**
- **Editor Interaction System**
- **Material/Texture Customization System**
- **Craft Serialization System**
- **Project/Repository Management System**
- **Validation System**
- **Metrics Calculation System**

## 18.2 Separation of concerns
Strongly separate:
- part authoring definitions
- instantiated craft data
- editor interaction state
- runtime flight physics representation
- serialization contract shared across applications

The editor should produce craft data that can later be consumed by the simulation runtime.
The preferred architecture is:
- **Part library definitions** in external data files
- **Craft definitions** exported/imported as YAML
- **Ship projects** stored as local Git-friendly directory structures
- **Flight module** reading the same YAML as a standalone app

This prevents hard coupling between the editor and the simulation layer.

## 18.3 Deterministic reconstruction
Given the same part definitions and craft file, the editor should reconstruct the exact craft consistently.

---

## 19. Recommended Simplifications vs Full KSP1

To keep development realistic, preserve the editor principles but simplify implementation in the following ways.

### 19.1 Simplified content scope
Limit parts to 5 families only.

### 19.2 Controlled procedurality
Use bounded procedural parameters rather than arbitrary mesh editing.

### 19.3 Simplified attachment classes
Only stack + surface in v1.

### 19.4 Simplified aerodynamics editor-side
Provide design markers and basic validation, not full CFD-like preview.

### 19.5 Simplified engine variants
Let engine family vary via a few presets rather than fully procedural nozzle design.

This is the right compromise: **KSP1 editing feel, smaller implementation footprint**.

---

## 20. Acceptance Criteria

The editor is acceptable when the following are true:

1. A user can build a multi-stage rocket using only drag-and-drop and snapping.
2. A tank can be resized in length and diameter within defined constraints.
3. An engine can snap to tank bottom nodes and stay valid when resized within compatible limits.
4. A nose cone can attach to the top of a tank and adapt to size.
5. Fins can be surface-attached with radial symmetry.
6. RCS blocks can be surface-attached with visible orientation preview.
7. Built-in texture sets, colors, and normal maps can be changed per part instance.
8. Custom textures can be assigned to supported parts, especially tanks.
9. Move and rotate gizmos work on selected parts/subtrees after snapping.
10. The symmetry tool supports at least 2x, 3x, 4x, and 8x radial placement around the parent axis.
11. Undo/redo supports all common editor actions.
12. Craft can be saved and loaded without losing hierarchy, symmetry, or parametric values.
13. Each ship project can be created and stored as a local Git repository.
14. The editor can display current branch and repository status.
15. The editor can create branches and switch branches safely.
16. Craft can be exported to YAML and re-imported without semantic loss.
17. Craft with custom textures can be exported as a ZIP package containing manifest YAML and referenced assets.
18. A separate standalone flight module can parse the YAML craft file and reconstruct the same ship structure.
19. A separate standalone flight module can load the ZIP package, resolve packaged custom textures, and render the same customized ship appearance.
20. The editor renders CoM as a visible glowing marker and provides a visible CoT indicator with misalignment warning behavior.
21. Invalid placements, invalid dimension edits, broken asset references, and unsafe branch-switch scenarios are blocked or safely downgraded with clear feedback.
22. The result clearly feels like a KSP1-style assembly editor despite the reduced part catalog.

---

## 21. Suggested Future Extensions

Not required now, but compatible with this design:
- decouplers
- adapters
- command pods / avionics core
- batteries / solar panels
- docking ports
- landing legs
- fairings
- procedural interstages
- reusable subassemblies
- SPH plane editor mode
- staging configuration UI

---

## 22. Final Design Position

The correct target is **not** to copy every KSP1 system in full complexity.
The correct target is to preserve the **editor grammar** of KSP1:
- rooted craft construction
- intuitive snap placement
- stack and surface attachment
- symmetry-driven design
- fast iteration
- visually immediate feedback

while replacing the huge discrete parts catalog with a **small procedural parts ecosystem**.

That is the strongest product direction because it keeps what made the KSP1 editor great, while avoiding content bloat and enabling a cleaner technical foundation.

