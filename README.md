# titrationTools
A Wolfram Mathematica package for importing, formatting, and visualizing spectrophotometric binding titration data.
titrationTools provides interactive, automated, and publication-ready plotting functions tailored for analyzing changes in UV-Vis or fluorescence spectra during chemical titrations (e.g., metal-ion binding, host-guest complexation, or ligand titrations).

## Features

- Interactive exploration: Dynamic interfaces to scrub through wavelengths and instantly view titration profiles (absorbance vs. equivalents).
- Automated exporting: Publication-quality multi-plot grids exportable directly to PDF, SVG, or PNG.
- Spectral animation: Quick rendering of animated frames showing spectral shifts as titrant equivalents increase.
- 3D visualization: 3D waterfall plots featuring custom gradient fills and uniform subsampling for dense datasets.
- Flexible ingestion: Designed to process data directly from matrix-style text or Excel formats common for many spectrophotometers (we use a HP 8452A UV-Vis spectrophotometer and Olis software).

## Data layout requirements

The package is optimized to ingest Excel worksheets (typically exported from UV-Vis software). Functions like ```titrationExplorer```, ```titrationPrinter```, and ```spectraExplorer``` expect the following file structure:
- Sheet Name: Defaults to "dilution corrected" because of our internal workflow. Can be adjusted by the user.
- Equivalents row: A row containing the string "equivalents" followed by the numerical values of added titrant equivalents (e.g., 0, 0.25, 0.5, 1.0, ...).
- Data table: A table containing a cell labeled "Wavelength" in the first column. The rows below this header must contain:
- Column 1: Wavelength values (in nm).
- Columns 2+: Absorbance values corresponding to each individual spectrum in the titration sequence.

## Function reference

### 1. ```titrationExplorer```
Imports titration data from an Excel workbook and generates an interactive dual-plot dashboard.

```mathematica
titrationExplorer[inputFile]
titrationExplorer[inputFile, options]
```

- Visual interface:
  - Left plot: A line plot of the full stack of spectra. It features a vertical red line marking the currently selected wavelength. Click and drag the mouse horizontally across the plot to dynamically update the selection. Sliders at the top allow you to crop the wavelength view range.
  - Right plot: A scatter plot showing the titration profile (absorbance vs. equivalents) evaluated at the selected wavelength. 
- Options:
  - "tab" (Default: "dilution corrected"): The target worksheet name within the Excel file.
  - "maxeq" (Default: All): The upper limit of equivalents to display on the profile plot.

### 2. ```titrationPrinter```
Imports an Excel workbook and constructs a formatted static multi-plot grid suitable for printing, lab notebooks, or archiving.

```mathematica
titrationPrinter[inputFile, {λ1, λ2, λ3, λ4}]
titrationPrinter[inputFile, {λ1, λ2, λ3, λ4}, options]
```

- Visual Interface: Generates a 2x3 grid featuring:
  - A prominent bold title showing the file name.
  - A large line plot showing the full stack of spectra (dilution-corrected absorbance vs. wavelength).
  - Four individual titration profiles (absorbance vs. equivalents) displayed as scatter plots for each of the specified wavelengths (λ1 through λ4), with an inset label.
- Options:
  - "tab" (default: "dilution corrected"): The target worksheet name.
  - "minlambda" (default: 260): Lower wavelength boundary for the spectra stack.
  - "maxlambda" (default: 600): Upper wavelength boundary for the spectra stack.
  - "maxeq" (default: ```All```): Maximum equivalents to include on the scatter axes.
  - "output" (default: ```Automatic```): Controls where and how the grid is rendered:
    - Automatic: Displays the plot directly on your screen inside the notebook.
    - "format" (e.g., "PDF", "SVG", "PNG"): Automatically exports a standalone vector or raster file with the same base name and file path as the input data.
    - {"format", exportOptions}: Exports with custom Mathematica Export arguments.
    - {"format", outputPath}: Saves the output file to a specific custom path.
    - {"format", outputPath, exportOptions}: Full configuration for custom path and export parameters.

### 3. ```spectraExplorer```
Creates a self-contained, animated player to step through your titration.

```mathematica
spectraExplorer[inputFile]
spectraExplorer[inputFile, options]
```

- Visual interface: Generates an animated line plot sequence via ```ListAnimate```. Each frame renders an individual spectrum with a red inset label detailing the exact number of added equivalents (e.g., equiv = 1.50). Useful for checking isosbestic points or changing spectral features across a titration.
- Options:
  - "tab" (Default: "dilution corrected"): The target worksheet name.
  - PlotRange (Default: {{260, 800}, {-0.05, 1.4}}): Specifies explicit ```{{xMin, xMax}, {yMin, yMax}}``` boundaries for the animation viewer.

### 4. ```waterfallPlot```
Generates a 3D rendering of a stack of spectra, by stacking them along a depth axis in 3D.

```mathematica
waterfallPlot[data]
waterfallPlot[data, howMany]
waterfallPlot[data, howMany, colors]
waterfallPlot[data, howMany, colors, options]
```

- Arguments:
  - data: A 2D numeric array. Column 1 must contain wavelengths. Subsequent columns contain the absorbance rows for individual spectra. If Row 1 contains sequential sample indices (a standard export pattern), the function automatically detects and discards it.
  - howMany (optional, default: ```All```): An integer specifying how many spectra to uniformly subsample. Useful for preventing visual clutter in large datasets. Subsampled spectra retain their true, uncompressed index locations along the y-axis.
  - colors (optional, default: ```{Blue, Red}```): A single color or a list of colors. If a list is provided, a smooth gradient is blended from the first spectrum to the last.
- Options:
  - ```PlotRange``` (Default: {Full, Full, Full}): Extents for {wavelength, spectrumIndex, absorbance}.
  - ```BoxRatios``` (Default: {1, 4, 1}): Controls the aspect ratio dimensions of the 3D bounding box.
  - Supports any standard option applicable to ```ListLinePlot3D```. Axis labels are omitted by default.

## License

This project is open-source and provided through the Apache 2.0 license.
