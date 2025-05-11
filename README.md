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
However, looking at the spectrogram when convolving a 'y' signal with the raw IR reveals much low frequency content, making me think something went wrong here....

*Either I messed up the deconvolution somewhere and I'm not catching it, or the IR capture from The Garage clipped and we didn't notice when we took the measurements with Joel.* 

I've high passed the trimmed BRIR (Trimmed 100 samps before peak, and 3000 after the peak) using a 4th order butterworth hp at 30 Hz to try and deal with it. 
This probably give a less accurate result, however for practical use this is much more pleasant than without...

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

I'm wondering if there's a better way to approach this, instead of this "brute force" approach?

however since clipping seemingly happens, I don't think there's much we can to fix without taking sweeps again.
It really could be a botched measurement and that makes me sad. Especially considering that people hadn't attempted BRIRs of The Garage previously.





I also briefly looked into Toeplitz Deconvolution, but I didn't implement so just used "Vanilla" deconv.
