# STAR PERIOD TOOL - folder workflow

## Start here

1. Install Python 3.9 or later.
2. Double-click INSTALL.bat once to create a local environment and install dependencies.
3. Put your observation table in UPLOAD_DATA_HERE. Each table should describe one star and one photometric band. Do not paste data into the code.
4. Double-click RUN.bat (or run `python MainC.py` from a configured environment).
5. Choose the input file by its number. Enter an object name, signal type and input time unit. After choosing the analysis mode, review the proposed period ranges and select one or enter custom bounds in DAYS. If asked for column names, press Enter for automatic detection or enter the exact header.
6. Choose clipping: 3 means the default conservative 3-sigma residual pass; 0 disables clipping. Choose full analysis, four-method comparison, one selected method, or preprocessing only.
7. The preprocessing page opens first. Review retained and flagged points, return to the console and enter y to continue or n to stop. After analysis, RESULT.html opens with the period table and plots. Each run is saved in its own RESULTS/run_... folder.

The original file is never overwritten. Previous results are retained. UPLOAD_DATA_HERE may contain multiple files; the interactive menu lets you choose one. Files are NOT concatenated or treated as one star automatically.

## Folder structure

- UPLOAD_DATA_HERE/: put your observations here. The supplied file `1` is copied unchanged from the uploaded archive. Its signal type and scientific provenance are NOT inferred from its name or numeric values.
- MainC.py: the single complete application, including all four methods, comparison, refinement and uncertainty handling. Do not run separate old LS/PDM/wavelet/sigma-clipping scripts.
- RESULTS/: each run has its own results and preprocessing audit.
- EXAMPLES/synthetic_ratio.csv: synthetic V/R example with known period 0.7 days; not real stellar observations.
- requirements.txt: external scientific libraries. MainC.py embeds all application modules.
- INSTALL.bat / RUN.bat: Windows installation and launch buttons.

## Accepted observations

CSV, TSV, whitespace text, .dat and tabular FITS are accepted. Files without extensions are accepted as text. Headerless two-column tables are read as `time signal`; headerless three-column tables as `time signal error`. The FIRST row is retained. More than three headerless columns require explicit headers. Time must be numeric, decimal separators must be points, and errors must be standard uncertainties in the signal's units.

Example with headers:

```csv
time,mag,mag_err
2459000.123,12.31,0.02
2459000.145,12.28,0.02
2459000.178,12.24,0.03
```

Three rows only illustrate the format. At least 12 valid observations and eight distinct times are required; full validation usually needs many more. Omit unknown errors, do not fill them with zeros. The type can be magnitude, flux, vr, other or auto. For headerless data auto cannot determine the physical meaning and uses a generic signal. The supplied UPLOAD_DATA_HERE/1 has no headers or error column: time and signal are detected, but you must establish what that signal represents.

The program does not download observations from a star name, calibrate data, perform barycentric correction or combine bands. Input time defaults to days; --time-unit supports seconds/minutes/hours. Search bounds always use days and must be chosen for the scientific question. There is no universally suitable range.

## Processing sequence

1. Read, sort and validate observations. Save invalid/excluded rows and a copy of the input.
2. If clipping is enabled, perform preliminary full-range searches and fit candidate harmonic predictors. This is necessary to distinguish deviations from a variable signal from real extrema.
3. Apply ONE conservative residual-clipping pass at the chosen sigma threshold. A point must be flagged by all supported candidate predictors. The residual scale is based on median/MAD and supplied errors. Clipping is skipped if it would remove more than 10% or leave too few observations. This is not guaranteed identification of instrumental outliers.
4. Run LS, multiharmonic Fourier regression, PDM and WWZ on the retained observations using one shared frequency grid.
5. In full mode, compare methods by blocked prediction and injection/recovery; select harmonic complexity and refine the joint model; estimate conditional period dispersion using residual blocks.

