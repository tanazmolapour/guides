# DTI Protocol

Requires:

[MRIConvert](http://lcni.uoregon.edu/~jolinda/MRIConvert/)

[FSL](http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/)


Open MRIConvert to convert the .IMA files from Siemens (only need the DTI series). Specify an output folder and then add all of the .IMA files from the DTI series. Change the file conversion to 'FSL NIfTI' and before converting, make sure that 'Save multivolume series as 4D files' and 'Save as .nii file' are checked in the options. 

This program should convert the series into 3 files, a .gz volume, and two text files (bval and bvec).

Launch FSL using terminal and change the directory to where the data is stored. Using the GUI, open FDT Diffusion and in the pull down options at the top, select EDDY CURRENT CORRECTION. Select the .gz file as the Diffusion weighted data and name the output file accordingly (_ecc or the like). The reference volume should be set to 0 (the first volume should be the one with the zero b-value, but this can be checked using MRIcron). Press 'Go' and wait for the correction to complete.

To create a brain mask, open BET from the main FSL window. Use the corrected data file (_ecc) as input. In 'Advanced Options' untick everything except 'Output binary brain mask image'. The 'Fractional Intensity Threshold' will determine how much of the tensors outside the surface of the cortex are calculated (suggested value is 0.3, but if this cuts off too much of the brain then decrease this). Press 'Go'.

From the FDT DIFFUSION window, select DTIFIT RECONSTRUCT DIFFUSION TENSOR from the pulldown selections at the top. Tick 'Specify input files manually'. Your diffusion weighted data is your corrected data file (_ecc), your BET binary brain mask is the most recent file you have created, the gradient directions is your .bvec file and the b values are your .bval file. Press 'Go'.

To view the created files, open FSLView from the main menu. Open the Fractional Anisotropy map, which is the file with FA in the title. If you then choose File/Add, you can select which overlay you would like (V1,V2,V3 are orthogonal vectors, V1 is the one we want). Once this is added, select it in the toolbox window at the bottom. There should be an 'Information' button in this box - if you press this it will bring up a window labelled 'Overlay Information Dialog'. In this window change 'Image type' to 'Diffusion tensor' and 'Display as' to either 'Lines', 'RGB' or 'RGB' Lines. Underneath this, change 'Modulation' to the FA file ('dti_FA').

