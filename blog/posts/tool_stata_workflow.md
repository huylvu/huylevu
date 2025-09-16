
When we start using Stata—or any software—we often postpone setting up a clean folder and file structure. Most people only think about organizing code and data at the very end, or when a coauthor asks for replication files. That is exactly what we want to avoid.

Workflow management is critical. Setting up folders should be one of the first steps; otherwise you will waste time later figuring out what you did, where the latest code lives, and which files are the authoritative ones. A good litmus test: when you open an old project, you should immediately understand the purpose of each folder and file. This guide covers practical workflow principles. It is not about Stata syntax per se, but about organizing code and related assets. While Stata examples are used here, the workflow tips generalize to any language or tool. These are not rigid rules, but helpful best practices to get your research projects organized.

This guide has six parts:

- Part 1: Organize folders
- Part 2: Name files and folders consistently
- Part 3: Use project-relative paths
- Part 4: Split work across focused do-files
- Part 5: Master do-file to run the project end-to-end
- Part 6: Code styling

# Part 1: Put EVERYTHING into folders

For data analysis, this includes obtaining raw data, writing scripts to clean and transform it, assembling analysis datasets, and producing tables and figures—often with multiple versions of each. Research also involves literature, writing drafts, project docs, reports, and deliverables—all versioned too.

ALL of these files should live in structured folders. For small projects, a simple structure works well:

Number folders by order of use. For example, first you collect raw data (`01_raw`), then write scripts (`02_dofiles`). While scripting you produce temporary files (`03_temp`) before saving final curated data into the main folder (`04_master`). You might also keep geospatial data in its own folder (`05_GIS`). Once data are clean, use them to produce tables (`06_tables`) and figures (`07_figures`). Numbering enforces workflow order and keeps lists sorted logically, instead of alphabetically.

> Tip: Avoid spaces in file and folder names, and avoid special characters (`~#@!\%^&` and `-`). Prefer underscores `_`.

> Tip: Prefer lowercase for file, folder, and variable names. It speeds up typing and reduces friction. Reserve capitalization for labels and presentation.

Keep variable names simple. Stata limits variable names to 32 characters—staying within that is good practice. If your raw data arrive messy, clean variable names early using string functions or regular expressions.

## Managing larger projects

For larger, multi-dataset, multi-paper projects, separate data setup from analysis. A common structure looks like this:

Here, the `data` directory contains raw data and scripts that clean them to produce final datasets under the master folder. This separation scales well as data management grows complex (merging, reshaping, labeling, GIS joins, etc.). Final datasets are then consumed by paper-specific do-files inside `paperX/` folders, each of which has a “small project” structure of its own.

> Tip: Always include a `README.txt` in the project root and, if helpful, inside key subfolders. Document directory purpose, file order, and how to access data (links, tables, or APIs). Future-you and collaborators will thank you.


# Part 2: Keep files in order

Scripts should be ordered so you can quickly tell what runs first and which version is current. I prefer both prefixes (execution order) and suffixes (versioning). Example patterns:

```
01_setup_v4.do  
02_merge_v2.do  
03_tables_v11.do  
04_regressions_v15.do  
05_figures_v2.do  
06_master_v1.do
```

You can also use dated suffixes, e.g. `01_setup_210531.do`, `01_setup_210602.do`.

> Tip: Use yymmdd date format for easy chronological sorting.

> Tip: Archive older versions outside the main working folder (e.g. an `archive/` subfolder).

Apply consistent naming beyond do-files: to figures, tables, and draft versions. It pays off when you need to find the right artifact among many.
 

# Part 3: Use project-relative paths

Set the project root once, then refer to everything else relative to it. Avoid hard-coded absolute paths scattered across files.

Example:

```
global MyProject "/Users/levuhuy/Library/CloudStorage/OneDrive-Personal/2_stata/1_ves/analysis" 
	global data "$MyProject/data"
		global   raw         "$data/raw"                    // raw data for analysis
		global   clean       "$data/clean"                  // cleaned data
		global   temp        "$data/temp"                   // temporary data
		global   lab         "$data/label_data"             

```

Note the use of `//` for inline comments. Once this runs, you are effectively “in” the project, and subfolders can be referenced relatively. For example, to read a raw dataset:

```
use "$raw/rawdata", clear
```

And after cleaning:

```
use "$raw/rawdata", clear

<some cleaning commands>

order  var1 var2 x* y*
sort   var1 var2
compress

save "$temp/rawdata.dta", replace
```

