# subject_hexgrid — workflow notes

Subject observation density map using H3 hexagonal binning. Optional clip
boundary and overlay layer (both from ER spatial feature groups). Published at
`github.com/cllrssml/subject-hexgrid`.

Custom package: `hex-tasks` at `/home/sam/Ecoscope_Projects/hex-tasks/`.

---

## Task chain

`set_workflow_details` → `set_er_connection` → `set_time_range` →
`set_subject_group_name` → `get_subjectgroup_observations` →
`set_boundary_group_name` →
`get_spatial_features_group` (boundary; `skipif: any_dependency_is_empty_string`) →
`set_overlay_group_name` →
`get_spatial_features_group` (overlay; `skipif: any_dependency_is_empty_string`) →
`set_colormap` → `set_base_maps` → `set_h3_resolution` →
`hex_bin_observations` (clips to boundary if provided; handles non-polygon geometry) →
`classify_hex_bins` → `apply_color_map` →
`create_hex_polygon_layer` (per-row contrasting outlines) →
`normalize_boundary_geometry` → `create_polygon_layer` (boundary outline; `legend: ~`) →
`create_overlay_layer` (splits mixed geometry; returns list) →
`combine_map_layers` → `draw_ecomap` →
`persist_text` → `create_map_widget_single_view` →
stat chain: `count_observations` → widget_obs_count →
`count_subjects` → widget_subject_count →
`count_hexbins` → widget_hexbin_count →
`format_h3_resolution` → widget_resolution →
`format_optional_name` (boundary) → widget_boundary →
`format_optional_name` (overlay) → widget_overlay →
`gather_dashboard` (`time_range: ~`).

Step id for h3 resolution MUST be `h3_resolution`, NOT `set_h3_resolution`.
Step id for colormap MUST be `colormap_picker`.

## Dashboard layout

8 widgets — `widget_id` order matches `gather_dashboard` widgets list:

| widget_id | Widget | x | w | y | h |
|---|---|---|---|---|---|
| 0 | Observations | 0 | 3 | 0 | 3 |
| 1 | Subjects | 3 | 2 | 0 | 3 |
| 2 | Hexbins | 5 | 2 | 0 | 3 |
| 3 | Resolution | 7 | 3 | 0 | 3 |
| 4 | Group | 0 | 3 | 3 | 3 |
| 5 | Boundary | 3 | 4 | 3 | 3 |
| 6 | Overlay | 7 | 3 | 3 | 3 |
| 7 | Map | 0 | 10 | 6 | 16 |

Row 1 (y=0, h=3): `3+2+2+3 = 10`. Row 2 (y=3, h=3): `3+4+3 = 10`.
Map full-width (y=6, h=16). Give the widest slot (w=4) to Boundary — longest expected value.

## rjsf-overrides

All three group name fields get live ER dropdowns (Web only):
```yaml
rjsf-overrides:
  properties:
    subject_group.properties.group_name.ecoscope:transform:
      - transformer: EarthRangerEnumResolver
        transformer_kws:
          type: subject_group_name
          depends_on: properties.er_client_name.properties.data_source.properties.name
    boundary_name.properties.group_name.ecoscope:transform:
      - transformer: EarthRangerEnumResolver
        transformer_kws:
          type: spatial_feature_group
          depends_on: properties.er_client_name.properties.data_source.properties.name
    overlay_name.properties.group_name.ecoscope:transform:
      - transformer: EarthRangerEnumResolver
        transformer_kws:
          type: spatial_feature_group
          depends_on: properties.er_client_name.properties.data_source.properties.name
```

## hex-tasks package

All tasks verified working in this workflow:

- `hex_bin_observations` — H3 hexbin counts with optional polygon clip; `h3_resolution` is plain `int`
- `set_subject_group_name`, `set_boundary_group_name`, `set_overlay_group_name` — form fields
- `set_h3_resolution` — form field (step id: `h3_resolution`)
- `format_h3_resolution` — returns "Resolution N (~X km²)" string
- `set_colormap` — colormap picker (step id: `colormap_picker`)
- `normalize_boundary_geometry` — converts any geometry to Polygon/MultiPolygon (convex hull fallback)
- `create_hex_polygon_layer` — hex layer with per-row contrasting outline (white/black by fill luminance)
- `create_overlay_layer` — splits GDF by geometry class; returns a list; `combine_map_layers` handles via `extend`
- `combine_map_layers` — merges hex + optional boundary + optional overlay; handles SkipSentinel and list overlays
- `classify_hex_bins` — Natural Breaks on `count`; adds `count_class` and `count_range` columns
- `count_observations`, `count_subjects`, `count_hexbins`, `max_hexbin_count` — stat scalars
- `format_optional_name` — returns group name or "—" if empty/skipped
- `export_geodataframe` — **deprecated.** Use built-in `persist_df` with `filetype: gpkg` instead
- `combine_patrol_layers` — legacy; use `combine_map_layers` for new workflows

## Boundary workflow pattern (optional clip)

```
set_boundary_group_name          # empty string = skip
  → get_spatial_features_group   # skipif: any_dependency_is_empty_string
  → hex_bin_observations         # boundary= clips hexbins; handles non-polygon internally
  → normalize_boundary_geometry  # converts ER LineString features to polygon (for rendering)
  → create_polygon_layer         # legend: ~  (suppress — outline only)
                                 # layer_style: filled: false, stroked: true
  → combine_map_layers           # handles SkipSentinel; do NOT add any_dependency_skipped
```

Key partial flags:
- `boundary_layer` → `legend: ~`
- `boundary_features` → `skipif: any_dependency_is_empty_string`
- `boundary_polygon` and `boundary_layer` → `skipif: any_dependency_skipped, any_is_empty_df`
- `combine_map_layers` → NO `any_dependency_skipped` in skipif (handles SkipSentinel itself)

## Post-compile patch

```bash
cp -r /home/sam/Ecoscope_Projects/hex-tasks ecoscope-workflows-*-workflow/hex-tasks
sed -i 's|path = "/home/sam/Ecoscope_Projects/hex-tasks"|path = "./hex-tasks"|' \
  ecoscope-workflows-*-workflow/pixi.toml
cd ecoscope-workflows-*-workflow && pixi install && cd ..
```

## GitHub

Repo: `github.com/cllrssml/subject-hexgrid`. `hex-tasks` bundled inside compiled dir.
Community post: community.earthranger.com/c/ecoscope/16 (June 2026) — first
community-published Platform SDK workflow.
