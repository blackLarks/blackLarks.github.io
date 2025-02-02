---
title: Digital Communication System Simulation (Basics)
date: 2025-01-28 09:27:50
tags: Theory
---

This blog is my notes about how to conduct a digital communication system simulation in Matlab. My Matlab version is R2023a. Two textbooks that I recommend here is *Digital Communications* by Proakis and *Wireless Communication Systems in Matlab* by Mathuranathan Viswanathan. You can also follow the MIT open course [Principle of Digital Communication](https://ocw.mit.edu/courses/6-451-principles-of-digital-communication-ii-spring-2005/pages/readings-and-lecture-notes/).

<figure id="fig1" style="text-align: center">   
    <img src="/img/CS/CS.svg" alt="Digital Communication System" style="width: 90%;">   	
    <figcaption>
    Figure 1: Block diagram of a typical digital communication system
    </figcaption> 
</figure>  

<a href="#fig1">Figure 1</a> shows the basic structure of a digital communication system (Proakis). In this blog, I skip the Source encoder, Channel encoder and the corresponding decoders. For simplexity, I assume that the information source generates random sequence binary bits, the channel is Additive white Gaussian noise (AWGN) Channel and QPSK with Gray labelling is applied (<a href="#fig2">Figure 2</a>).

# Brief Principle

Here I give some brief explanations about the communication system. I will skip the concepts about the signal vector space (Proakis Ch2.2).

## Constellation Diagram

As <a href="#fig2">Figure 2</a> shown, we can map a bit pair to a point on the constellation diagram, which is called *symbol*. Gray labelling is applied, which means that the bit sequence with the least Hamming distance is chosen.

The following Matlab code is used to map the bit sequence to the symbol.
```matlab
% QPSK Modulator
% Gray-coded QPSK symbol mapping
% Maps bit pairs to complex constellation points
inPhaseData = transmittedBits(1:2:end);    % I-channel bits
quadData = transmittedBits(2:2:end);       % Q-channel bits
qpskSymbols = (1 - 2*inPhaseData) + 1j * (1 - 2*quadData);
```

<figure id="fig2" style="text-align: center">   
    <img src="/img/CS/mapper.svg" alt="QPSK constellation diagram" style="width: 60%;">   	
    <figcaption>
    Figure 2: QPSK Constellation Diagram
    </figcaption> 
</figure>  

Suppose 00 is $(d,d)$, 01 is $(-d,d)$, 10 is $(d,-d)$, 11 is $(-d,-d)$. The average energy of the symbol is $E_s$. Then we can get the relationship between $E_s$ and $d$ as follows:

$$
\frac{2d^2 \times 4}{4} = E_s \implies d = \sqrt{\frac{E_s}{2}}
$$

**Remark:** If we set $E_s = 1$, which means we normalize the energy of the constellation diagram, then $d = \frac{1}{\sqrt{2}}$. Normalization can maintain the average energy of the symbol to be 1, no matter what the constellation diagram is, which is useful for performance comparison. But I prefer not to normalize and keep the notation $E_s$ for the energy of the symbol in order to keep the form of signal to noise ratio (SNR) as $E_s/N_0$.

## AWGN Channel
A discrete communication channel can be completely specified by the conditional probability mass function (PMF) $p(\mathbf{r}|\mathbf{s})$, i.e., the probability of the received signal $\mathbf{r}$ given the transmitted signal $\mathbf{s}$. If the channel is memoryless, the conditional probability mass function can be written as

$$
p(\mathbf{r}|\mathbf{s}) = \prod_{i} p(r_i|s_i).
$$

Here we consider the discrete memoryless AWGN channel. The received signal is given by

$$
\mathbf{r} = \mathbf{s} + \mathbf{n}
$$

where $\mathbf{s}$ is the transmitted signal, $\mathbf{n}$ is the complex Gaussian noise. The noise is given by

$$
\mathbf{n} = (n_I, n_Q)
$$

where $n_I$ and $n_Q$ are the real and imaginary parts of the noise, which are independent and identically distributed (i.i.d.) [Gaussian random variables](https://en.wikipedia.org/wiki/Gaussian_random_variable) with zero mean and variance $N_0/2$. 

## Optimum Receiver

The optimum receiver is the one that minimizes the probability of error. The probability of error is given by

$$
P_e = \Pr (\mathbf{u} \neq \hat{\mathbf{u}})
$$

where $\mathbf{u}$ is the transmitted signal and $\hat{\mathbf{u}}$ is the received signal.


# Monte Carlo method

The basic performance is Bit Error Rate (BER), which can be expressed as $\Pr (\mathbf{u} \neq \hat{\mathbf{u}})$. In an implementation point-of-view, we need to find an approach with statistical confidence for simulation, taking into account the random transmitted signals and the random noise. 

A good solution is [Monte Carlo method](https://en.wikipedia.org/wiki/Monte_Carlo_method) below. With the simulated results, we can conduct performance verification by comparing simulated and theoretical results.

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

The heart of the algorithm is the Main Simulation Loop, which operates at two levels:
- An outer loop that tests different SNR points
- An inner Monte Carlo loop that runs until we have statistical confidence in our results


