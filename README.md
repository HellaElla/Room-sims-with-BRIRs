# Convolving BRIRs

This code aims to obtain IRs from recorded sine sweeps using a KU-100 binaural head microphone(s). 

the inverse sweep is tackled like this: 

```
def create_inverse_sweep(sweep):
    # Time-reverse the sweep
    inverse = sweep[::-1]
    
    # Apply amplitude envelope correction for log sweeps
    t = np.arange(len(sweep))
    envelope = np.exp(-t / len(t) * np.log(sweep.shape[0]))
    
    # Apply the envelope to the time-reversed sweep
    inverse = inverse * envelope
    
    return inverse
```
and is then convolved with the recorded sweep in order to deconvolve and obtain IR. 
However, looking at the spectrogram when convolving a 'y' signal with the IR reveals much low frequency content, making me think something went wrong here....

I've high passed the trimmer BRIR using a 4th order at 30 Hz and this seemed to fix the low frequency blowout... at least perceptually.

```
# Design a high-pass filter
cutoff_freq = 30
nyquist = sample_rate / 2
normalized_cutoff = cutoff_freq / nyquist
order = 4  # Filter order

# Using second-order sections for better numerical stability
sos = butter(order, normalized_cutoff, btype='highpass', output='sos')

# Apply the filter to each channel
if len(brir_trimmed_both.shape) == 1:
    # For mono
    brir_highpassed = sosfilt(sos, brir_trimmed_both)
else:
    # For stereo or multi-channel
    brir_highpassed = np.zeros_like(brir_trimmed_both)
    for channel in range(brir_trimmed_both.shape[1]):
        brir_highpassed[:, channel] = sosfilt(sos, brir_trimmed_both[:, channel])
```

This probably give a less accurate result, however for practical use this is much more pleasant.
