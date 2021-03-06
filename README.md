# spikes_detection_fcbg
Three pipelines for automatic epileptic spikes detection

## Pipelines description

All pipelines imply simple preprocessing, such as resampling to 250 Hz and filtering from 2 to 35 Hz.

**Naive pipeline (`naive`):**
Operates only on the channel with the highest variance. Applies standardization to this channel and make classification by variance thresholding.
Yields the highest rate of false positives.

**RECOMMENDED: Simple SVM (`var1_svm`):**
Operates only on the channel with the highest variance. Applies standardization to this channel.
Computes 44 features for each epoch and uses Support Vector Machines for classification.

**Simple AdaBoost (`var1_abdt`):**
Operates only on the channel with the highest variance. Applies standardization to this channel.
Computes 44 features for each epoch and uses AdaBoost for classification.
Yields the best detection of spikes within seizures, however, can lead to increased false positives.

[comment]: <> (_RECOMMENDED_ **TKEO + SVM &#40;`var1_tkeo_svm`&#41;:**)

[comment]: <> (Operates only on the channel with the highest variance. Adds a channel obtained through applying Teager-Kaiser Nonlinear Operator &#40;TKEO&#41; on the channel with the highest variance. )

[comment]: <> (Computes 44 x 2 channels = 88 features for each epoch and uses Support Vector Machines for classification.)

[comment]: <> (**Full pipeline &#40;`full_pipeline_svm`&#41;:**)

[comment]: <> (Aggregates information from all 16 channel using Principal components analysis &#40;PCA&#41;. Adds a channel obtained through applying Teager-Kaiser Nonlinear Operator and PCA. )

[comment]: <> (Computes 44 x 2 channels = 88 features for each epoch and uses Support Vector Machines for classification.)

[comment]: <> (Only applicable for 16 channel EEG recordings. Tested with only one particular montage, PCA maybe montage dependent.)


## Requirements

- [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/windows.html "Conda installation")
- python==3.7

Python libraries:
- [mne](https://mne.tools/stable/install/mne_python.html "MNE") == 0.22.0
- [scikit-learn](https://scikit-learn.org/stable/install.html "Sklearn") == 0.24.1
- [scipy](https://www.scipy.org/install.html "SciPy") == 1.4.1
- [joblib](https://joblib.readthedocs.io/en/latest/installing.html "Joblib") == 0.15.1
- [numpy](https://numpy.org/install/ "NumPy") == 1.18.2
- [imbalanced-learn](https://pypi.org/project/imbalanced-learn/ "Imblearn") == 0.8.0
- [antropy](https://github.com/raphaelvallat/antropy "Antropy") == 0.1.4
- [PyWavelets](https://github.com/PyWavelets/pywt "PyWavelets") == 1.1.1
- [coloredlogs](https://pypi.org/project/coloredlogs/#installation "Colorlogs") == 14.0

[comment]: <> (- pandas == 1.0.3)
To satisfy all the requirements:
1. Install [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/windows.html "Conda installation")
2. Open a Conda Terminal and clone the following repository using command:
```commandline
git clone https://github.com/NataliiaMolch/spikes_detection_fcbg.git
```
3. Finally, go to `spikes_detection/` directory and make a setup with one of two options:

**Automatic setup (Linux):**
```commandline
cd spikes_detection_fcbg/
python setup.py
```

OR

**Manual setup (Linux):**
```commandline
cd spikes_detection_fcbg/
conda env create -f environment.yml
echo 'export PATH=$(pwd):$PATH' >>  ~/.bashrc
source ~/.bashrc
chmod u+x spikes_detection
```

[comment]: <> (An environment with a name `spikes_detection` will be created.)

[comment]: <> (To activate the environment in future run:)

[comment]: <> (```)

[comment]: <> ($ conda activate spikes_detection)

[comment]: <> (```)


## Usage

First, make sure that all the requirements are satisfied (use the instructions above).

Before running program for spikes detection add configurations to `config.ini` file (can be opened with any text editor).
Make sure that MAIN and DATA configurations are up to date.

### Linear interpolation of bad channels

If there are bad channels they need to interpolated for correct work of the algorithms. One can use a build-in 
interpolation of bad channels performed via linear splines. To indicate for each recording what channels should be interpolated
create a "<recording_name>_bads.txt" file where in the first line all the bad channels are listed, separated by 
*a comma and a space*.

For example, for a recording "M9_epi2_LH.sef" with 1st and 13th channels broken create a file "M9_epi2_LH_bads.txt" with content:

```text
e1, e13

```

_Note_ that, "<recording_name>_bads.txt" file should be at the same location as "<recording_name>.sef" file.

### Excluding artifacts and/or seizures

Presence of movement artefacts can negatively affect the quality of predictions. Detection of spikes during seizures is
not perfect with presented algorithms, however, presence of seizures should not affect the quality of predictions.

To exclude seizures and artefacts, create a separate .mrk file at the same location as "<recording_name>.sef" and name it:
"<recording_name>_exclude.mrk". Mark artefacts/seizures beginning and ending with "Art"/"Sz" and "Art.end"/"Sz.end" 
correspondingly.

For example, for a recording "M9_epi2_LH.sef" create a "M9_epi2_LH_exclude.mrk" file with possible content:

```text
TL02
2020000	2020000	Art
2056000	2056000	Art.end
2104000	2104000	Art
2112000	2112000	Art.end
2184000	2184000	Sz
2496000	2496000	Sz.end
...
```

### Program execution

Use terminal to run the program by either ways (prior modification of configuration file is needed):

**Easy run:**
```commandline
 cd spikes_detection_fcbg/
 spikes_detection
```
_Note_ `spikes_detection` cmd has two optional options: `[-p] [-e]`
Option `-p`: full path to configuration file with .ini extension.
If `-p` option is not given, then searches for config.ini file in current working directory.
Option `-e`: the name of the conda environment.
If `-e` option is not given, then uses spikes_detection conda environment by default.

OR 

**The other way**
```commandline
cd spikes_detection_fcbg/
conda activate spikes_detection
python make_predictions.py
```

_Note_ `make_predictions.py` has one argument `[--config_file]` - path to configuration file.
If not given, searches for `config.ini` file in current working directory.