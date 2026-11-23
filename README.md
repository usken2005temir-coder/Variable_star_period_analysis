# Star Period Tool

A research tool for comparing candidate periods in single-object observation tables.

## Windows quick start

1. Install Python 3.9 or later.
2. Open the StarPeriodTool folder and run INSTALL.bat once.
3. Put your observations in UPLOAD_DATA_HERE.
4. Run RUN.bat and follow the interactive prompts.
5. Review the proposed search ranges, preprocessing plot and final RESULT.html report.

Read StarPeriodTool/START_HERE.md for input formats, methods, uncertainty handling and scientific limitations.

The application is in StarPeriodTool/MainC.py. Available methods: Lomb-Scargle, multiharmonic Fourier regression, PDM and WWZ. You can run one method, compare methods or run additional validation and refinement. No universally best method or guaranteed physical period is claimed.

The supplied UPLOAD_DATA_HERE/1 is an example from the original project with unconfirmed physical interpretation. EXAMPLES/synthetic_ratio.csv is synthetic data, not real stellar observations.

RESULTS is created automatically. Empty .gitkeep files preserve directory structure in Git.
