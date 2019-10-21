# nnAudio
Audio processing by using pytorch 1D convolution network. By doing so, spectrograms can be generated from audio on-the-fly during neural network training. 

# Dependencies
Numpy 1.14.5

Scipy 1.2.0

PyTorch 1.1.0

# Instructions
All the required codes are contained inside the jupyter-notebook. The audio processing layer can be integrated as part of the neural network as shown below.


```diff
class Model(torch.nn.Module):
    def __init__(self, avg=.9998):
        super(Model, self).__init__()
        # Getting Mel Spectrogram on the fly
+       self.spec_layer = Spectrogram.STFT(sr=44100, n_fft=n_fft, freq_bins=freq_bins, fmin=50, fmax=6000, freq_scale='log', pad_mode='constant', center=True)
        self.n_bins = freq_bins         
        
        # Creating CNN Layers
        self.CNN_freq_kernel_size=(128,1)
        self.CNN_freq_kernel_stride=(2,1)
        k_out = 128
        k2_out = 256
        self.CNN_freq = nn.Conv2d(1,k_out,
                                kernel_size=self.CNN_freq_kernel_size,stride=self.CNN_freq_kernel_stride)
        self.CNN_time = nn.Conv2d(k_out,k2_out,
                                kernel_size=(1,regions),stride=(1,1))    
        
        self.region_v = 1 + (self.n_bins-self.CNN_freq_kernel_size[0])//self.CNN_freq_kernel_stride[0]
        self.linear = torch.nn.Linear(k2_out*self.region_v, m, bias=False)
        
    def forward(self,x):
+        z = self.spec_layer(x)
        z = torch.log(z+epsilon)
        z2 = torch.relu(self.CNN_freq(z.unsqueeze(1)))
        z3 = torch.relu(self.CNN_time(z2))
        y = self.linear(torch.relu(torch.flatten(z3,1)))
        return torch.sigmoid(y)
```

## Demostration
The spectrogram outputs from nnAudio are nearly identical to the implmentation of librosa. The only difference is CQT, where we normalized the CQT kernel with L1 norm and then CQT output is normalized with the CQT kernel length. I am unable to explain the normalization used by librosa. 

To use nnAudio, you need to define the neural network layer. After that, you can pass a batch of waveform to that layer to obtain the spectrograms. The input shape should be `(batch, len_audio)`.

```
import Spectrogram 
CQT_layer = Spectrogram.CQT2019(sr=44100, n_bins=84*2, bins_per_octave=24, fmin=55) # Defining the neural network
spec = CQT_layer(x) # x is the audio clips with shape=(batch, len_audio)
```
![alt text](https://github.com/KinWaiCheuk/nnAudio/blob/master/performance_test/performance_chrom.png)

## Speed
The speed test is conducted using DGX Station with the following specs

CPU: Intel(R) Xeon(R) CPU E5-2698 v4 @ 2.20GHz 

GPU: Tesla v100 32gb

RAM: 256 GB RDIMM DDR4

During the test, only 1 single GPU is used, and the same test is conducted when 

(a) the DGX is idel

(b) the DGX has ongoing jobs
![alt text](https://github.com/KinWaiCheuk/nnAudio/blob/master/speed_test/speed.png)

