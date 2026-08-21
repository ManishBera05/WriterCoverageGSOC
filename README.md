# Google Summer of Code 2026 Final Report
## Project: Improve Word Processor Test Coverage
**Organization:** [LibreOffice / The Document Foundation](https://www.libreoffice.org/)  
**Contributor:** [Manish Bera](https://github.com/ManishBera05) (GSoC 2025, 2026) <br>
**Mentor:**  Jonathan Clark

---

## 📑 Table of Contents

* [1. Project Overview & Background](#1-project-overview--background)
  * [1.1 The Objective: Metric-Driven Behavior Locking](#the-objective-metric-driven-behavior-locking)
* [2. Methodology & Workflow](#2-methodology--workflow)
* [3. Quantifiable Results & Metrics](#3-quantifiable-results--metrics)
  * [3.1 Primary Focus Module: `sw/source/core/text/`](#31-primary-focus-module-swsourcecoretext)
  * [3.2 Overall Top-Level Writer Core Impact (`sw/source/core/`)](#32-overall-top-level-writer-core-impact-swsourcecore)
  * [3.3 Coverage Dashboards](#33-coverage-dashboards)
* [4. Key Technical Accomplishments & Tested Features](#4-key-technical-accomplishments--tested-features)
* [5. List of Commits & Merged Patches](#5-list-of-commits--merged-patches)
* [6. Future Work & Opportunities](#6-future-work--opportunities)
* [7. Acknowledgments](#7-acknowledgments)

---

## 1. Project Overview & Background

LibreOffice Writer is one of the most widely used and feature-rich open-source word processing applications in the world. With a development history spanning nearly four decades, Writer has accumulated a massive codebase supporting complex multilingual typography, intricate page formatting, and comprehensive interoperability with proprietary formats (DOCX, RTF).

However, managing and modernizing this legacy code presents a fundamental challenge: Writer’s internal layout and text formatting engine (`sw/source/core/text/`) contains some of the oldest and most intricate C++ code in the project. Because of this complexity, modifying or refactoring this code historically carried a high risk of accidental regressions.

### The Objective: Metric-Driven Behavior Locking
The goal of this project was **not** bug hunting. Instead, the mission was to analytically improve the automated test coverage of Writer's core text engine. 

By utilizing code coverage metrics (`gcov` and `lcov`) to identify untested C++ execution branches, this project reverse-engineered edge-case document states and created a comprehensive suite of automated `CppUnit` tests using `parseLayoutDump()` and VCL PDF export verification. This suite acts as a permanent safety net, mathematically locking in the current, correct behavior so core developers can safely refactor and optimize Writer's text engine for years to come.

---

## 2. Methodology & Workflow

The project adopted a continuous, 5-stage metric-driven development cycle:

```
[ LCOV Analysis ] ➔ [ Reverse-Engineer Document ] ➔ [ CppUnit Assertions ] ➔ [ LCOV Verification ] ➔ [ Merge ]
```

1. **Analytical Profiling (`lcov`):** Generated line-by-line HTML coverage dashboards to locate "cold" (0% coverage) C++ execution paths in `sw/source/core/text/`.
2. **Reverse-Engineering State:** Deduced the exact combination of styles, margins, typography rules, or frame intersections required to force the layout engine down the untested C++ branches.
3. **Document Engineering (`.fodt`):** Crafted minimal Flat XML ODF (`.fodt`) test documents, stripped of unnecessary XML bloat using `bin/flat-odf-cleanup.py` to keep tests lightweight and human-readable.
4. **Automated Assertions:** Wrote `CppUnit` tests using `parseLayoutDump()` and XPath queries (`assertXPath`) to assert exact layout geometries, portion types, and formatting states.
5. **Dual-Pipeline Testing (Layout vs. Paint):** Validated both internal layout box calculations via `sw_layoutwriter3` and end-to-end VCL drawing/glyph rendering via `vcl_pdfexport2`.

---

## 3. Quantifiable Results & Metrics

Throughout the summer, profiling was tracked both across the specific target module (`sw/source/core/text/`) and at the overall top-level Writer Core engine (`sw/source/core/`, spanning over 230,000 lines of C++ code).

### 3.1 Primary Focus Module: `sw/source/core/text/`

| Metric | Baseline (May 26, 2026) | Final (August 21, 2026) | Net Improvement |
| :--- | :--- | :--- | :--- |
| **Line Coverage** | **79.7%** (19,862 lines) | **83.6%** (20,840 lines) | **+978 lines (+3.9%)** |
| **Function Coverage** | **87.4%** (1,538 functions) | **89.8%** (1,581 functions) | **+43 functions (+2.4%)** |


### 3.2 Overall Top-Level Writer Core Impact (`sw/source/core/`)

| Metric | Baseline (May 26, 2026) | Final (August 21, 2026) | Net Improvement |
| :--- | :--- | :--- | :--- |
| **Total Lines Covered** | **72.0%** (167,441 / 232,705) | **72.5%** (168,683 / 232,705) | **+1,242 lines (+0.5%)** |
| **Total Functions Covered** | **76.3%** (67,765 / 88,818) | **76.7%** (68,082 / 88,818) | **+317 functions (+0.4%)** |

---

### 3.3 Coverage Dashboards

#### Baseline Coverage Report (May 2026):
<!-- PLACEHOLDER FOR BEFORE SCREENSHOT -->
![Baseline LCOV Coverage Report](https://github.com/ManishBera05/WriterCoverageGSOC/blob/main/before.png)
*Figure 1: Baseline LCOV report for `sw/source/core/text` showing 79.7% Line Coverage.*

#### Final Coverage Report (August 2026):
<!-- PLACEHOLDER FOR AFTER SCREENSHOT -->
![Final LCOV Coverage Report](https://github.com/ManishBera05/WriterCoverageGSOC/blob/main/after.png)
*Figure 2: Final LCOV report for `sw/source/core/text` showing 83.6% Line Coverage (+978 lines newly covered).*

---

## 4. Key Technical Accomplishments & Tested Features

Throughout the summer, tests were written across multiple core C++ source files, locking down previously untested behaviors:

1. **Drop Caps (`txtdrop.cxx`):**
   * Locked in the `lcl_IsDropFlyInter` safety mechanism, which prevents layout corruption when an anchored floating object (Fly frame) intersects with a multi-line Drop Cap.
   * Covered word boundary splitting for `ScriptType::ASIAN` (CJK) and `ScriptType::COMPLEX` (CTL/Bengali) scripts.
   * Tested vertical text typography calculations for right-to-left Asian text frames.
2. **Case Mapping & Ligatures (`fntcap.cxx`):**
   * Covered `sw_CalcCaseMap` and `bCaseMapLengthDiffers` by testing string length expansion during line breaking when small caps / capital transformations expand Unicode ligatures (e.g., `ﬄ` expanding from 1 character to 3 characters `FFL`).
3. **Multi-Portions & Warichu (`pormulti.cxx` & `itrform2.cxx`):**
   * Covered the `UpdatePos` coordinate shifting loop when an Asian Ruby text multi-portion resides in the same paragraph as an As-Character anchored image (`FlyInContent`).
   * Covered `SwDoubleLinePortion` (Two-Lines-in-One / Warichu) bracket calculations.
4. **Text Adjustment & Justification (`itradj.cxx`):**
   * Covered `SwTextAdjuster::CalcDropAdjust` for Drop Caps residing in Centered or Right-aligned paragraphs.
   * Covered `pMulti->HasTabulator()` and `IsDouble()` space-counting routines when Warichu blocks reside in fully justified lines.
   * Tested `CalcFlyAdjust` when a centered paragraph containing a tabbed multi-portion intersects an anchored shape.
5. **Portion Expansion & Fields (`porexp.cxx` & `porfld.cxx`):**
   * Tested zero-width `SwExpandPortion::Format` fallback calculations for active Hidden Text fields and empty user variables.
   * Covered `SwCombinedPortion` (Kumimoji) font-width scaling calculations when combining more than 4 characters.
6. **VCL PDF Export Rendering (`pdfexport2.cxx`):**
   * Built end-to-end PDF export tests to exercise the VCL Paint engine, executing previously unreachable drawing routines.

---

## 5. List of Commits & Merged Patches

* [**Patch 205858**](https://gerrit.libreoffice.org/c/core/+/205858): `sw: add layout test for drop cap intersecting with fly frame`
* [**Patch 206195**](https://gerrit.libreoffice.org/c/core/+/206195): `sw: add layout tests for drop caps with Asian and Complex scripts`
* [**Patch 207361**](https://gerrit.libreoffice.org/c/core/+/207361): `sw: add layout test for drop caps in vertical text frames`
* [**Patch 207479**](https://gerrit.libreoffice.org/c/core/+/207479): `sw: add layout test for soft hyphen line-breaking around fly frames`
* [**Patch 208012**](https://gerrit.libreoffice.org/c/core/+/208012): `sw: add layout test for small caps ligature length expansion in fntcap.cxx`
* [**Patch 208254**](https://gerrit.libreoffice.org/c/core/+/208254): `sw: add layout test for title case word boundary line breaking`
* [**Patch 208638**](https://gerrit.libreoffice.org/c/core/+/208638): `sw: add layout test for SwMultiPortion with FlyInContent in itrform2.cxx`
* [**Patch 208875**](https://gerrit.libreoffice.org/c/core/+/208875): `sw: add layout test for SwDoubleLinePortion bracket calculations`
* [**Patch 208876**](https://gerrit.libreoffice.org/c/core/+/208876): `sw: add layout test for justified paragraph spacing with Warichu blocks`
* [**Patch 209023**](https://gerrit.libreoffice.org/c/core/+/209023): `sw: add layout test for centered text fly frame adjustment with tabbed multi-portions`
* [**Patch 209044**](https://gerrit.libreoffice.org/c/core/+/209044): `sw: add layout test for empty field SwExpandPortion zero-width formatting`
* [**Patch 209132**](https://gerrit.libreoffice.org/c/core/+/209132): `sw: add layout test for SwCombinedPortion font width scaling (>4 characters)`
* [**Patch 208901**](https://gerrit.libreoffice.org/c/core/+/208901): `vcl: add PDF export rendering test for drop cap painting verification`

---

## 6. Future Work & Opportunities

While this project successfully brought the testable layout coverage of `sw/source/core/text/` above 83%, the same metric-driven methodology can and should be applied to neighboring modules in the future:

1. **Document Model (`sw/source/core/txtnode/`):**
   * Expanding tests in `ndtxt.cxx` (Text Node lifecycle), `thints.cxx` (Text Attribute Hint management), and `attrlinebreak.cxx` using `UNO API` model tests (`sw/qa/extras/unoapi/`).
2. **Table & Frame Layout (`sw/source/core/layout/`):**
   * Applying `parseLayoutDump()` assertions to `tabfrm.cxx` (Table Frames), `rowfrm.cxx` (Row Calculations), and `flyfrm.cxx` to lock down complex nested tables, multi-column section breaks, and floating frame overlaps.
3. **End-to-End Rendering Tests:**
   * Expanding the `vcl/qa/cppunit/pdfexport` test suite to cover remaining VCL painting routines (such as graphic bullet animations and complex text layout font shaping).

---

## 7. Acknowledgments

I would like to express my deepest gratitude to my mentor, **Jonathan Clark**, for his invaluable guidance, clear technical direction, and prompt code reviews throughout the project and the LibreOffice community for entrusting me with the project. 

***
