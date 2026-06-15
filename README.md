# titrationTools

A toolkit for importing, analyzing, and visualizing spectrophotometric binding titration data.

titrationTools provides two complementary modules targeting different stages of a typical titration workflow:

- **titrationTools.m** — A Wolfram Mathematica package for interactive exploration, formatted multi-plot grids, spectral animations, and 3D waterfall plots of UV-Vis or fluorescence titration series.
- **mcr_tools.py** — A Python module wrapping SpectroChemPy's MCR-ALS engine for multicomponent spectral resolution of titration datasets, including PCA rank analysis and diagnostic visualization.

Both modules are designed to ingest the same matrix-style CSV or Excel data layouts common to UV-Vis spectrophotometers (developed and tested with an HP 8452A / Olis software workflow).

---

## Module 1: titrationTools.m (Wolfram Mathematica)

### Features

- **Interactive exploration:** Dynamic dual-plot dashboard to scrub through wavelengths and view titration profiles in real time.
- **Automated exporting:** Publication-quality multi-plot grids exportable to PDF, SVG, or PNG.
- **Spectral animation:** Animated frame sequences showing spectral shifts as titrant equivalents increase.
- **3D visualization:** 3D waterfall plots with custom gradient fills and uniform subsampling for dense datasets.
- **Flexible ingestion:** Reads matrix-style Excel or text exports from common UV-Vis software.

### Data layout requirements

Functions expect an Excel workbook with the following structure:

- **Sheet name:** Defaults to `"dilution corrected"` (adjustable).
- **Equivalents row:** A row containing the string `"equivalents"` followed by numerical titrant equivalent values.
- **Data table:** First column labeled `"Wavelength"` with wavelength values (nm) below; subsequent columns contain absorbance values for each spectrum in the titration sequence.

### Function reference

#### `titrationExplorer`

Imports titration data and generates an interactive dual-plot dashboard (spectra stack + titration profile at a selected wavelength).

```mathematica
titrationExplorer[inputFile]
titrationExplorer[inputFile, options]
```

Options: `"tab"` (default: `"dilution corrected"`), `"maxeq"` (default: `All`).

#### `titrationPrinter`

Generates a formatted 2x3 static multi-plot grid (full spectra stack + four single-wavelength titration profiles) for printing or archiving.

```mathematica
titrationPrinter[inputFile, {λ1, λ2, λ3, λ4}]
titrationPrinter[inputFile, {λ1, λ2, λ3, λ4}, options]
```

Options: `"tab"`, `"minlambda"`, `"maxlambda"`, `"maxeq"`, `"output"`.

#### `spectraExplorer`

Generates an animated player stepping through individual spectra with equivalents labels.

```mathematica
spectraExplorer[inputFile]
spectraExplorer[inputFile, options]
```

Options: `"tab"`, `PlotRange`.

#### `waterfallPlot`

Renders a 3D waterfall plot of a spectral stack with gradient coloring and optional uniform subsampling.

```mathematica
waterfallPlot[data]
waterfallPlot[data, howMany, colors, options]
```

---

## Module 2: mcr_tools.py (Python / SpectroChemPy)

`mcr_tools.py` is a front end to the [SpectroChemPy](https://www.spectrochempy.fr/) library's MCR-ALS implementation. It handles data packaging, PCA rank analysis, and full MCR-ALS optimization with diagnostic plots suitable for publication.

### Dependencies

- [SpectroChemPy](https://www.spectrochempy.fr/)
- NumPy, pandas, Matplotlib

### Features

- **`loadAndPackageData`:** Reads CSV or Excel files into a SpectroChemPy `NDDataset` object with labeled spectral and concentration axes. The object supports wavelength and concentration range slicing, flexible spectra type aliases (absorbance, fluorescence/intensity).
- **`runPCA`:** Runs Principal Component Analysis on the spectra stack and plots a logarithmic scree plot of contributions for the first few spectral components, preliminary to running MCR-ALS.
- **`runMCR`:** Configures and runs the multicurve resolution - alternating least squares (MCR-ALS) algorithm as implemented in SpectroChemPy, with the following constraints available:
  - Non-negativity on all concentrations and spectra (NNLS solver)
  - Monotonic concentration decrease on the initial species, monotonic increase on the final species
  - Unimodality (rise to max and return to baseline) on the concentration profile of any intermediate components
  - Hard spectral constraint (for spectra of known species)
  - Optional hard concentration constraint enforcing a pure initial species at the start of the titration
  - Generates four diagnostic plots: resolved spectra, concentration profiles (linear and log x-axis), fit quality / residuals, and initial/final spectrum comparisons.

### Quick start

```python
import mcr_tools as mcr

# Load data
dataset = mcr.loadAndPackageData(
    "my_titration.xlsx",
    concList=[0, 0.1, 0.25, 0.5, 1.0, 2.0, 5.0],
    titrantName="Cu2+",
    concUnit="mM",
    wlStart=300, wlEnd=700
)

# Check number of components with PCA
pcaModel = mcr.runPCA(dataset, numPCAcomponents=5, plotComponents=True)

# Run MCR-ALS (2-component system: guesses from first and last spectrum)
mcrModel, St, C = mcr.runMCR(dataset, guessIndices=[0, -1])
```

### Citation

If you use `mcr_tools.py`, please also cite SpectroChemPy:

> Travert Arnaud, Fernandez Christian (2026). *SpectroChemPy, a framework for processing, analyzing and modeling spectroscopic data for chemistry with Python* (version 0.9.4.dev74). Zenodo. DOI: [10.5281/zenodo.3823841](https://doi.org/10.5281/zenodo.3823841)

---

## License

This project is open-source under the [Apache 2.0 License](LICENSE).

## How to cite

If you use titrationTools in your research, please cite this repository (see `citation.cff` or the "Cite this repository" button on GitHub). A Zenodo DOI is available for each versioned release.
