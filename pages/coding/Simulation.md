# Simulation of retinotopic responses in V1
Here we show how to simulate retinotopic responses for specific visual stimuli over a specific cortical area. We assume you have measured the retinotopy of part of the cortical area. We will use this partial map to constrain our retinotopic model in order to generate the full map over the two hemispheres. Then we will use the cortical coordinates generated by the model to project any visual stimulus to the V1 space.


## 1. Fit the Model
#### Calibrate retinotopic model with real retinotopy

```Matlab
% RX, RY, : retinotopic cartesian coordinates
param = FitRetino(RX,RY)
```

<p align="center">
<img src="./figures/FitRetino.png" width="70%">
</p>

## 2. Match vasculature
 Find correspondence btw retinotopy's experiment vasculature and the vasculature of the experiment you want to simulate ()[vasculature registration software](https://github.com/giacomox/BruteForceRegistration/blob/master/README.md)).

```Matlab
% I,J : vasculature images
tform = SelectFeatures(I,J);
```

<p align="center">
<img src="./figures/selection.png" width="90%">
</p>


## 3. Generate Retinotopic map

```Matlab
[Xq, Yq ]= meshgrid( 1:size(RX,1) , 1:size(RX,2) );
[Uq Vq] = RetinoModel_INV(Xq,Yq,param)
```
<p align="center">
<img src="./figures/Interp2.png" width="70%">
</p>



## 4. Get ROI retinotopy
 Use the vasculature coordinates to generate the retinotopy of the experiment you want to simulate (i.e. crop the retinotopic map in the same way you cropped the vasculature image to match the two experiments)

```Matlab
UU = imwarp(Uq,tform,'OutputView', imref2d(size(I)));
VV = imwarp(Vq,tform,'OutputView', imref2d(size(I)));
```

<p align="center">
<img src="./figures/M15D20141209R1_Retino.png" width="70%">
</p>

## 5. Simulate response
 Project visual stimulus to retinotopic map and compare with real response

 You can use the same function you used to generate the visual stimulus in the visual space to generate the projection to  the cortical ROI<sup>4</sup> .

```Matalb
[x,y] = Gabor2D(UU,VV,param_stimulus)
```

<p align="center">
<img src="./figures/M15D20141209R1.png" width="70%">
</p>

In some cases (e.g. if using another software to display the stimuli and set their position on screen) you have to take in account the (i) stimulus position (the Matlab fun generates the stimulus image but the position is changed through the stimulation software) and (ii) the fact that the Y axis of the stimulus is inverted.

```Matlab
%% Set stimulus position
% Cx and Cy are the cartesian coordinates of the stimulus' center
Cx = 1; Cy = -1;
% UU and VV are the visual coordinates corresponding to the cortical
% coordinates we want to use to generate the retinotopic response
% simulation (the ROI).
UU = UU - Cx ;
VV = VV - Cy ;
```

```Matlab
%% Correct the stimulus
```

## 6. Cortical point image
Retinotopic projections must be "blurred" by convolving with a 2D gaussian kernel with adequate parameters, to simulate the effect of the CPI.

Since the receptive fields of sensory neurons extend over space (e.g. the average full-width-at-half-height (FWHH) of the receptive field of V1 neurons at 2-5 dva of eccentricity is ~0.6 dva)<sup>1,2</sup>, the receptive fields of laterally-displaced neurons largely overlap, and each point in the visual field is represented by a widespread population of neurons. This area, which is referred to as the *cortical point image (CPI)*, in V1 spans a region with full-width-at-half-height of ~2 mm for spiking activity and ~4 mm for sub-threshold activity<sup> 3</sup>. The predicted retinotopic response to a visual stimulus can therefore be obtained by projecting the stimulus to the retinotopic map and then convolving the result with the sub-threshold CPI<sup>4</sup> (e.g. 2D Gaussian filter, corresponding to a "blurring" effect). The effect of the CPI on the amplitude of the predicted response is negligible at low SFs, but it becomes dominant as the spatial frequency of the stimulus increases.

```
TO DO
```


## References
1. Van Essen, D. C. & Newsome, W. T. The visual field representation in the striate cortex of the macaque monkey: Asymmetries, anisptropies, and indiviual variability. Vision Res. 24, 429–448 (1984).

2. Hubel, D. H. & Wiesel, T. N. Uniformity of monkey striate cortex: a parallel relationship between field size, scatter, and magnification factor. J. Comp. Neurol. 158, 295–305 (1974).

3. Palmer, C. R., Chen, Y. & Seidemann, E. Uniform spatial spread of population activity in primate parafoveal V1. J. Neurophysiol. 107, 1857–67 (2012).

4. Benvenuti, G. et al. [Scale-Invariant Visual Capabilities Explained by Topographic Representations of Luminance and Texture in Primate V1.](https://www.ncbi.nlm.nih.gov/pubmed/30392796) Neuron 1–9 (2018).