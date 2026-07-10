# Graph Report - AlphaDentYolov26  (2026-07-08)

## Corpus Check
- 40 files · ~873,264 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 218 nodes · 251 edges · 28 communities (22 shown, 6 thin omitted)
- Extraction: 98% EXTRACTED · 2% INFERRED · 0% AMBIGUOUS · INFERRED: 4 edges (avg confidence: 0.5)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `1ad00379`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- compress.py
- README.md
- validate.py
- SKILL.md
- Caveman Help
- Caveman Compress
- SKILL.md
- AlphaDent
- caveman-commit
- caveman-review
- AlphaDent
- caveman-stats
- benchmark.py
- load_yolo_annotations
- Workflow: graphify
- IEEEtran.cls
- IEEEtran.cls
- graphify.md
- __init__.py
- CLAUDE.md

## God Nodes (most connected - your core abstractions)
1. `validate()` - 14 edges
2. `compress_file()` - 12 edges
3. `detect_file_type()` - 9 edges
4. `AlphaDent` - 9 edges
5. `should_compress()` - 8 edges
6. `AlphaDent` - 8 edges
7. `main()` - 7 edges
8. `Caveman Compress` - 7 edges
9. `Caveman Help` - 7 edges
10. `backup_dir_for()` - 6 edges

## Surprising Connections (you probably didn't know these)
- `benchmark_pair()` --calls--> `validate()`  [EXTRACTED]
  .agents/skills/caveman-compress/scripts/benchmark.py → .agents/skills/caveman-compress/scripts/validate.py
- `compress_file()` --calls--> `validate()`  [EXTRACTED]
  .agents/skills/caveman-compress/scripts/compress.py → .agents/skills/caveman-compress/scripts/validate.py
- `main()` --calls--> `backup_dir_for()`  [EXTRACTED]
  .agents/skills/caveman-compress/scripts/cli.py → .agents/skills/caveman-compress/scripts/compress.py
- `main()` --calls--> `compress_file()`  [EXTRACTED]
  .agents/skills/caveman-compress/scripts/cli.py → .agents/skills/caveman-compress/scripts/compress.py
- `main()` --calls--> `detect_file_type()`  [EXTRACTED]
  .agents/skills/caveman-compress/scripts/cli.py → .agents/skills/caveman-compress/scripts/detect.py

## Import Cycles
- None detected.

## Communities (28 total, 6 thin omitted)

### Community 0 - "compress.py"
Cohesion: 0.12
Nodes (27): main(), print_usage(), backup_dir_for(), build_compress_prompt(), build_fix_prompt(), call_claude(), compress_file(), is_sensitive_path() (+19 more)

### Community 1 - "README.md"
Cohesion: 0.09
Nodes (20): Before / After, Benchmarks, How It Work, <img src="../../docs/assets/dancing-rock.svg" width="20" height="20" alt="rock"/> Caveman (285 tokens), Install, 📄 Original (706 tokens), Part of Caveman, Security (+12 more)

### Community 2 - "validate.py"
Cohesion: 0.20
Nodes (17): count_bullets(), extract_code_blocks(), extract_headings(), extract_inline_codes(), extract_paths(), extract_urls(), Path, Line-based fenced code block extractor.      Handles ``` and ~~~ fences with v (+9 more)

### Community 3 - "SKILL.md"
Cohesion: 0.14
Nodes (12): cavecrew, Example chaining, How to invoke, Model overrides, See also, What it does, Auto-clarity (inherited), Chaining patterns (+4 more)

### Community 4 - "Caveman Help"
Cohesion: 0.14
Nodes (12): caveman-help, Example output, How to invoke, See also, What it does, Caveman Help, Configure Default Mode, Deactivate (+4 more)

### Community 5 - "Caveman Compress"
Cohesion: 0.17
Nodes (11): Boundaries, Caveman Compress, Compress, Compression Rules, Pattern, Preserve EXACTLY (never modify), Preserve Structure, Process (+3 more)

### Community 6 - "SKILL.md"
Cohesion: 0.17
Nodes (10): caveman, Example output, How to invoke, See also, What it does, Auto-Clarity, Boundaries, Intensity (+2 more)

### Community 7 - "AlphaDent"
Cohesion: 0.17
Nodes (11): AlphaDent, Citations, Convert 9 classes dataset to 4 classes, Dataset links, Draw Yolo annotations, Inference, Interactive Jupyter Notebook (Kaggle/Colab), Pretrained weights (+3 more)

### Community 8 - "caveman-commit"
Cohesion: 0.18
Nodes (9): caveman-commit, Example output, How to invoke, See also, What it does, Auto-Clarity, Boundaries, Examples (+1 more)

### Community 9 - "caveman-review"
Cohesion: 0.18
Nodes (9): caveman-review, Example output, How to invoke, See also, What it does, Auto-Clarity, Boundaries, Examples (+1 more)

### Community 10 - "AlphaDent"
Cohesion: 0.18
Nodes (10): AlphaDent, Citations, Convert 9 classes dataset to 4 classes, Dataset links, Draw Yolo annotations, Inference, Pretrained weights, Train (+2 more)

### Community 11 - "caveman-stats"
Cohesion: 0.29
Nodes (5): caveman-stats, Example output, How to invoke, See also, What it does

### Community 12 - "benchmark.py"
Cohesion: 0.60
Nodes (5): benchmark_pair(), count_tokens(), main(), print_table(), Path

### Community 13 - "load_yolo_annotations"
Cohesion: 0.50
Nodes (4): load_yolo_annotations(), Loads YOLO annotations for instance segmentation from a file.      :param anno, Visualizes YOLO annotations for instance segmentation on an image.      :param, visualize_yolo_instance_segmentation()

## Knowledge Gaps
- **99 isolated node(s):** `insert`, `update`, `insert`, `update`, `graphify` (+94 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **6 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `validate()` connect `validate.py` to `compress.py`, `benchmark.py`?**
  _High betweenness centrality (0.022) - this node is a cross-community bridge._
- **Why does `compress_file()` connect `compress.py` to `validate.py`?**
  _High betweenness centrality (0.011) - this node is a cross-community bridge._
- **What connects `Caveman compress scripts.  This package provides tools to compress natural lan`, `Split YAML frontmatter from body. Returns (frontmatter, body).      Memory fil`, `Resolve the out-of-tree backup directory for a given source file.      Backups` to the rest of the system?**
  _113 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `compress.py` be split into smaller, more focused modules?**
  _Cohesion score 0.12258064516129032 - nodes in this community are weakly interconnected._
- **Should `README.md` be split into smaller, more focused modules?**
  _Cohesion score 0.09090909090909091 - nodes in this community are weakly interconnected._
- **Should `SKILL.md` be split into smaller, more focused modules?**
  _Cohesion score 0.14285714285714285 - nodes in this community are weakly interconnected._
- **Should `Caveman Help` be split into smaller, more focused modules?**
  _Cohesion score 0.14285714285714285 - nodes in this community are weakly interconnected._