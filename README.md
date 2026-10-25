# Changelog

All notable changes to the RFF feature computation notebook are recorded here.

Baseline: `000_rff-data-calculate_20261005Smorffi原始代码备份.ipynb`  
Release: `RFF_Data_Calculate.ipynb`

## v2.0.0, 2026-10-25

### Changed

**1. Residual subtraction removed.**  
The baseline subtracted the ideal L-LTF symbol from the equalised symbols, so every downstream quantity described a residual. v2.0 keeps the equalised symbols themselves.

**2. Magnitude error sign restored.**  
The definition is now `mag = |Y| - |L|`.  
The baseline used the opposite order.  
A global sign flip does not change tree behaviour, and the definition now matches Section 2.1 of the paper.

**3. Phase error rewritten as a complex ratio.**  
The definition is now `phase = angle(Y * conj(L))`.  
The baseline subtracted two angles. That difference wraps outside the interval (-pi, pi].  
The share of subcarriers that wrap drops from 15.8% to 0.

**4. Statistics restricted to the 52 active subcarriers.**  
The baseline averaged over 64 bins, and 12 of those bins carry zeros. v2.0 adds an active-bin mask and computes the mean and variance on the active set.  
The stored vectors keep 64 points.

**5. Fractal dimension moved to the time domain.**  
The baseline ran Higuchi's method on the two equalised residuals.  
The two results were identical, so the duplicate column was dropped. v2.0 runs the method on the raw 288-sample preamble and stores a single column named `frac_dimension`.

**6. Duplicate FFT removed.**  
The baseline recomputed the FFT of both L-LTF halves just before the channel estimate. v2.0 reuses `k1` and `k2`.

**7. Dead comparison removed.**  
`np.where(Y1 == None, 0, Y1)` never fires on a complex array.  
The masked assignment above it already handles the missing frames.

**8. `iq_preamble` keeps its name and changes its content.**  
The column now stores the equalised L-LTF symbols as 128 complex values.  
The baseline stored residuals.  
Values sit near ±1 with a 1% to 5% deviation, and the 12 null bins read zero.  
The second half equals `2L - first half`, an identity that follows from the per-subcarrier channel estimate.

### Added

**9. Feature display cell.**  
Four panels: CFO distribution, phase error distribution, amplitude waveform, and constellation.  
Ported from the tutorial notebook.  
The cell reads the feature directory and writes PNG files.

### Data

**10. Schema.**  
The column count drops from 23 to 22.  
`frac_dimension_2` is gone.  
`iq_preamble` keeps its position after `preamble`.

### Results

Same 123 devices, 117903 valid frames, same 80/20 split, same forest settings.

- Without Kalman filtering: 82.17% to 88.64%
- With Kalman filtering: 88.04% to 94.71%
- Column pairs with |r| >= 0.999: 4 to 0
- Fractal dimension importance rank: 14 and 15 to 3

### Known issues

- `78_21_84_93_59_44_pre.csv` has a malformed header in the raw data. The header was repaired in the published CSVs.
