# WPI SP210iwz syringe pump
### Setup
- Stored with the pump should be a RS232 cable (looks like a telephone cable), and Prolific USB-to-Serial Comm Port adapter
- The correct drivers for this adapter must be installed ([for Windows](http://www.prolific.com.tw/US/ShowProduct.aspx?p_id=225&pcid=41), or for [macOS](http://www.prolific.com.tw/US/ShowProduct.aspx?p_id=229&pcid=41))
- The RS232 cable must be coupled to the ‘IN’ port of the pump
- Ensure syringes are adequately cleaned prior to use
- If necessary, syringes can be disinfected with bleach (but rinse thoroughly)
- Attach the PVC tubes to the ends of the syringes, and couple them together using a plastic Y-joiner 
- Ensure a new, clean mouthpiece is attached to the end of the tube (we have been using disposable dental air/water syringe spray nozzles)

*Note: 150 mL syringes can be purchased from [SDR scientific](http://www.sdr.com.au/plasticware.php), via email. The product code for these is ‘QPC1108	Syringe 140 cc Monoject’. Syringe barbs (QP11536) that will connect the syringe tips to the appropriately sized tubing (ACF00007) are also available. You may also need a Y tubing connector to merge the two syringes. Dental air/water nozzles can be ordered on eBay.*

### Code
There are five custom MATLAB functions to use the pump. Note that some of these functions will throw a pump buffer error, but I haven’t been able to debug this, and it doesn’t appear to interfere with the operation at all.

#### ```pumpSetup```
This is a general setup script that will establish a connection between the pump and your computer. Firstly, it makes sure you have an appropriate MATLAB toolbox on the path. If this fails, you will need to make sure it is installed. If you are running a PC, you will have to manually define which COM port the pump is connected to (via ‘Device Manager’). MacOS should do this automatically. The function will return an address to the pump that has to be used as an argument for all subsequent functions. It is important to remove the connection after use by first clearing the pump address variable and then clearing the connection by typing ```delete(instrfindall)```.

	function [pump] = pumpSetup
	addpath(genpath('/Applications/MATLAB_R2012b.app/toolbox/shared/instrument/')); % Make sure serial loader is on path
	if ismac
	    devs = dir('/dev/cu.usbserial*');
	    if length(devs) > 1
	        fprintf('Multiple devices detected. Please choose one.\n');
	        for i = 1:length(devs)
	            fprintf('%i: %s\n', i, char(devs(i)));
	        end
	        fprintf('\n');
	        devidxs = input('Type the number corresponding to the pump:\n','s');
	        devid = str2double(devidxs);
	        portNum = devs(devid);
	    end
	elseif ispc
	    devidxs = input('Please input the COM port number of the syringe pump: ', 's');
	    portNum = devidxs;
	else
	    error('Cannot identify operating system.\n');
	end
	if ~ispc
	    port = ['/dev/COM' portNum];
	else
	    port = ['COM' portNum];
	end
	pump = serial(port);
	fopen(pump);
	fprintf('Pump is now controllable.\n');
	return

#### ```pumpEmpty```
This function will infuse the pump at maximum speed until a key is pressed. This is useful for clearing any leftover fluid, or unwanted air from the syringes. Please make sure to stop the process before the syringes are completely empty (and clear the remainder manually if necessary), otherwise the pump can jam.

	function [dia, vol, rate] = pumpEmpty(pump) 
	% Infuses liquid to empty until key is pressed
	% Set to infuse
	cmd = '2 mode i';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Set diameter
	cmd = '2 dia 38.40';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Set volume and rate
	cmd = sprintf('2 voli %05.0f ml', 0);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	cmd = sprintf('2 ratei %05.1f ml/m', 120);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Query diameter, volume and rate
	cmd = '2 dia?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	dia = fscanf(pump, '%s');
	cmd = '2 voli?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	vol = fscanf(pump, '%s');
	cmd = '2 ratei?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	rate = fscanf(pump, '%s');
	if any(strcmp('2E',{rate vol dia}))
	    fscanf(pump);
	    error('The pump has thrown an error. Pump buffer cleared.');
	end
	% Run pump
	cmd = '2 run';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	KbWait([], 2);
	% Stop pump
	cmd = '2 stop';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	return

#### ```pumpRefill```
This function is the opposite of pumpEmpty, and will run withdrawal until a key is pressed. Again, please stop the process before the pump reaches it’s limits.

	function [dia, vol, rate] = pumpRefill(pump) 
	% Withdraws liquid to refill until a key is pressed.
	% Set to withdraw
	cmd = '2 mode w';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Set diameter
	cmd = '2 dia 38.40';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Set volume and rate
	cmd = sprintf('2 volw %05.0f ml', 0);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	cmd = sprintf('2 ratew %05.1f ml/m', 147);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Query diameter, volume and rate
	cmd = '2 dia?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	dia = fscanf(pump, '%s');
	cmd = '2 volw?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	vol = fscanf(pump, '%s');
	cmd = '2 ratew?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	rate = fscanf(pump, '%s');
	if any(strcmp('2E',{rate vol dia}))
	    fscanf(pump);
	    warning('The pump has thrown an error. Pump buffer cleared.');
	end
	% Run pump
	cmd = '2 run';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	KbWait([], 2);  
	% Stop pump
	cmd = '2 stop';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	return

#### ```pumpWit```
This function will allow you to specify a volume to withdraw. It has been adjusted under the assumption that two syringes are being used simultaneously, and will withdraw at the maximum rate allowed. If ‘0’ is given as the volume argument, the pump will terminate any current actions. This function is useful to prevent leaking when the experiment is not active.

	function [dia, vol, rate] = pumpWit(pump, v) 
	% Withdraws liquid (to prevent leaking) at maximum rate. Precision is 3
	% decimal points of a mL.
	v = v / 2; % Because there are two syringes
	% Set to withdraw
	cmd = '2 mode w';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	if v ~= 0
	% Set diameter
	cmd = '2 dia 38.40';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Set volume and rate
	cmd = sprintf('2 volw %05.3f ml', v);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	cmd = sprintf('2 ratew %05.1f ml/m', 147);
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	% Query diameter, volume and rate
	cmd = '2 dia?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	dia = fscanf(pump, '%s');
	cmd = '2 volw?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	vol = fscanf(pump, '%s');
	cmd = '2 ratew?';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	rate = fscanf(pump, '%s');
	if any(strcmp('2E',{rate vol dia}))
	fscanf(pump);
	error('The pump has thrown an error. Pump buffer cleared.');
	end
	% Run pump
	cmd = '2 run';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	else
	% Stop pump
	cmd = '2 stop';
	fprintf(pump, '%s\r\n', cmd);
	fscanf(pump); % discard garbage response
	end
	return

#### pumpInf
This function will allow you to infuse a specific volume (maximum of 9.8 mL). Again, it is hard coded for use with two syringes, but this can simply be modified by manually doubling the required volume if you only have one syringe. Importantly, the rate of infusion is constant, which means that experimental timing will not be affected by the use of different volumes.

	function [dia, vol, rate, err] = pumpInf(pump, v) 
	% Takes volume (v) from 1-140. If v is 0, pump will
	% stop. This version will adapt the rate to the volume, delivering
	% volume (max 9.8 mL) in a fixed interval (2 seconds). Precision is 3 decimal points of a mL.
	flushinput(pump); % Remove data from input buffer
	v = v / 2; % Because there are two syringes
	r = v * 30; % 1mL/s is 60mL/min (pump units). For each mL of volume, we should have a rate of 60mL/min. We have already halved the volume because we are using two syringes, but we need to halve the rate again if we want a longer delivery (2s). For 2mL/s (e.g. 120mL/min) we simply double the modifier. The maximum value the pump can take for rate is 147 (147mL/min), thus the maximum volume we can deliver in a second is 2.45mL per syringe (4.9mL).
	% Set to infuse
	cmd = '2 mode i';
	query(pump, cmd, '%s\r\n');
	if v ~= 0
	% Set diameter
	cmd = '2 dia 38.40';
	query(pump, cmd, '%s\r\n');
	% Set volume and rate
	cmd = sprintf('2 voli %05.3f ml', v);
	query(pump, cmd, '%s\r\n');
	cmd = sprintf('2 ratei %05.1f ml/m', r);
	query(pump, cmd, '%s\r\n');
	% Query diameter, volume and rate
	cmd = '2 dia?';
	query(pump, cmd, '%s\r\n');
	dia = fscanf(pump, '%s'); % Record response
	cmd = '2 voli?';
	query(pump, cmd, '%s\r\n');
	vol = fscanf(pump, '%s'); % Record response
	cmd = '2 ratei?';
	query(pump, cmd, '%s\r\n');
	rate = fscanf(pump, '%s'); % Record response
	cmd = '2 error?';
	query(pump, cmd, '%s\r\n');
	err = fscanf(pump, '%s'); % Record response
	if strcmp(err,'4')
		flushinput(pump); % Remove data from input buffer
		warning('The pump has thrown an error (serial overrun). Pump buffer cleared.');
	end
	% Run pump
	cmd = '2 run';
	query(pump, cmd, '%s\r\n');
	else
	% Stop pump
	cmd = '2 stop';
	query(pump, cmd, '%s\r\n');
	end
	return

### Troubleshooting
If there are issues with the pump, there are a number of things to check first:
- Firstly check the connections to the computer, power point, and make sure the power is switched on (switch it off and then on)
- Check that the latch switch (dorsal facing knob) is not pointing right (this means that the pump is not latched to the driver and will not move)

Sometimes the pump will not work because it has stalled. To fix this, there are a number of things to try:
- Firstly, clear the pump connection by clearing the ‘pump’ variable and then clearing the connection by typing ```delete(instrfindall)```
- Try reconnecting. If this doesn’t work, disconnect again and try either of the above tips before reconnecting again.

The pump may also fail to work when the syringes become fatigued. This can happen especially in the case where liquids other than water are frequently used, or when the syringe is washed with very hot water. If this is the case, you will have to replace the syringes.