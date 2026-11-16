---
name: honesty-pass
description: Use when code must be forced into honest contracts. This skill defines mandatory rules for functions whose names, return types, and caller behavior do not agree.
---

# Honesty Pass

This is a hard gate. These are mandatory rules. If code violates a rule, rewrite the code.

This skill is a responsitory of fraud patterns, which are hard violations that are unacceptable rot and must be removed.

When this skill is invoked, immediately apply it to the current changeset unless the user names a narrower target.

## Dishonest prerequisites

Functions with dishonest prerequisites acceptsraw, under-proven data, then starts slicing, converting, indexing, or unwrapping as if the required structure was already guaranteed. When that guarantee is missing, it panics. The result is branchy, weaker, less optimizable code that makes the compiler sit out while the function improvises its own broken proof system. That is not a runtime surprise. That is a boundary failure.

The solution is to move the proof upward. If the function needs validated bytes, exact slice lengths, a resolved handle, a loaded definition, or a specific mode, it must demand those as inputs. Honest input removes most panics because the bad states stop being representable at the point of use. If a function would panic because of a missing required value, make the value an input instead.

## Case 1:

BAD:

```rust
fn schematic_pointer_world_per_pixel(
    orthographic_height: f32,
    window_height: f32,
    view_height_scale: f32,
) -> f32 {
    assert!(
        orthographic_height.is_finite() && orthographic_height > 0.0,
        "schematic pointer orthographic height should be positive and finite"
    );
    assert!(
        window_height.is_finite() && window_height > 0.0,
        "schematic pointer window height should be positive and finite"
    );
    assert!(
        view_height_scale.is_finite() && view_height_scale > 0.0,
        "schematic pointer view height scale should be positive and finite"
    );
    orthographic_height / window_height * view_height_scale
}
```

GOOD: Make the restriction a prerequisite.

```rust
fn world_units_per_pixel_from_orthographic_height(
    orthographic_height: PositiveFiniteF32,
    window_height: PositiveFiniteF32,
    view_height_scale: PositiveFiniteF32,
) -> f32 {
    orthographic_height.get() / window_height.get() * view_height_scale.get()
}
```

GOOD: Not assert at all, not your responsibility.

```rust
fn world_units_per_pixel_from_orthographic_height(
    orthographic_height: f32,
    window_height: f32,
    view_height_scale: f32,
) -> f32 {
    orthographic_height / window_height * view_height_scale
}
```

### Case 2:

BAD:

```rust
fn animate_camera_to_selected_deck(
    camera: &mut CameraController,
    target_bounds: CameraTargetBounds,
) {
    assert!(
        camera.mode() != CameraMode::Orbit,
        "deck alignment should only run in cutaway modes"
    );
    let position = camera.target().target();
    camera.set_target(
        target_bounds.clamp_target(Vec3::new(
            position.x,
            deck_target_y(camera.deck().selected_layer()),
            position.z,
        )),
        CameraTransition::Smooth,
    );
}
```

GOOD: Accept a narrower type that already proves this operation is valid.

```rust
fn animate_cutaway_camera_to_selected_deck(
    camera: &mut CutawayCameraController,
    target_bounds: CameraTargetBounds,
)
```

Case 3:


BAD: get_safe is vague, get hides a panic, the type is mixing storage access with caller policy about required membership

```rust
#[must_use]
pub(crate) fn get_safe<Q>(&self, key: &Q) -> Option<&V>
where
    K: Borrow<Q>,
    Q: Eq + Hash + ?Sized,
{
    self.map.get(key).map(Box::as_ref)
}

#[must_use]
pub(crate) fn get<Q>(&self, key: &Q) -> &V
where
    K: Borrow<Q>,
    Q: Debug + Eq + Hash + ?Sized,
{
    self.get_safe(key)
        .unwrap_or_else(|| panic!("{} should contain key {key:?}", self.definitions_label))
}
```

GOOD: deleting the panicking get, keeping only optional lookup at this boundary, and reshaping callers so strict functions take &V as an input prerequisite

