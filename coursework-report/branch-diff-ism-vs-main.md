# Branch Comparison Report: ism vs main

Generated on: 2026-03-27

## Scope and Method

This report compares branch ism against main using merge-base comparison.

Commands used:
- git rev-list --left-right --count main...ism
- git diff --shortstat main...ism
- git diff --name-status main...ism
- git diff --numstat main...ism
- git log --oneline --no-merges main..ism

## Branch Divergence

Result:

```text
39 36
```

Interpretation:
- 39 commits are only on main.
- 36 commits are only on ism.

So the branches are diverged, not linear.

## Overall Delta Summary

```text
236 files changed, 84563 insertions(+), 261 deletions(-)
```

Status counts:

```text
A 215
M 21
```

Binary-like vs text entries:

```text
binary-like entries: 59
text entries: 177
```

## Concentration by Top-Level Area

```text
81 bram-mlp-base
76 docs
31 bram-mlp-with-tb
21 coursework-report
14 src
7  test
3  lab1_results
1  .gitignore
1  lab3.1.png
1  lab3.12.png
```

Main point: most branch volume is docs/notebooks/hardware artifacts; core framework code is a smaller but important subset.

## Detailed Core Source Differences

### 1) DRAM-aware hardware metadata routing

File:
- src/chop/passes/graph/analysis/add_metadata/add_hardware_metadata.py

Changes:
- add_component_source now takes pass_args and reads interface.storage.
- Adds DRAM branch to choose component/dependencies from DRAM_INTERNAL_COMP.
- Interface storage is no longer hardcoded BRAM; it is set from pass args.
- add_hardware_metadata_analysis_pass forwards pass_args into add_component_source.

Impact:
- Metadata can now explicitly switch emitted hardware behavior between BRAM and DRAM-backed parameter flow.

### 2) New DRAM component map

File:
- src/chop/passes/graph/analysis/add_metadata/hardware_metadata_layers.py

Changes:
- Adds DRAM_INTERNAL_COMP with linear and relu mappings.
- Adds common/rtl/dummy_param_dram.sv to DRAM dependency list.

Impact:
- DRAM mode now has a separate dependency path from default INTERNAL_COMP.

### 3) Top-level DRAM parameter stream ports in emit_top

File:
- src/chop/passes/graph/transforms/verilog/emit_top.py

Changes:
- Emits top-level ports for DRAM parameter streams:
	- <node>_<arg>
	- <node>_<arg>_valid
	- <node>_<arg>_ready
- Adds protocol comments for valid/ready transfer semantics.
- Skips internal BRAM source module instantiation for DRAM params.

Impact:
- Parameter source responsibility moves to top-level integration when DRAM is selected.

### 4) DRAM preloading path in cocotb emit_tb

File:
- src/chop/passes/graph/transforms/verilog/emit_tb.py

Changes:
- Adds DRAM detection from metadata.
- Adds fallback discovery of DRAM-style DUT ports from named model parameters.
- Instantiates StreamDriver per discovered DRAM parameter port.
- Quantizes/slices parameter tensors and queues DRAM blocks.
- Adds assertions/logging for missing or empty DRAM queues.

Impact:
- simulate now supports testing streamed parameter loading behavior, not only BRAM ROM flow.

### 5) DRAM-aware emit_bram behavior

File:
- src/chop/passes/graph/transforms/verilog/emit_bram.py

Changes:
- Adds DRAM branch to skip ROM/DAT generation.
- Removes stale BRAM source/dat files for DRAM-tagged parameters.
- Uses robust module config access for floor quantizer selection.

Impact:
- Prevents BRAM artifacts from conflicting with DRAM-mode top-level RTL/testbench builds.

### 6) New DRAM testbench emitter module

File:
- src/chop/passes/graph/transforms/verilog/dram_emit_tb.py (new)

Changes:
- Adds DRAM-oriented cocotb testbench generation path.

Impact:
- Introduces a dedicated DRAM TB implementation/prototype in the codebase.

### 7) Simulation defaults and wave handling

File:
- src/chop/actions/simulate.py

Changes:
- Default waves changed from False to True.
- waves is now passed into runner.build in addition to runner.test.

Impact:
- Wave generation is now default-on and consistently configured.

### 8) Additional non-DRAM functional fixes

Files and changes:
- src/chop/nn/quantizers/block_minifloat.py
	- Simplifies backward signature to backward(ctx, grad_output).
- src/chop/nn/quantizers/utils.py
	- Makes binary scaled quantizers dimension-agnostic.
- src/chop/passes/graph/transforms/pruning/prune.py
	- Uses module.weight directly for conv/linear weight source.
- src/chop/passes/graph/transforms/pruning/pruning_methods.py
	- Allocates masks on tensor.device.
- src/chop/passes/graph/transforms/verilog/emit_internal.py
	- Adds DRAM placeholder branch.
- src/chop/passes/graph/transforms/verilog/util.py
	- Adds DRAM placeholder branch.
- test/passes/graph/transforms/verilog/test_emit_verilog_linear.py
	- Updates pass args: wait_units key and shorter wait_time.

## Source Delta Magnitudes (src + targeted test)

```text
333   0  src/chop/passes/graph/transforms/verilog/dram_emit_tb.py
210   0  src/chop/passes/graph/transforms/verilog/emit_tb.py
80    2  src/chop/passes/graph/transforms/verilog/emit_top.py
39    0  src/chop/passes/graph/analysis/add_metadata/hardware_metadata_layers.py
34    6  src/chop/passes/graph/transforms/verilog/emit_bram.py
30   10  src/chop/passes/graph/analysis/add_metadata/add_hardware_metadata.py
10    8  src/chop/nn/quantizers/utils.py
6     4  src/chop/passes/graph/transforms/pruning/prune.py
6     0  src/chop/passes/graph/transforms/verilog/util.py
5     0  src/mase_components/common/rtl/dummy_param_dram.sv
5     0  src/chop/passes/graph/transforms/verilog/emit_internal.py
2     2  src/chop/passes/graph/transforms/pruning/pruning_methods.py
2     1  src/chop/actions/simulate.py
1     9  src/chop/nn/quantizers/block_minifloat.py
1     1  test/passes/graph/transforms/verilog/test_emit_verilog_linear.py
```

