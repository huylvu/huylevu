# A Lightweight Stata Workflow for Research

A minimal yet solid Stata workflow for empirical research:

1. Project structure: `/data`, `/temp`, `/output`, `/code`
2. A master do-file orchestrating import, cleaning, building, estimation, export
3. Controlled logging, set seed, version control with Git

Tip: use `esttab`, `reghdfe`, `ppmlhdfe`, and `frames` (>=16) to organize data.
