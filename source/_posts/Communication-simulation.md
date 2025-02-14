---
title: Digital Communication System Simulation (QPSK)
date: 2025-02-14 20:47:00
tags: Theory
---

This blog is my second blog about digital communication system simulation. In this blog, I will simulate a QPSK uncoded communication system (AWGN channel) with Matlab. I try to do it step by step following the Monte Carlo method.

```algorithm
Algorithm: MonteCarloSimulation
Input: simulationParameters, systemParameters
Output: simulationResults

1: Initialize variables:
   errorCount ← 0
   processedBits ← 0

2: for each SNR in snrRange do
       while errorCount < minErrorEvents do
           data ← GenerateRandomBits(blockSize)
           modulatedSignal ← ModulateData(data)
           receivedSignal ← SimulateChannel(modulatedSignal, SNR)
           decodedData ← DemodulateSignal(receivedSignal)
           errorCount ← CountErrors(data, decodedData)
       end while
       
       BER[SNR] ← errorCount/processedBits
   end for

3: return BER
```

## Steps of the simulation

### GenerateRandomBits

It is straightforward to generate random bits with Matlab with the function *randi*, which can generate uniformly distributed random integers. Here I use the row vector form to represent the information bits.

```matlab
transmittedBits = randi([0, 1], 1, numBitsPerBlock);
```

### ModulateData

In this step, we need to map the information bits to the QPSK signal. The details of QPSK can be found in the previous blog.

```matlab
% QPSK Modulator
% Gray-coded QPSK symbol mapping
% Maps bit pairs to complex constellation points
inPhaseData = transmittedBits(1:2:end);    % I-channel bits
quadData = transmittedBits(2:2:end);       % Q-channel bits
qpskSymbols = (1 - 2*inPhaseData) + 1j * (1 - 2*quadData);
```

### SimulateChannel

Here we need to simulate the complex AWGN channel. The mathematical model is easy to understand but there are some details to be careful in the implementation.

$$
\text{SNR}, \text{signalPower} \to N_0 \to \text{receivedSignal}
$$

In order to make the awgn channel general for both normalized and non-normalized signal power, we need to calculate the signal power first. Then we can calculate the noise power spectral density $N_0$ from the SNR (linear scale) and the average signal power. After that, we need to generate the complex Gaussian noise with the variance of $\sqrt{\frac{N_0}{2}}$ and add it to the transmitted signal to get the received signal.

```matlab
function receivedSignal = add_awgn_noise(transmittedSymbols, signalToNoiseRatioDb)
% add_awgn_noise Simulates an Additive White Gaussian Noise (AWGN) channel
%
% This function models the effects of thermal noise in a communication channel
% by adding complex Gaussian noise to the transmitted signal. The noise power
% is calculated based on the desired signal-to-noise ratio (SNR).
%
% Theory:
%   - AWGN is a fundamental channel model in communications
%   - The noise is "additive" because it's added to the signal
%   - "White" means the noise has uniform power across all frequencies
%   - "Gaussian" refers to the normal distribution of noise amplitudes
%
% Inputs:
%   transmittedSymbols  - Complex baseband symbols (e.g., QPSK constellation points)
%   signalToNoiseRatioDb - Energy per symbol to noise density ratio (Es/N0) in dB
%
% Output:
%   receivedSignal     - Transmitted symbols corrupted by AWGN
%
% Example:
%   receivedSignal = addAwgnChannel(qpskSymbols, 10) % Add noise at 10dB Es/N0

    % Convert signal-to-noise ratio from dB to linear scale
    % SNR(linear) = 10^(SNR(dB)/10)
    signalToNoiseRatioLinear = 10^(signalToNoiseRatioDb/10);
    
    % Calculate average signal power
    % P = (1/N) * Σ|x[n]|²
    symbolCount = length(transmittedSymbols);
    signalPower = sum(abs(transmittedSymbols).^2) / symbolCount;
    
    % Calculate noise power spectral density (N0)
    % Using the relation: Es/N0 = SignalPower/NoisePower
    noiseDensity = signalPower / signalToNoiseRatioLinear;
    
    % Generate complex Gaussian noise
    % - Real and imaginary parts are independent
    % - Each component has variance = noiseDensity/2
    % - Factor of 1/2 ensures total noise power equals noiseDensity
    noiseStdDev = sqrt(noiseDensity/2);
    noiseReal = noiseStdDev * randn(size(transmittedSymbols));
    noiseImag = noiseStdDev * randn(size(transmittedSymbols));
    noiseComplex = noiseReal + 1j*noiseImag;
    
    % Add noise to transmitted signal
    % Received signal = transmitted signal + noise
    receivedSignal = transmittedSymbols + noiseComplex;
    
    % Note: The received signal now has the following properties:
    % 1. Signal component preserved exactly
    % 2. Noise component is complex Gaussian
    % 3. Signal-to-noise ratio matches specified Es/N0
end
```	

