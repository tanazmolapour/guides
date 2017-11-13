# Electrocardiography protocol
### Participant setup
- Firstly, provide participants with a brief verbal explanation of what ECG is, how it will be set up, the risks (none), and what will be recorded
- You will need two external (EXG) electrodes for measurement (usually EXG1 and EXG2), as well as the CMS/DRL electrodes as an implicit reference, some electrode gel and adhesive rings
- For all electrode positions, make sure skin is clean by using exfoliant paste and an alcohol wipe, but make sure to wait an appropriate time for the alcohol to evaporate prior to fitting electrodes
- EXG1 should be attached a centimetre under the middle of the right clavicle
- EXG2 should be attached to left side of body, slightly above the third rib (counting from the bottom)
- The CMS/DRL electrodes should be placed a centimetre under the left clavicle (approximately contralateral to EXG1), three inches apart from each other (if possible)
- This configuration can be referred to as a modified Lead II Einthoven configuration

### Triggers
If you are wanting to insert markers (triggers) into your data for task segmenting or response-related activity, you have two options. For each of these, you can use values from 0-255.

#### Keyboard
- You can manually insert triggers using the function keys (F1, F2, etc) on the recording computer
- The function keys insert values by powers of 2 (i.e., 1,2,4,8,16 etc)
- These can be combined to create all the values, but this is not recommended, as keys have to be pressed precisely simultaneously
- If more than 10 values, or precise timing are required, use a parallel port instead

#### Parallel port
- You can send triggers from the task presentation computer (running MATLAB) to the recording computer via a parallel port (aka LTP, or printer port)
- This has the advantaged of being tightly linked to stimulus presentation, so timing is precise and automated
- **Most new computers do not have parallel ports**, if this is the case, please contact the Decision Neuroscience Lab (www.decision.neuro.melb@gmail.com) for information about a USB alternative
- Firstly, you must install the io32 or io64 driver, depending on your operating system. You can download the 32 bit version [here](http://apps.usd.edu/coglab/psyc770/IO32.html), and the 64 bit version [here](http://apps.usd.edu/coglab/psyc770/IO64.html), or if you are running 32 bit MATLAB on 64 bit Windows, [here](http://apps.usd.edu/coglab/psyc770/IO32on64.html). Follow the instructions on the website
- The 25 pin cable is attached to the unit that interfaces the EEG system with the recording computer, and should run into the testing booth
- Attach this to the testing computer, and identify which (LTP) port number it is attached to, and the port address via ‘Device Manager’ in Windows. The address should be the first value under the ‘range’ section (alternatively you can use trial and error by appending ‘d-f’ to ‘010’)

A configuration script must be run at the start of each experiment (substitute ‘io32’ with the driver you are using):

		% Open parallel port and test
		ioObj = io32;
		status = io32(ioObj);
		if status
		    error('Cannot initialise io32.\n');
		end
		address = hex2dec('e010'); % This is the standard LPT1 port address (‘0x378’)
		io32(ioObj,address,0); % Send a 0 to reset

Here is example code for sending a trigger:

	function sendTrigger(address, triggerValue)
	io32(ioObj,address,triggerValue);
	WaitSecs(0.01);
	io32(ioObj,address,0);
	return

Note that a 0 is sent after the intended value. This is to make sure all the pins on the port are reset. Note also that there is a limitation on the speed of sending sequential triggers, so if you do want to send triggers in rapid succession, please test this and adjust the WaitSecs() value accordingly.

### Recording
- Once setup, start the ActiView software
- The settings on the left side of the browser configure only what is displayed — they will not affect the recorded data
- Recommended settings here are to set the Y-scale to ~2 mV, the display channels to EXG1 and EXG2 (select ‘Add 8 EXG), the reference electrode to EXG1 (by selecting ‘Free choice’ in the dropdown menu), and the trigger format to ‘Decimal’
- Clicking ‘Start’ in the top left corner will initiate the display of the signal, but will not record data
- The signal should have a well defined QRS complex
- If the signal exhibits repetitive spikes, this likely relates to improper contact of the CMS/DRL electrodes. To check this, remove these electrodes and force contact with your fingertips — if the signal recovers, then the CMS/DRL electrodes were not making proper contact with the participant. Try refitting them in a new position. (Often this problem resolves itself after a few minutes, and is possibly due to skin conductance issues)
- An adaptation period of at least 5 minutes is advisable before the beginning of recording (although this can also be removed after recording)
- The options of the right of the browser relate to recording — generally the defaults (e.g. 512 Hz sampling rate) are recommended
- When you press ‘Start File’, the program will prompt you to enter a save name and location for the data file
- ***THE DATA IS NOT BEING RECORDED YET!***
- You must press the recording button again to start recording. To double check, look at the right hand side of the browser and make sure the size of the file is increasing
- Once you have finished recording, press the button again

### Analysis
This will not be covered in detail, but some useful tools are briefly outlined below. Before any stage of processing, it is generally recommended that the default file format ‘.bdf’ is converted to ‘.edf’ using the [BDF to EDF converter](http://www.biosemi.com/download.htm)
#### EEGlab
- [EEGlab](http://sccn.ucsd.edu/eeglab/) is a MATLAB toolbox with a graphical user interface that can be used for data analysis
- Files must be read in using the BioSig toolbox
- R-wave peaks can be identified with custom code (e.g. the findPeaks() function), or by using the QRS detection algorithm in the [FMRIB plug-in](http://fsl.fmrib.ox.ac.uk/eeglab/fmribplugin/)
- It is recommended that data are manually inspected to make sure this has worked
- Batch scripts can be easily written by running through the analysis for a single participant, and then using the ‘eegh’ command to generate code for that procedure

#### Kubios HRV
- Alternatively, data can be analysed using [Kubios HRV](http://kubios.uef.fi)
- Kubios HRV automatically detrends the data, corrects artefacts, and identifies R-wave peaks
- It also allows easy viewing of the entire data file, and corrupted segments can be removed
- It automatically generates time-domain measures (e.g. RR intervals, SDNN), frequency-domain measures (e.g. VLF, LF and HF components of HRV), as well as non-linear measures (e.g. Poincaré plots, entropy)
- Also provides ECG-derived respiration (EDR)
- **Kubios HRV cannot be used for batch processing, and cannot read in triggers**
- A pdf manual for Kubios HRV 2.0 can be found [here](http://bsamig.uef.fi/kubios/kubios_hrv_users_guide.pdf)


*For more information about data analysis (e.g. event related responses, task phase analysis, moving average analysis) or for MATLAB scripts, please contact Bowen (bowenjfung@gmail.com), or the Decision Neuroscience Lab (www.decision.neuro.melb@gmail.com).*