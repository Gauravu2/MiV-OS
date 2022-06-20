---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.13.8
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Connectivity Analysis Methods

+++

## 1. Data Load and Preprocessing

```{code-cell} ipython3
:tags: [hide-cell]

import os, sys
import miv
import scipy.signal as ss

from miv.io import DataManager
from miv.signal.filter import ButterBandpass, MedianFilter, FilterCollection
from miv.signal.spike import ThresholdCutoff
from miv.statistics import pairwise_causality
from miv.visualization import plot_spectral, pairwise_causality_plot
import numpy as np

from miv.typing import SignalType
import matplotlib.pyplot as plt
import matplotlib.colors as colors
import matplotlib.cm as cm
```

```{code-cell} ipython3
# Experiment name
experiment_query = "experiment0"

# Data call
signal_filter = (
    FilterCollection()
        .append(ButterBandpass(600, 2400, order=4))
        .append(MedianFilter(threshold=60, k=30))
)
spike_detection = ThresholdCutoff(cutoff=5)

# Spike Detection
data_collection = optogenetic.load_data()
data = data_collection.query_path_name(experiment_query)[0]
#false_channels = [12,15,36,41,42,43,45,47,53,57,55,58,61,62]
#data.set_channel_mask(false_channels)
with data.load() as (signal, timestamps, sampling_rate):
    # Preprocess
    signal = signal_filter(signal, sampling_rate)
    spiketrains = spike_detection(signal, timestamps, sampling_rate)
```

## 2. Pairwise Granger Causality

Estimates pairwise Granger causality between two channels

pairwise_causality(signal, start, end):

Parameters

#----------\
signal : SignalType\
Input signal\
start : float\
starting point from signal\
end : float\
End point from signal\
Returns

#-------\
C : Causality Matrix containing directional causalities for X -> Y and Y -> X,
    instantaneous causality between X,Y, and total causality. X and Y represents electrodes


```{code-cell} ipython3
#Example
pairwise_causality(signal, 0, 1000)
# Estimates the causality between each pair of electrodes for signals strating from 0-1000
```

## Pairwise Granger Causality Plot

Plots pairwise Granger Causality

pairwise_causality_plot(signal, start, end):

#Parameters\
#----------\
#signal : SignalType\
Input signal\
start : float\
starting point from signal\
end : float\
End point from signal\

#Returns\
figure, axes\ 
Contains subplots for directional causalities for X -> Y and Y -> X,
instantaneous causality between X,Y, and total causality. X and Y represents electrodes


```{code-cell} ipython3
#Example
pairwise_causality_plot(signal, 0, 1000)
# Plots the causality between each pair of electrodes for signals strating from 0-1000
```

## Welch Coherence 

Plots Power Spectral Densities for channels X and Y, Cross Power Spctral Densities and Coherence between them using Welch's method\

plot_spectral(signal, X, Y, sampling_rate, Number_Segments)\


Parameters\
#----------
signal : SignalType\
    Input signal\
X : float\
    First Channel \
Y : float\
    Second Channel\
sampling_rate : float\
    Sampling frequency\
Number_Segments: float\
Number of segments to divide the entire signal\

Returns\
#-------
figure: plt.Figure\
axes\

References:\ 

1) https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.coherence.html
2) P. Welch, “The use of the fast Fourier transform for the estimation of power spectra: A method based on time averaging over short, modified periodograms”, IEEE Trans. Audio Electroacoust. vol. 15, pp. 70-73, 1967.


```{code-cell} ipython3
#Example 
plot_spectral(signal,1,42,30000,10000)

##Plots the PSDs, CPSD and Coherence for channel 1 & 43 for a sampling rate of 30000, with signal divided into 10000 segments
```