```rust
pub(crate) fn get<Q>(&self, key: &Q) -> Option<&V>
where
    K: Borrow<Q>,
    Q: Eq + Hash + ?Sized,
{
    self.map.get(key).map(Box::as_ref)
}
```

Then move the prerequisite upward:

```rust
let Some(definition) = definitions.get(key) else {
    // handle it.
};
```

## Dishonest Return Types

Dishonest return types are fake certainty. This is coward code. If a function name promises a type, it must return that type. Not Option<T>, a panic, a partial shell, or a value the caller has to unwrap, assert, or panic to recover. That is a boundary lie.

The solution is to move the choice upward: the function must DEMAND all prerequisites in its parameters such that it can guarantee the output.

### Case 1

BAD: `place_entity_preview(...)` does not return a directly usable preview.

```rust
let preview = place_entity_preview(scene_state.interaction());
let Some(ghost_highlight) = preview.ghost else {
    panic!("place-entity preview should output ghost preview state");
};
```


```rust
let preview = place_entity_preview(...);
assert!(preview.some_preview_field.is_some());
```

### Case 2

BAD: this operation requires a valid occupied render chunk slot, slot alone is not enough input to prove that

```rust
fn update_render_chunk_slot(&mut self, slot: RenderChunkSlot, /* ... */) {
    let retained = self
        .render_chunk_slots
        .get_mut(slot)
        .unwrap_or_else(|| panic!("camera render chunk slot {slot} should exist"));

    // work on retained
}
```

GOOD:

```rust
fn update_render_chunk_retained(
    retained: &mut RenderChunkRetained,
    /* ... */
) {
    // work on retained
}
```

## Dishonest content

Dishonest content is not just weak contracts, its actively lying about what it is.

The code uses domain words for generic behavior, wraps one concept in another name without adding a contract, or hides real behavior behind a more respectable surface. This is worse than an awkward API. It manufactures false meaning. It teaches the reader the wrong model of the code, invites copy-paste mutations of the same lie, and spreads counterfeit specificity across the codebase.

The solution is to just delete it and merge upward, drawing boundaries along more honest lines.

BAD: fakery.

```rust
fn u32_layer(layer: i32, name: &str) -> u32 {
    u32::try_from(layer).unwrap_or_else(|_| panic!("{name} should be nonnegative"))
}
```

First, it lies about domain meaning. u32_layer claims to know something about layers, but it does not. It is just a generic integer conversion.

Second, it lies about responsibility. It looks like a small conversion helper, but it secretly owns panic policy.

Third, it lies about reusability. Because it is named for one domain concept while doing generic work, it invites copy-paste variants like:

- u32_row
- u32_column
- u32_depth
- u32_index

That is how codebases rot. Generic behavior gets wrapped in fake domain names, and every caller starts depending on tiny panic-decorated helpers instead of proving prerequisites honestly.

So this kind of function is dangerous because it:

- hides a generic conversion behind fake domain vocabulary
- hides a panic behind a helper-looking name
- couples domain wording to low-level numeric conversion
- encourages duplication of the same bad pattern everywhere

BAD: Results that panic.