IMPORTANT: recommendation and joint-model validation use ORIGINAL valid observations BEFORE optional clipping, preserving our earlier validation protocol. Base-method results use the retained data. This distinction is visible in START_RESULT.md; cleaned and original results must not be described as the same experiment. The old automatic flux-to-magnitude conversion and unlimited rolling-median clipping are not used.

## Results to open

- START_RESULT.md: unified overview, clipping counts, method candidates, conditional recommendation and joint-model status.
- PREPROCESSING.md: preprocessing settings, source preservation and time units.
- valid_data_before_clipping.csv: valid observations before clipping.
- clean_data.csv: retained observations, with original numeric time units restored.
- outliers.csv: observations flagged by the residual procedure, not proven bad measurements.
- sigma_clipping_result.png: retained and flagged observations.
- excluded_rows.csv: invalid numeric values or nonpositive errors.
- source_input.*: unchanged source snapshot, or source_input for an extensionless original.
- report.md / comparison.csv: base-method comparison on retained observations.
- phase_curves.png / periodograms.png: phased observations, candidate peaks and spectral window.
- recommendation.md: conditional comparison; comparable or inconclusive are valid outcomes.
- advanced.md / joint_periods.csv / joint_fit.png: refined model and conditional uncertainty.
- error_audit.csv / run.json / tool_manifest.json: error provenance, full settings and source checksums.

The exported preprocessing tables use `time`, `signal`, optional `error`, and `Signal Label`. Their `time` is in the original input units; see PREPROCESSING.md. Other internal tables use days from the saved epoch. For component-based V/R propagation on a later run, use the original table with V/R components, not the reduced clean export.

## Errors, V/R and interpretation

Supplied errors enter LS/Fourier/joint inverse-variance fits. PDM is unweighted; WWZ uses temporal weights. With --bootstrap N, every selected base method repeats searches on perturbed data. Without errors, fits have equal weights; full mode can use residual blocks without inventing measurement errors. Explicit --noise-estimate difference estimates common noise from adjacent differences, excluding large gaps. This can include real variability and is not an instrumental error.

For V/R, use --signal-kind vr. A period describes spectral intensity-ratio variability, not necessarily pulsation. Supplied ratio errors are accepted, or propagate component errors using --vr-v-col V --vr-r-col R --vr-v-error-col eV --vr-r-error-col eR --vr-error-correlation 0. First-order propagation includes covariance; rho=0 is an assumption. Large relative errors or shared normalization need care. Do not combine ready ratio errors and component propagation.

Full defaults: four methods, residual clipping at 3 sigma, automatic joint-model harmonic count 1..6, at most one independent periodic component, conditional method comparison (12 injections), 100 joint uncertainty repetitions and block noise. Full runs can take minutes or longer. Basic mode retains clipping and the four-method comparison but disables automatic advanced/recommendation additions. Choose clipping 0 for no clipping. No method is forced to be the best; aliasing and insufficient information can remain unresolved. Conditional intervals are not calibrated coverage guarantees. A fine grid is not a small period uncertainty.

## Command-line examples

No path is needed when UPLOAD_DATA_HERE contains exactly one supported observation file:

```powershell
python MainC.py --signal-kind other --min-period 10 --max-period 500
```

Choose a specific file, run the base comparison with 3-sigma clipping:

```powershell
python MainC.py --input UPLOAD_DATA_HERE/my_star.csv --time-col time --signal-col mag --signal-kind magnitude --min-period 0.02 --max-period 0.2 --basic
```

Disable clipping, or change its threshold:

```powershell
python MainC.py --input UPLOAD_DATA_HERE/my_star.csv --min-period 0.02 --max-period 0.2 --no-clip
python MainC.py --input UPLOAD_DATA_HERE/my_star.csv --min-period 0.02 --max-period 0.2 --clip-sigma 4
```

Use --help for scientific options; --manual prints this guide; --self-test runs a short synthetic check. --validation-input accepts independent compatible observations without refitting. --max-modes can be increased for multiperiodic candidate models, but recovering all physical modes is not guaranteed.

