# fMRI preprocessing
This is a tutorial giving an overview of an fMRI preprocessing pipeline developed by GG and members of the O’Doherty lab. It is based on Nipype workflow, and uses an ICA-based approach for data cleaning.  
  
## Dependencies
The pipeline is a shell script which means that it can be run remotely with minimised dependencies. The pipeline relies, however, on installs of:

- [FSL](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/)
- [ANTs](http://stnava.github.io/ANTs/) (if normalisation is desired)

## Overview
The pipeline consists of 7 modularised scripts which perform the following actions:

- Reorientation/fieldmapping
- ICA
- Classification and component removal
- Fieldmap correction
- Structural normalisation
- Functional normalisation
- Smoothing

The following describes the pipeline, as well as some extra steps described in the tutorial, in greater detail.

### Check data
This should consist of a manual inspection of the data to ensure standard quality. This includes structural and functional data, as well as any field maps or physiological measures collected. At this point it is also a good idea to rename or reorganise the data in a standard way (e.g. [BIDS](http://bids.neuroimaging.io)).

### Reorientation, BET, and fieldmap preparation
This first consists or realigning the data to a standard, and running the brain extraction tool (BET) to isolate brain tissue from non-brain tissue (bone, ventricles, etc.). Secondly, any collected fieldmaps are also reoriented to this standard, and prepared for future field correction.

This is carried out by the first module of the script, `01.Reorient_Preprocess_Fieldmaps.sh`.

### ICA
This next step runs [MELODIC independent component analysis](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/MELODIC) (ICA) on the data, which will identify the key components that account for the most variance in space and time. These components can be viewed using the [Melview](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Melview) tool. Note that at this point the raw data is not affected.

This is carried out by the second module of the script, `02.Run_ICA_Analysis.sh`.

### ICA classification and component removal
This next step relies on [FIX](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FIX), a support vector machine classifier which applies noise/signal labels to the ICA components. 

This classifier has been trained on a number of different samples, two of which are in-house samples (i.e. from the Caltech scanner, using behavioural tasks similar to those used in the lab) provided [here](https://github.com/giogen/fMRI-ICA-denoising). These are specific to data acquired using either a TR of 2.5-3 secs, with 3 mm voxels, or using multiband EPI with a 1 sec TR and 2.5 mm voxels.

There are also some pre-trained classifiers available on the [FIX website](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FIX/UserGuide). In general, it is recommended to use a custom trained classifier, optimised for the scanner and experiment.

Once these are classified, the script then removed the noise components from the data.  
  
However, note that it is possible in general to view and amend the classification labels prior to removal of the components.

This is carried out by the third module of the script, `03.Apply_ICA_Denoising.sh`. Note that the script requires a pointer toward the desired classifier.

### Fieldmap correction
This applies fieldmap correction to the structural and functional data.

This is carried out by the fourth module of the script,
`04.Apply_Fieldmaps_Spatial_Unwarping.sh`.

### Structural and functional normalisation
The next step carries out normalisation using ANTs, before the structural and function data are aligned together.

Note that this also re-runs BET for the functional image.

This is carried out by the fifth (`05.Compute_Structural_Warp.sh`) and sixth (`06.Compute_Functional_Warp.sh`) modules of the script.

### Smoothing
Finally, a gaussian kernel is applied to smooth the data.

This is carried out by the final module of the script, `07.Apply_Spatial_Smoothing.sh`.

/images/preprocessing_tutorial.jpg