### DemodulateSignal
For QPSK, the demodulation is very simple. We just need to check the real and imaginary parts of  the received signal and make a decision.

```matlab
% QPSK Demodulator
% Optimal detection for AWGN channel
detectedInPhase = real(receivedSymbols) < 0;  % I-channel detection
detectedQuad = imag(receivedSymbols) < 0;     % Q-channel detection

% Reconstruct bit stream from symbol decisions
detectedBits = reshape([detectedInPhase; detectedQuad], 1, []);
```

### CountErrors

This is the **<font color= #d44375>core part</font>** of the Monte Carlo method. In the Monte Carlo iteration **while** loop, we need to count the number of errors by comparing the information bits and the detected (demodulated) bits in a block-by-block manner. 

If the accumulated errors is enough for our simulation or we have processed enough blocks, we can stop the iteration and calculate the final simulated Bit Error Rate (BER).

**<font color= #d44375>Remark:</font>** By using Monte Carlo method, we can get the average BER of our communication system. The results is of statistical confidence if we have enough error events and trials.

````matlab
% ...inside the Monte Carlo iteration loop...
% Error Analysis
% Bit error counting
blockErrors = sum((transmittedBits ~= detectedBits));
accumulatedErrors = accumulatedErrors + blockErrors;
processedBits = processedBits + numBitsPerBlock;
% ...
end

% Calculate empirical BER for current Eb/N0
bitErrorRate(snrIndex) = accumulatedErrors/processedBits;
%...
```

## Theoretical BER

We want to validate our simulated results with the theoretical BER of QPSK. The theoretical BER of QPSK in AWGN channel is given by:

$$
P_e = Q\left(\sqrt{\frac{2E_b}{N_0}}\right)
$$

**<font color= #d44375>Remark:</font>** The BER is related the specific labelling scheme of the QPSK constellation. This BER is for the Gray-labelling scheme. For detailed derivation, please refer to Proakis's book Chapter 4.3.

In Matlab, we can use the function *qfunc* to denote the Q-function. The Theoretical BER is calculated as follows:

```matlab
% Theoretical Performance Bound
% Calculate theoretical QPSK BER in AWGN
% QPSK BER = BPSK BER due to orthogonal channels
ebNoLinear = 10.^(ebNoDbRange/10);                    % Convert dB to linear
theoreticalBer = qfunc(sqrt(2*ebNoLinear));           % Q-function for QPSK
```

Remember that when we calculate the theoretical BER, we need to convert the EbN0 from dB to linear. But for plotting, we can use the dB scale for better visibility.

## Plot Results

I personally think that a carefully designed plot is the most powerful way to visualize the simulation results. Here I use the semilogy function to plot the BER vs Eb/N0 in <a href="#fig1">Figure 1</a>.


```matlab
% Performance Visualization
figure('Position', [100 100 800 600]);

% Plot configuration
plotLineWidth = 2;
markerSize = 17;

% Monte Carlo simulation results
semilogy(ebNoDbRange, bitErrorRate, 'b.', ...
         'MarkerSize', markerSize, ...
         'DisplayName', 'Monte Carlo Simulation');
hold on;

% Theoretical bound
semilogy(ebNoDbRange, theoreticalBer, 'b-', ...
         'LineWidth', plotLineWidth, ...
         'DisplayName', 'Theoretical Bound');

% Plot aesthetics
grid on;
ylim([1e-4 1]);
xlim([min(ebNoDbRange) max(ebNoDbRange)]);
xlabel('$E_b/N_0$ (dB)', 'FontSize', 12, 'Interpreter', 'latex');
ylabel('Bit Error Rate (BER)', 'FontSize', 12, 'Interpreter', 'latex');
title('QPSK Performance Analysis: Monte Carlo vs Theoretical', ...
      'FontSize', 14, 'Interpreter', 'latex');
legend('Location', 'southwest', 'FontSize', 10);
legend('boxoff');
set(gcf, 'Color', 'white');
set(gca, 'FontSize', 11);
```

**<font color= #d44375>Remark:</font>** You can find the complete code in this [Github repository](https://github.com/blackLarks/Digital-Communication-Simulation).

The final BER plot (simulated v.s. theoretical) is shown as follows:

<figure id="fig1" style="text-align: center">   
    <img src="/img/CS/QPSK_Uncoded_BER.svg" alt="QPSK Uncoded System BER Performance" style="width: 60%;">   	
    <figcaption>
    Figure 1: QPSK Uncoded System BER Performance
    </figcaption> 
</figure>  

**<font color= #d44375>Remark:</font>** The simulation results are consistent with the theoretical bound. The Monte Carlo simulation is accurate and reliable.