```rust
pub(crate) fn glb_json_root_value(absolute_path: &Path) -> Result<serde_json::Value, String> {
    let bytes = std::fs::read(absolute_path)
        .map_err(|error| format!("failed to read GLB `{}`: {error}", absolute_path.display()))?;
    if bytes.len() < 20 {
        return Err(format!("GLB `{}` is too short", absolute_path.display()));
    }
    let magic = u32::from_le_bytes(
        bytes[0..4]
            .try_into()
            .unwrap_or_else(|_| panic!("GLB header magic slice should be exactly four bytes")),
    );
    if magic != 0x4654_6C67 {
        return Err(format!(
            "file `{}` is not a GLB container",
            absolute_path.display()
        ));
    }
    let json_chunk_length = u32::from_le_bytes(
        bytes[12..16]
            .try_into()
            .unwrap_or_else(|_| panic!("GLB JSON chunk length slice should be exactly four bytes")),
    ) as usize;
    let json_chunk_kind = u32::from_le_bytes(
        bytes[16..20]
            .try_into()
            .unwrap_or_else(|_| panic!("GLB JSON chunk kind slice should be exactly four bytes")),
    );
    if json_chunk_kind != 0x4E4F_534A {
        return Err(format!(
            "GLB `{}` is missing a JSON chunk",
            absolute_path.display()
        ));
    }
    let json_end = 20 + json_chunk_length;
    if json_end > bytes.len() {
        return Err(format!(
            "GLB `{}` declares a JSON chunk longer than the file",
            absolute_path.display()
        ));
    }
    serde_json::from_slice(&bytes[20..json_end]).map_err(|error| {
        format!(
            "failed to parse GLB JSON chunk from `{}`: {error}",
            absolute_path.display()
        )
    })
}
```

It is dishonest because it claims to be a Result-returning boundary parser, but it still hides panic paths inside boundary-data handling. The worst part is this pattern:

bytes[0..4]
    .try_into()
    .unwrap_or_else(|_| panic!(...))

and the two copies below it. The rule violation is Result boundary for external file parsing, hidden panic paths on external data, and panic-based slice conversion where the real honest shape is infallible extraction after prerequisite proof.

The honest fix is to remove the fake fallibility at the slice-conversion step entirely: extract the four-byte arrays in a way that is structurally infallible after the length check or return Err if you want that shape checked dynamically. But the better fix is to check the length ahead of time, or demand the length in the params--That's not just cleaner, it's faster, and the compiler becomes your enforcer. Dishonest boundaries are not just confusing. They are often slower, branchier, and weaker than the honest alternative.

## Maybe functions

Maybe functions are often indecision forwarders, and a structural lie. They almost promise something, but back out coyly with a "lol just kidding!". It's coward code that refuses to prove the prerequisite, refuses to handle the absence, and instead turned uncertainty into everyone else's problem.

The solution is to cut it in half. One early function finds or proves the prerequisites. The next function demands them and actually guarantees the result.

BAD:
```
fn maybe_place_item_preview(
    interaction: &crate::interactions::state::InGameInteractionState,
) -> Option<PlaceItemPreview> {
    let Some(item_kind_id) = interaction.place_item().active_item_kind_id() else {
        return None;
    };
    let Some(contact) = interaction.surface_contact() else {
        return None;
    };
    let preview_cell = solid_placement_cell(contact.support());
    let catalog_entry = item_browser_catalog_entry_by_id(item_kind_id);
    let ghost = GhostPreviewState {
        anchor: HighlightAnchor::WorldTransform(Transform::from_translation(
            crate::cell_range::NormalizedCellRange::from_endpoints(preview_cell, preview_cell)
                .center_position(),
        )),
        display_state: HighlightDisplayState::Valid,
        preview_scale: crate::volume_static_entities::vec3_from_cell_size(
            catalog_entry.preview_cell_size,
        ),
    };
    let commit = npc_world_cell_item_location(contact.support()).map(|location| PlaceItemCommit {
        item_kind_id: item_kind_id.to_owned(),
        location,
    });
    Some(PlaceItemPreview { ghost, commit })
}
```

This is many functions jammed together, they need to be split into parts that can be honest about what they guaranteed.

GOOD:

```rust
let Some(prerequisites) = place_item_preview_prerequisites(interaction) else {
    clear_place_item_preview_state(...);
    return None;
};

let preview = place_item_preview(prerequisites);
```

BAD:

```rust
    /// Returns either a dynamic offset index or [`PhaseItemExtraIndex::None`],
    /// as appropriate.
    pub fn maybe_dynamic_offset(dynamic_offset: Option<NonMaxU32>) -> PhaseItemExtraIndex {
        match dynamic_offset {
            Some(dynamic_offset) => PhaseItemExtraIndex::DynamicOffset(dynamic_offset.into()),
            None => PhaseItemExtraIndex::None,
        }
    }
```

