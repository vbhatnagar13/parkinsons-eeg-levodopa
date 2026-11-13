# Resting-State EEG in Parkinson's Disease: Beta and Theta Power ON vs OFF Levodopa

**Author:** Vibhor Bhatnagar · Python, MNE-Python, SciPy, statsmodels

## Why I did this

[Write 2–3 sentences in your own words: what drew you to this question and to neurotech. This section is yours; delete this line once you've written it.]

## Question

Parkinson's disease is associated with abnormal beta-band (about 13–30 Hz) activity in motor circuits, and dopaminergic medication (levodopa) is reported to reduce it in recordings from deep electrodes. Does a similar effect show up at the scalp? Specifically, in resting-state EEG over central scalp channels:

1. Does beta power change in the same patients ON vs OFF levodopa?
2. Does beta power differ between patients and healthy controls?

The planned analysis covered beta only. A theta-range comparison was added afterward as an **exploratory** follow-up (see Results).

## Data

[OpenNeuro ds002778](https://openneuro.org/datasets/ds002778): resting-state EEG collected at UC San Diego and curated by Alex Rockhill (University of Oregon).

- 15 Parkinson's patients, each recorded twice: OFF medication (at least 12 hours after stopping) and ON medication
- 16 age-matched healthy controls, one session each
- 512 Hz sampling, recordings of a few minutes each (e.g., 200 s OFF and 213 s ON for `sub-pd3`); 32 scalp electrodes used (the 8 external `EXG` channels and the trigger channel were dropped)
- Note from the dataset documentation: for `sub-pd6` and `sub-pd16`, the ON session used data converted from preprocessed files instead of raw data. I flagged them and repeated the main analysis without them.

The dataset's documentation advises against training a classifier to separate patients from controls (fewer than 8 patients would end up in a test set), so this project uses group-level statistics instead of machine learning.

## Methods

**Preprocessing** (identical for every recording)
- Kept the 32 scalp channels, re-referenced to the average of all channels
- Band-pass filtered 1–45 Hz (zero-phase FIR)
- No ICA / blink removal. Blinks are slow and frontal (checked visually in the raw traces), and the measurement is over central channels `C3`, `Cz`, `C4`; the 1 Hz high-pass already removes most slow activity.

**Features**
- Welch power spectrum (2 s windows), averaged over `C3`, `Cz`, `C4`
- Relative band power = power in the band ÷ total 1–45 Hz power. Relative power reduces differences in overall signal strength between people.
- Planned bands, fixed before looking at group results: low beta 13–20 Hz and high beta 20–30 Hz

**Statistics**
- ON vs OFF (same patients): Wilcoxon signed-rank test; effect size = mean difference ÷ SD of the differences
- Patients vs controls: Mann–Whitney U test; effect size = Cohen's d
- 95% confidence intervals from 10,000 bootstrap resamples
- Multiple comparisons: Benjamini–Hochberg false discovery rate (FDR)

## Results

### Planned analysis: beta power (null result)

Relative power over `C3`, `Cz`, `C4`. Mean difference is the first group minus the second.

| Comparison | Mean diff | 95% CI | Effect size | p (FDR) |
|---|---|---|---|---|
| Low beta, ON − OFF | +0.010 | −0.006 to +0.023 | 0.33 | 0.38 |
| High beta, ON − OFF | +0.011 | −0.008 to +0.027 | 0.31 | 0.46 |
| Low beta, PD OFF − controls | +0.009 | −0.024 to +0.044 | 0.18 | 0.74 |
| High beta, PD OFF − controls | −0.025 | −0.077 to +0.023 | −0.34 | 0.59 |

No beta comparison was significant after correction, and every confidence interval includes zero. Excluding `sub-pd6` and `sub-pd16` gave the same conclusion. Individual differences between people were much larger than the ON/OFF change (see the paired plot below).

![Per-patient OFF vs ON](figures/paired_beta.png)

### Exploratory analysis: theta power

After plotting the group-average spectra, the patient curves appeared higher than controls around 6–8 Hz. I then tested two low-frequency windows, standard theta (4–8 Hz) and the 6–8 Hz range where I saw the gap, with the same methods. **The windows were chosen after viewing the figure, so these results are exploratory and the p-values are optimistic.**

| Band | Comparison | Mean diff | 95% CI | Effect size | p (FDR) |
|---|---|---|---|---|---|
| 4–8 Hz | ON − OFF (paired) | +0.022 | +0.009 to +0.034 | 0.87 | 0.013 |
| 6–8 Hz | ON − OFF (paired) | +0.012 | +0.005 to +0.019 | 0.77 | 0.013 |
| 6–8 Hz | PD ON − controls | +0.037 | +0.011 to +0.072 | 0.83 | 0.013 |
| 4–8 Hz | PD ON − controls | +0.045 | +0.008 to +0.092 | 0.72 | 0.055 |
| 6–8 Hz | PD OFF − controls | +0.025 | +0.001 to +0.062 | 0.57 | 0.052 |
| 4–8 Hz | PD OFF − controls | +0.023 | −0.014 to +0.066 | 0.39 | 0.26 |

The FDR correction above covers these six tests. If all ten tests in the project (beta and theta) are corrected together, the three clearest effects (both ON − OFF comparisons and PD ON − controls at 6–8 Hz) still have q ≈ 0.02.

![Group average spectra](figures/group_spectra.png)

## Interpretation

- **Beta:** No reliable change with medication and no difference from controls in central scalp EEG in this sample. This is consistent with scalp findings in the literature being mixed. It does not show that no effect exists: with 15 patients and effects near 0.3, the study is underpowered.
- **Theta:** Theta/low-alpha power was higher ON than OFF in the same patients, and patients ON medication were higher than controls. This is the opposite of what I expected (I would have guessed more low-frequency slowing OFF). It is a lead worth replicating, not an established result.

## Limitations

- Only 15 patients and 16 controls; effects are uncertain and the beta analysis is underpowered
- The theta windows were selected after viewing the data (post-hoc); the result needs replication in independent data
- I do not know the order of the ON and OFF sessions or whether drowsiness or time of day differed between them; either could change theta power
- The theta analysis was not repeated without `sub-pd6` and `sub-pd16`
- Only three central electrodes were analyzed; effects elsewhere on the scalp were not examined
- Relative power can shift when the overall shape of the spectrum changes; I did not separate periodic activity from the 1/f background (e.g., with `specparam`)
- Clinical ratings in the dataset were collected by trained lab staff, not a neurologist, and were not used here

## Reproducing this

1. Open `notebook.ipynb` in Google Colab (CPU is enough).
2. Run the cells in order. The notebook downloads the dataset (about 520 MB) and processes all 46 recordings in roughly 10–15 minutes.

```
pip install -r requirements.txt
```

## Credits and data citation

Data: UC San Diego Resting State EEG Data from Patients with Parkinson's Disease (OpenNeuro ds002778), curated by Alex Rockhill. The dataset's documentation asks users to cite:

- Jackson et al. (2019), *eNeuro*, waveform shape in Parkinson's disease
- Swann et al. (2015), *Annals of Neurology*, elevated synchrony in Parkinson's disease
- George et al. (2013), *NeuroImage: Clinical*, dopaminergic therapy and beta coherence
- Appelhoff et al. (2019), *Journal of Open Source Software*, MNE-BIDS
- Pernet et al. (2019), *Scientific Data*, EEG-BIDS

Analysis done with [MNE-Python](https://mne.tools).
