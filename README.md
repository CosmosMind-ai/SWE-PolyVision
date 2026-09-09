---
license: cc-by-4.0
task_categories:
- text-generation
language:
- en
tags:
- software-engineering
- multimodal
- swe-bench
- code-repair
size_categories:
- n<1K
---

# SWE-PolyVision Public Tasks

Public question package for SWE-PolyVision from CosmosMind AI Lab.

This release contains the 48 tasks used in the benchmark. Each task includes only
the task statement, the fixed repository revision, and the visual evidence needed
to inspect the issue. Developer patches, reference fixes, verifier assets, model
results, traces, and answer materials are intentionally excluded.

The task statements are derived from the benchmark packages and scrubbed at the
boundary before the pre-fix discussion / reference-fix section.

## Layout

```
dataset.jsonl              one JSON object per line
tasks/<instance_id>/
  problem_statement.txt    issue text
  Dockerfile               environment definition
  meta.json                task metadata
  assets/                  issue screenshots; videos also carry
                           derived/<id>/frame-NN.png extracted frames
instance_ids.json          the 48 instance ids
```

## `dataset.jsonl` schema

```json
{
  "repo": "WordPress/gutenberg",
  "instance_id": "WordPress__gutenberg-77229",
  "base_commit": "763b5253aa8d3487ce10f95dfb983d9ca2ff00d5",
  "problem_statement": "issue text, including reproduction steps and image references",
  "created_at": "2026-04-10T12:35:52Z",
  "version": "v22.6.0-rc.1",
  "environment_setup_commit": "763b5253aa8d3487ce10f95dfb983d9ca2ff00d5",
  "image_assets": "{\"problem_statement\":[\"assets/WordPress__gutenberg-77229/001.png\"],\"patch\":[],\"test_patch\":[]}"
}
```

`image_assets` is a JSON-encoded string, following the SWE-bench convention.
Its paths are derived from the actual files under `tasks/<instance_id>/assets/`.

`created_at` and `version` are empty for a subset of tasks whose upstream
metadata did not record them.

## Citation

To be announced.
