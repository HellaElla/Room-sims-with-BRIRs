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