First, organize columns with `order`. Then sort and `compress` to save space (especially useful for large datasets). Keep creation of core variables in setup/merge scripts when possible.


# Part 4: Split tasks across focused do-files

Reuse the earlier pattern:

```
01_setup_v4.do  
02_merge_v2.do  
03_tables_v11.do  
04_regressions_v15.do  
05_figures_v2.do  
06_master_v1.do
```

`01_setup_v4.do` loads and cleans raw data and writes to `temp/`. You might have multiple setup files:

```
01_setup_emissions_v1.do  
01_setup_economic_v4.do
```

The merge script assembles cleaned sources into a master analysis file. Version with dates if you like, but stay consistent. Authors often split tables and figures into separate do-files; if you do, reflect the table/figure numbers in filenames:

```
03_table1_v1.do  
03_table2_v4.do
04_regression1_v2.do  
04_regression2_v6.do
05_figure1_v1.do  
05_figure3_v5.do
```

## A few tips for new projects

Early phases are messy even with folders. A typical flow might be: import → create variables → make some figures → create more variables → run regressions.

> Tip: Segment code within a do-file so it can be split later, or split early into multiple do-files.

> Tip: Move variable creation to setup/merge when possible. If a figure/table-specific do-file must create variables (e.g. reshaping), group that code at the top with comments.


# Part 5: Link do-files with a master script

As scripts mature, a master do-file orchestrates the full pipeline. It is essentially one long script decomposed into modules.

```
01_setup_v4.do  
02_merge_v2.do  
03_tables_v11.do  
04_regressions_v15.do  
05_figures_v2.do  
**06_master_v1.do**
```

The master file might look like:

```
**** <some project info here> ****
clear  
cd <set directory here>
<set some locals and globals here>

// run all the dofiles
do ./dofiles/01_setup_v11.do  
do ./dofiles/02_merge_v2.do  
do ./dofiles/03_tables_v4.do  
do ./dofiles/04_regressions_v5.do  
do ./dofiles/05_figures_v2.do

*** END OF FILE ***
```

Aim to reach a point where this file runs the entire project end-to-end.


# Part 6: Code styling

Emphasize three things: comment your code, split long lines, and use tabs for alignment.

## Commenting

Stata supports three comment styles:

```
* comments on individual lines
// comments on individual lines and after some code
/*   // for marking out code blocks
*/
```

Use them liberally. You can even use stars to visually separate sections:

```
************************** 
**************************  
***                    ***  
*** The Stata Guide    ***  
***    Tutorials       ***  
***                    *** 
**************************  
**************************
```

## Split long lines

Use triple slashes `///` to break long commands, especially graph syntax:

```
twoway (connected y1 x1, msize(vsmall))(connected y2 x2, msize(vsmall)), aspect(1) xlabel(-1(0.5)1) ylabel(-1(0.5)1) xline(0) yline(0)
```

Becomes:

```
twoway ///  
(connected y1 x1, msize(vsmall)) ///  
(connected y2 x2, msize(vsmall)) ///  
, ///  
xlabel(-1(0.5)1) ylabel(-1(0.5)1) ///  
xline(0) yline(0) ///  
aspect(1)
```

## Use tabs for alignment

Tabs help align options for readability:

```
twoway                                 ///  
	(connected y1 x1, msize(vsmall))   ///  
	(connected y2 x2, msize(vsmall))   ///  
		,                              ///  
		xlabel(-1(0.5)1)               ///  
		ylabel(-1(0.5)1)               ///  
			xline(0)                   ///  
			yline(0)                   ///  
			aspect(1)
```

## Comment for sanity

Combine line breaks, alignment, and inline notes to clarify intent:

```
drop if          ///  
_ID==14 | ///    // Puerto Rico  
_ID==28 | ///    // Alaska  
_ID==38 | ///    // American Samoa  
_ID==39 | ///    // United States Virgin Islands  
_ID==43 | ///    // Hawaii  
_ID==45 | ///    // Guam  
_ID==46          // North Mariana Islands
```

Equivalent compact forms:

```
drop if _ID==14 | _ID==28 | _ID==38 | _ID==39 | _ID==43 | _ID==45 | _ID==46
```

or using `inlist`:

```
drop if inlist(_ID, 14, 28, 38, 39, 43, 45, 46)
```

Add brief comments so future-you does not need to reverse engineer IDs.