If INSTALL.bat reports that `py` is missing, install Python first. If installation fails, read the error before rerunning. A manual installation is:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe MainC.py
```

## Scientific references

Astropy LS: https://docs.astropy.org/en/stable/timeseries/lombscargle.html
Stellingwerf (1978), PDM: https://doi.org/10.1086/156444
Foster (1996), WWZ: https://doi.org/10.1086/118137
SciPy least squares: https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.least_squares.html
Uncertainty propagation: https://www.nist.gov/pml/nist-technical-note-1297/nist-tn-1297-appendix-law-propagation-uncertainty

This is a research tool and software deliverable, not a completed thesis manuscript or proof of universal period accuracy.

## Visual stages and separate methods

The interactive menu offers seven modes: full validation; four-method comparison; LS only; Fourier only; PDM only; WWZ only; preprocessing only. Preprocessing is exported BEFORE the final searches. PREPROCESSING.html remains available if you stop. RESULT.html contains the completed numerical results and plots. The browser displays local reports; observations are not uploaded.

Command-line options:

- `--step-by-step`: pause after the preprocessing report (interactive terminal required).
- `--open-report`: open the local HTML reports in the default browser.
- `--preprocess-only`: save preprocessing and stop.
- `--basic --methods ls`: run only LS after preprocessing. Replace ls with fourier, pdm or wwz.
- `--no-clip`: disable clipping, including for an already cleaned table.

Example:

```powershell
python MainC.py --input UPLOAD_DATA_HERE/star.csv --min-period 0.02 --max-period 0.2 --basic --methods pdm --step-by-step --open-report
```

Separate-method mode selects the final method. Residual clipping can still use preliminary candidate searches from other methods to protect real variability. To run strictly one method without this preprocessing search, use `--basic --methods pdm --no-clip`.

The existing scientific protocol is preserved: the base methods analyze retained data; full-mode recommendation and joint-model validation analyze original valid data. Both scopes are labelled in the report. A conditional preferred method is not guaranteed, and is not a ranking of the cleaned-data estimates.

## Choosing a period range without knowing the period

The interactive wizard loads and validates the observations before asking for search bounds. It shows the baseline, positive time gaps, three sampling-based search proposals, a plot and the estimated frequency-grid size in RANGE_ADVICE.html. Exposure duration in seconds is optional; leave it blank if unknown. Advice uses valid observations before clipping, one object and one band.

1. Sampling-based start: 5 times the median positive gap to one third of the baseline.
2. Extended search: twice the 10th percentile of positive gaps to half the baseline.
3. Long-period exploration: one third of the baseline to the baseline (only 1 to 3 cycles).
4. Manual bounds: use physical expectations or independently published constraints when available.

These numerical factors are transparent interface heuristics, not established optimal bounds, a strict Nyquist limit or a preliminary period measurement. With a known exposure duration, the first two lower bounds are at least twice that duration; shorter signals can be attenuated by integration, and this convention does not guarantee accurate recovery. Uneven sampling can support periods shorter than the proposed bounds. Sparse data, gaps and aliases may make any range inconclusive. No star class is inferred from the filename.

An unavailable range is labelled as such (insufficient baseline or grid limit). The application never silently coarsens the grid or truncates the requested range. A large grid is an estimate of frequency count, not a runtime guarantee. CLI users can deliberately change --max-grid-size or choose a narrower interval. Long baselines with rapid measurements can be expensive.

The chosen bounds, formulas, sampling statistics, input time unit and input checksum are saved in range_advice.json and copied into the analysis results. Reports link to the advice. Periods outside the chosen range remain untested; review boundary peaks and expand the range when scientifically justified. Explicit CLI --min-period/--max-period usage is unchanged.

Reference: https://docs.astropy.org/en/stable/api/astropy.timeseries.LombScargle.html (baseline and irregular-sampling caveats).