This might seem fine, but it's the same problem because "PhaseItemExtraIndex" only exists to be a long-distance flag, an Option smuggling container! The code did not decide anything, it just pushed indecision farther downstream.

## Thesaurus code

Thesaurus code is bullshit indirection. It's semantic spam that does not add a contract, an invariant, a transformation, or a boundary. It just takes one concept, gives it a second name, and forces the reader to pretend those names mean different things. Nothing was clarified. Nothing was made safer.

The solution is to just delete the rename and merge upward.

### Examples

BAD: This does not define a new contract. It just renames contains into contains_item_stack.

```rust
#[must_use]
pub fn contains_item_stack(&self, item_stack_id: u64) -> bool {
    self.item_records.contains(&item_stack_id)
}
```

BAD:

```rust
pub fn apply_authoritative_mutation(
    &mut self,
    mutation: ModuleGraphMutation,
    catalog: &NodeManifestResource,
) -> Result<(), ModuleGraphError> {
    self.apply_mutation(mutation, catalog)
}
```

apply_mutation already means mutation application on that graph. apply_authoritative_mutation just renames the same action with a more dramatic noun phrase.


BAD: Types used as field names.

```rust
#[derive(Clone, Debug, Deserialize, Eq, Hash, PartialEq)]
#[serde(transparent)]
pub struct NpcRigProfileId(pub String);

impl NpcRigProfileId {
    #[must_use]
    pub fn as_str(&self) -> &str {
        self.0.as_str()
    }
}

impl From<&str> for NpcRigProfileId {
    fn from(value: &str) -> Self {
        Self(value.to_owned())
    }
}
```

That's just a string with helpers, not an id. Name the field as an id, not the type.

BAD: 

```rust
#[derive(Component, Clone, Copy, Debug, Eq, PartialEq)]
pub(crate) struct RetainedMeshModelRoot(pub(crate) Entity);
```

That's literally the entire struct--just a name to hide the original type.

BAD:

```rust
pub struct NodeManifestResource {
    manifest: &'static NodeManifest,
}
```

BAD:
```rust
pub struct AuthoritativeModuleGraph {
    graph: ModuleGraph,
}
```

For AuthoritativeModuleGraph and NodeManifestResource, they only exist to carry the thing inside and to give it more context. That is pointless, dishonest, and even hurts performance.

BAD:
```rust
pub(crate) struct ProjectionMesh(Handle<Mesh>);
```

## Extreme examples

BAD: Extreme example of bad Rust code.

```rust
fn mesh_float32x4_attribute(
    mesh: &Mesh,
    attribute_id: bevy::mesh::MeshVertexAttributeId,
) -> Vec<[f32; 4]> {
    match mesh.attribute(attribute_id).unwrap_or_else(|| {
        panic!("retained GPU mesh payload should keep Float32x4 attribute {attribute_id:?}")
    }) {
        VertexAttributeValues::Float32x4(values) => values.clone(),
        other => panic!("retained GPU mesh payload expected Float32x4, found {other:?}"),
    }
}
```

It fails on:

  - dishonest name
  - dishonest inputs
  - hidden panic policy
  - hidden clone policy
  - fake generality
  - domain truth only visible in the panic text

## Workflow

Review the current changeset only for honesty.

Use the examples and rules above as hard gates. Do not excuse, defer, or normalize dishonest code because it is small, convenient, or
already working.

For each dishonest function, type, or boundary:

1. Identify the exact lie.
2. Identify the honest boundary and the honest prerequisite.
3. Decide the strongest honest shape.
4. Implement the fix immediately.
5. Re-check the surrounding callers and callees, because dishonesty usually spreads across the boundary.

Repeat until the entire changeset is honest.

Do not leave dishonest code behind for a later cleanup pass.
Do not commit dishonest code.
Do not stop at renaming if the structure is still lying.
