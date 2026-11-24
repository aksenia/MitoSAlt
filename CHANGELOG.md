# Changelog

## 1.1.2 (unreleased)

Error handling and robustness. The deletion/duplication calling is unchanged.

### Upgrade notes
- Runs that fail now exit with a non-zero code (1, 2 or 3) instead of 0; wrappers that relied on exit code 0 should check it.
- The scripts are now `MitoSAlt.pl` and `MitoSAlt_SE.pl`; `MitoSAlt1.1.1.pl` and `MitoSAlt_SE1.1.1.pl` remain as links to them.
- The version is read from the `VERSION` file.

### Added
- Exit codes: 0 success, 1 invalid input or configuration, 2 a pipeline step failed, 3 no MT reads or alignments left after a step.
- `log/<study name>.summary.tsv`, written on success and on failure, one `key<TAB>value` per line: `status` (running/ok/failed), `exit_code`, `message` (failed runs), `version`, `sample`, `mt_reads` (reads on the MT contig in the nuclear+MT alignment), `mt_alignments` (alignments in the MT BAM), `mt_mean_depth` (mean coverage of the MT contig in the MT BAM, to judge whether a run without events had enough coverage), `split_reads` (lines in `.breakpoint`), `clusters` (lines in `.cluster`), `events_reported` (rows in `.tsv`), `warnings` (Perl warnings during the run). Keys for steps that did not run are omitted.
- The summary is written as `status running` at the start, so a killed run is recognisable; Perl errors and SIGINT/SIGTERM/SIGHUP end the run with exit code 2 and a summary.
- The final `.tsv` is always written; a header-only file means no events passed the filters.
- `--version` option.
- Checks before running: arguments, configuration keys, STEPS values, numeric parameter values, tools, indexes and reference files for the enabled steps, and that `refchr` is a contig in the reference indexes.
- Optional delplot.R arguments for the plot and TSV output directories.

### Changed
- Every external command is checked, with `pipefail`, and the pipeline stops on the first failure instead of finishing with exit code 0. Temporary files are kept when a run fails.
- delplot.R is run with Rscript from the script directory; its messages go to the run log instead of delplot.Rout.
- delplot.R stops with a message when an R package is missing instead of installing packages during a run.
- Output directories are created automatically.
- One log per run, `log/<study name>.log`, also shown on the terminal: it starts with the version, command, host and working directory and holds our messages and the output of every tool, including HISAT2 and BBMap, which previously wrote only to the file. It is overwritten by each run instead of appended to. Perl warnings are marked `WARNING:`; R warnings are printed as they occur.
- Missing configuration or FASTQ files are reported in the log and summary.

### Fixed
- `insall.packages` typo in delplot.R.
- The copy number step (cn_mt) stops with a message when no random regions are generated from `gsize` (exit code 2) or there are no nuclear reads in them, e.g. for MT-only input (exit code 3); `bedtools groupby` reports this but exits 0, so the nuclear coverage line was silently missing from `.cnmt.txt`.
- Writes of `.breakpoint`, `.bed`, `.cluster` and the temporary BED files are checked, so a full disk stops the run instead of leaving truncated files; commands whose output is read directly (`check_paired`, the MT alignment count) are checked with `pipefail` too.
- The summary is written to a temporary file and renamed, and a failed write ends the run with an error; reading intermediate files is checked as well.
- The copy number step stops when `sambamba depth window` writes no coverage rows.
- delplot.R stops when clusters have no reads in the `.breakpoint` file instead of writing a header-only `.tsv`.
- A `plot/<study name>.pdf` from an earlier run is removed before plotting, so it cannot be mistaken for this run's result.
- Host shell startup files given in `BASH_ENV`/`ENV` (e.g. an lmod setup passed through by Singularity) are no longer run for every command.
- README: script names, the `hsindex` key, the `lastdb` path, the alignment table file name and the `.tsv` column names now match the code.
