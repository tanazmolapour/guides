# Physiological recording with Biosemi Actiview II

## Electrocardiogram
- Requires CMS/DRL ground electrodes, and 2 EXG electrodes (usually EX1 and EX2)
- For all electrode positions, make sure skin is clean before adhesion using exfoliant paste and alcohol wipe
- EX1 should be attached a centimetre under the middle of the right clavicle
- EX2 should be attached to left side of body, slightly above the third rib (counting from the bottom)
- CMS/DRL a centimetre under the left clavicle, three inches apart from each other (if possible)
- In Actiview, the best way to view the data is to set the reference electrode to EX1, and the y-scale to ~2mV
- The signal should have a well defined QRS complex
- An adaptation period of at least 5 minutes is advisable

## Galvanic skin response
- Requires CMS/DRL ground electrodes and GRS electrodes (only available for Mobile EEG system)
- Have participants wash and dry their hands prior to attaching electrodes
- Position each electrode on the first and second fingers of the left hand, in between the second and third joint
- Use the larger adhesive rings to maximise contact with the skin
- CMS/DRL electrodes can be placed on the upper arm if they are not being used to measure ECG
- GSR signals can be viewed under the ‘Auxiliary Sensors’ tab in ActiView
- Adjust the scale to ~100mV or ~200mV if necessary 
- Make sure to check ‘Add displayed sensors’ when saving the data
- An adaptation period of at least 10 minutes is strongly advised

## Eyeblink rate (EOG)
- Requires CMS/DRL ground electrodes, and 2 EXG electrodes (usually EX3 and EX4 if also measuring ECG)
- For all electrode positions, make sure skin is clean before adhesion using exfoliant paste and alcohol wipe (ask participants to close their eyes to protect from alcohol fumes)
- EX3 should be attached to the lateral side of the left eye
- EX4 should be attached underneath the left eye
- CMS/DRL electrodes can be placed on the mastoids or the upper arm if they are not being used to measure ECG
- To view the eyeblink signal, set the reference electrode to EX3
- Ideally, eyeblink rate should be captured over a 5-10 minutes interval in constant illumination without visual stimulation

## Triggers
- To send triggers to Actiview from a computer, you will need a parallel (LTP) port and software to send a trigger
- If using MATLAB, see the io32 / io64 website to download a mex file and get help with this
- Alternatively, triggers can be manually entered using the Fn keys on the keyboard (these can be combined for a total maximum of 256, but it is advisable to use each button separately)

## General troubleshooting
- If there are weird spikes in the data window of Actiview, the CMS/DRL electrodes are not making proper contact. Try refitting them elsewhere. Sometimes this problem resolves itself (maybe because of differences in skin conductance)
