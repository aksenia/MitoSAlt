# Changelog

## Unreleased

Error handling and robustness. The deletion/duplication calling is unchanged.

### Added
- Exit codes: 0 success, 1 invalid input or configuration, 2 a pipeline step failed, 3 no MT reads or alignments left after a step.
- `log/<study name>.summary.tsv`, written on success and on failure, one `key<TAB>value` per line: `status` (ok/failed), `exit_code`, `message` (failed runs), `version`, `sample`, `mt_reads` (reads on the MT contig in the nuclear+MT alignment), `mt_alignments` (alignments in the MT BAM), `split_reads` (lines in `.breakpoint`), `clusters` (lines in `.cluster`), `events_reported` (rows in `.tsv`). Keys for steps that did not run are omitted.
- The final `.tsv` is always written; a header-only file means no events passed the filters.
- `--version` option.
- Checks before running: arguments, configuration keys and STEPS values, tools, indexes and reference files for the enabled steps, and that `refchr` is a contig in the reference indexes.
- Optional delplot.R arguments for the plot and TSV output directories.

### Changed
- Every external command is checked, with `pipefail`, and the pipeline stops on the first failure instead of finishing with exit code 0. Temporary files are kept when a run fails.
- delplot.R is run with Rscript from the script directory; its messages go to the run log instead of delplot.Rout.
- delplot.R stops with a message when an R package is missing instead of installing packages during a run.
- Output directories are created automatically; STDERR is written to the run log as well.

### Fixed
- `insall.packages` typo in delplot.R.
- The copy number step (cn_mt) stops with a message when no random regions are generated from `gsize` (exit code 2) or there are no nuclear reads in them, e.g. for MT-only input (exit code 3); `bedtools groupby` reports this but exits 0, so the nuclear coverage line was silently missing from `.cnmt.txt`.
- Host shell startup files given in `BASH_ENV`/`ENV` (e.g. an lmod setup passed through by Singularity) are no longer run for every command.
- README: script names, the `hsindex` key, the `lastdb` path, the alignment table file name and the `.tsv` column names now match the code.