## Documentation and Artifact Breakdown

Status counts by area:

```text
docs/labs: A 31, M 1
docs/source/modules/documentation/tutorials: A 38, M 6
coursework-report: A 21
```

Notable additions:
- Multiple new/duplicated tutorial notebooks under docs/source/modules/documentation/tutorials.
- New classification model tutorial notebook and generated graph SVG outputs.
- Large generated hardware trees under bram-mlp-base and bram-mlp-with-tb.
- Report assets and LaTeX outputs under coursework-report.

### Exact core source/test files changed

```text
M src/chop/actions/simulate.py
M src/chop/nn/quantizers/block_minifloat.py
M src/chop/nn/quantizers/utils.py
M src/chop/passes/graph/analysis/add_metadata/add_hardware_metadata.py
M src/chop/passes/graph/analysis/add_metadata/hardware_metadata_layers.py
M src/chop/passes/graph/transforms/pruning/prune.py
M src/chop/passes/graph/transforms/pruning/pruning_methods.py
A src/chop/passes/graph/transforms/verilog/dram_emit_tb.py
M src/chop/passes/graph/transforms/verilog/emit_bram.py
M src/chop/passes/graph/transforms/verilog/emit_internal.py
M src/chop/passes/graph/transforms/verilog/emit_tb.py
M src/chop/passes/graph/transforms/verilog/emit_top.py
M src/chop/passes/graph/transforms/verilog/util.py
A src/mase_components/common/rtl/dummy_param_dram.sv
M test/passes/graph/transforms/verilog/test_emit_verilog_linear.py
```

### Exact coursework-report files added

```text
A coursework-report/BRAM/power.png
A coursework-report/BRAM/sythntehis report.txt
A coursework-report/BRAM/timing.png
A coursework-report/BRAM/util.png
A coursework-report/BRAM/utilisation report.txt
A coursework-report/DRAM/power on chip.png
A coursework-report/DRAM/power.png
A coursework-report/DRAM/timing 1.png
A coursework-report/DRAM/timing 2.png
A coursework-report/DRAM/util.png
A coursework-report/DRAM/utilisation report.txt
A coursework-report/outline-for-report.md
A coursework-report/report 2.aux
A coursework-report/report 2.out
A coursework-report/report.aux
A coursework-report/report.fdb_latexmk
A coursework-report/report.fls
A coursework-report/report.out
A coursework-report/report.pdf
A coursework-report/report.synctex.gz
A coursework-report/report.tex
```

## Largest Text Additions (Top Entries)

```text
18746  docs/source/modules/documentation/tutorials/tutorial_6_task1 (1).ipynb
9022   docs/source/modules/documentation/tutorials/tutorial_5_task_2 (1).ipynb
7308   bram-mlp-base/mlp_1/mlp_1.sim/sim_1/behav/xsim/xsim.dir/matrix_bank_behav/obj/xsim_15.c
3510   docs/source/modules/documentation/tutorials/Lab 3/tutorial_6_task1_clean.ipynb
3003   docs/source/modules/documentation/tutorials/bert-base-uncased.svg
2911   docs/source/modules/documentation/tutorials/mase_graph.svg
2797   docs/source/modules/documentation/tutorials/tutorial_5_task1_separated.ipynb
2797   docs/source/modules/documentation/tutorials/Lab2/tutorial_5_task1_separated.ipynb
1903   docs/source/modules/documentation/tutorials/Lab 1/tutorial_2_lora_finetune.ipynb
1702   docs/source/modules/documentation/tutorials/classification-model.ipynb
```

## Commit Timeline (ism-only, non-merge)

```text
572eb21 Random latex files
e07644b fixed BRAM not working on tb
44544bf report update
9e79dee Report changes
489679a Updated QAM model
e8b4607 Report progress, increasing QAM precision
32b9fcc started on the report
debea21 working testbench
6ec0f83 Ali's Stuff
42527ea initial commit on new DRAM tb generation
4c07530 DRAM and BRAM data files
d9f66c5 Decent progress on hardware metadata pass to the classification model
4d4ec54 new notebook for generating the bram mlp
9150e9c added bram mlp testbench
867a6e4 added bram mlp for help with testbenches
2a36096 includes plan for Ali
afeaaf5 initial commit for training model
e813cfd towards constructing my own top file
f5d2c24 added a comment in the VerilogInterfaceEmitter: "this is for DRAM"
136a5a2 Didn't add the md file
ad332c3 Have a read of dram weight streaming, detailed how we might start opening ports up and getting working
c8a1494 We now understand DRAM
794c8d1 Working metadata pass
5b2cd4b Trying to add hardware  metadata
bbfd14c Old lab garbage
2d446d7 Fixing compression comparison
be0c360 Lab 3 results
1eb6854 Help me seriosusly
51153e2 sdhgishgoih
6681387 HELP ME
2eca174 HEGOIADHOIHAIOHIRGH
3e10177 Good work on tutorials
```

## Full Inventory Command

For the full file-level inventory used in this report:

```text
git diff --name-status main...ism
```

## Review Recommendation

To make review and merge safer, split into separate PRs/patches:
- Core framework logic (src + targeted tests).
- Documentation/tutorial notebooks.
- Generated hardware and simulation artifacts.
- Report assets and LaTeX outputs.

