   # QCL_spectrometer_scripts
MATLAB scripts for operating a double-beam MIRcat QCL-based spectrometer with SR865A lock-in detection.

Please cite Mukherjee, S., Fried, S.D.E., Hong, N.Y., Bagheri, N., Kozuch, J., Mathews, I.I., Kirsh, J.M., and Boxer, S.G. Detection of Drug Binding in Live Cells using Quantum Cascade Laser Spectroscopy with Nitrile-Incorporated Proteins. In preparation.

This MATLAB live script interfaces with a Daylight Solutions MIRcat QCL and SR865A lock-in amplifier to collect IR scans over the wavelength range specified. The equipment parameters are set as following:

MIRcat-QT-2000 with M2048-P tunable module (cooled to 19°C using a closed-loop water chiller):
  30% Duty Cycle
  Pulse rate: 1 MHz
  780 mA (full current)
  Process Trigger Mode: Internal Step Mode
  Pulse Mode: Internal Pulse Mode
  Wavelength Trigger Start: 2360.00 cm-1
  Wavelength Trigger Stop: 1970.00 cm-1
  Wavelength Trigger Inverval: 10.00
  Wavelength Trigger Pulse Width: 1 us
250-micron pathlength aqueous samples
Stanford Research Systems SR865A Lock-in Amplifier
  Time constant set to 300 microseconds
  Input range 1 V
  500 mV sensitivity
The laser trigger output is connected to the SR865A reference input for lock-in detection.
The laser pin 3 (wavelength trigger) is connected to the trigger input on the SR865A to trigger starting of the data capture buffer at the 2360 cm-1 starting wavelength.

The first section of the script gives scan parameters most likely to be adjusted by the user at each use: 
1. When using a balanced detector such as the custom equipment provided by VIGO, there will be 3 channel outputs: balanced (channel-1), signal (channel-2), and reference (channel-3). Set channel number depending on which output BNC is connected to the lock-in amplifier. Note that for the lock-in amplifier, dark currents should be compensated prior to use, reading the BNC outputs on an oscilloscope and adjusting the knobs included in the detector design. Our VIGO detector had two modes: balanced and autobalanced. We consistently used it in balanced mode.
2. Set parameter has_sample epending on whether the measurement is a blank-vs-blank measurement (has_sample=0) or a sample-vs-blank measurement (has_sample=1). This is used later for calculating absorbance spectra from the data and for file naming purposes.
3. Change save_dir to where the data is to be saved to.
4. Change run_dir to were the MIRcatSDK libraries are saved. These can be downloaded from Daylight Solutions.
5. The headerfile in the run_dir should be named (usually named 'MIRcatSDK.h')
6. Set the data capture buffer size, which will scale with the number of data points. Our default is to use 32-kb capture buffers.
7. The SR865A maximum capture rate is divided by 2^n, where n is set as the constant 'rate'. Our default is to use rate=4.
8. Starting wavelength is set. Our default is to start at 2360 cm-1 (same as the wavelength trigger and capture buffer start) and scan towarads lower frequencies.
9. Laser scan rate set. Our default is to use 2000 cm-1
10. Number of repetitions set. Our default is 16 scans as described in our accompanying publication.

The second section of the script creates variables to control the laser using the SDK library. These sections are based on example scripts provided by Daylight Solutions.

The third section of the script initializes both the MIRcat QCL and the SR865A lock-in amplifier.

The fourth section of the script arms the laser.

The fifth section of the script performs successive sweep scans. Note that the laser start and stop frequencies can be adjusted if preferred (for instance, if a different QCL module is used). Pauses are used during the laser scan to ensure the capture buffer is filled and re-equilibrate the sample temperature, and to send the data capture buffer to the host computer. Our default is to use a 2-second pause during and after the scan to fill the data capture buffer, then a 1-second pause to accommodate transfer of the data capture buffer to the host computer. These defaults may be shortened slightly if preferred.

The sixth section of the script disarms and disconnects the laser from the host computer.

The seventh section of the script processes the data sent in the capture buffers to the host computer, in multiple steps. These steps are modfied from the methods described by Alcaraz et al., 2015. Anal. Chem.
1. Phase correction for cases where a significant portion of the data is stored in the y-cartesian coordinate, by minimizing the expression x*sin(p)-y*cos(p) in terms of the phase offset angle p. In cases where the phase correction is small, this step may be skipped by simply setting p to 0. Note that the reference and balance channels are usually phase-flipped compared to the signal channels.
2. The frequency axis is recalibrated according to the methods described in Mukherjee et al. This recalibration equation may be different for various instruments and results in a small difference in the frequency axis.
3. A 3rd-order, 41-datapoint Savitsky-Golay filter is applied to the data. Parameters of the filter may be adjusted as preferred; we found this combination optimal for the types of signals of interest in our experiments.
4. Anomalous spectra are removed using similarity indices using the method of Alcaraz et al., in our case using cutoff parameters of 0.96 for the mean across spectra and 0.8 for the product (a combination which we found empirically optimal).
5. The replicate spectra are averaged.
6. A sharp Fourier filter with Blackman-Harris window is applied to the data to remove high-frequency components (including etalon fringes and noise), using default order 400 long-pass filter, cutoff freequency 1000 Hz. The parameters may be adjusted as desired.
7. Spectra in terms of voltage (V) were converted to power using calibration curves of the form Power(V)=a*V/(1-b*V), where the parameters a and b were determined using an external calibration with a thermal power meter. These parameters are slightly different betweent the two MCT detectors on the balanced instrument and are likely to be different from one instrument to the next, requiring recalibration. The purpose is to overcome nonlinearity associated with the instrument response function at higher incident powers.

The eight and final section of the script converts the collected power spectra into absorbance spectra using the balanced detection.
