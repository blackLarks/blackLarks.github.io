---
title: Digital Communication System Simulation (Basics)
date: 2025-02-03 23:27:50
tags: Theory
---

This blog is my notes about how to conduct a digital communication system simulation in Matlab. My Matlab version is R2023a. Two textbooks that I recommend here are *Digital Communications* by Proakis and *Wireless Communication Systems in Matlab* by Mathuranathan Viswanathan. You can also follow the MIT open course [Principle of Digital Communication](https://ocw.mit.edu/courses/6-451-principles-of-digital-communication-ii-spring-2005/pages/readings-and-lecture-notes/).

<figure id="fig1" style="text-align: center">   
    <img src="/img/CS/CS.svg" alt="Digital Communication System" style="width: 90%;">   	
    <figcaption>
    Figure 1: Block diagram of a typical digital communication system
    </figcaption> 
</figure>  

<a href="#fig1">Figure 1</a> shows the basic structure of a digital communication system (Proakis). In this blog, I skip the Source encoder, Channel encoder and the corresponding decoders. For simplexity, I assume that the information source generates random sequence binary bits, the channel is Additive white Gaussian noise (AWGN) Channel and QPSK with Gray labelling is applied (<a href="#fig2">Figure 2</a>).

## Brief Principle

Here I give some brief explanations and my intuitions about the communication system. I will skip the concepts about the signal vector space (Proakis Ch2.2).

### Constellation Diagram

**<font color= #d44375>Intuition:</font>** By grouping bits into symbols (like QPSK's 2 bits per symbol), we can transmit more bits in the same time period. Symbols allow us to encode multiple bits using phase and amplitude, which can be transmitted at lower frequencies. 

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

### AWGN Channel

**<font color= #d44375>Intuition:</font>** The reason for AWGN channel assumption is that AWGN is usually the dominant noise in the real-world communication systems. This assumption offers a simple model for analyzing the baseline performance of more complex communication systems.

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
\mathbf{n} = n_I+ j\cdot n_Q
$$

where $n_I$ and $n_Q$ are the real and imaginary parts of the noise, which are independent and identically distributed (i.i.d.) [Gaussian random variables](https://en.wikipedia.org/wiki/Gaussian_random_variable) with zero mean and variance $N_0/2$. Actually, the noise $\mathbf{n}$ is of infinite dimensions, but due to the orthogonality of the signal space, only the projection on the signal space, i.e., the constellation diagram, is needed.

### Optimum Receiver

**<font color= #d44375>Intuition:</font>** We always want to find the optimum receiver to maximize the probability of correct decision, like a detective trying to reconstruct what happened based on available evidence. This is also the essence intuition behind our conditional probability model for the channels.

For a general vector channel model, the optimum receiver is the one that maximizes the probability of correct decision, which is given by

$$
P_c = \Pr (\mathbf{u} = \hat{\mathbf{u}})
    = \int P[\mathbf{u} = \hat{\mathbf{u}}|\mathbf{r}] p(\mathbf{r}) d\mathbf{r}
$$

where $\mathbf{u}$ is the information bits and $\hat{\mathbf{u}}$ is the detected (or decoded) signal (<a href="#fig1">Figure 1</a>).

Since $\mathbf{r}$ is oberserved and nonnegative, we can express the optimal detector as 

$$
\hat{\mathbf{u}} = \arg \max_{\mathbf{u}} P[\mathbf{u}|\mathbf{r}]
$$

Equivalently, due to the one-to-one mapping between $\mathbf{u}$ and $\mathbf{s}$, we can express the optimal detector as

$$
\hat{\mathbf{s}} = \arg \max_{\mathbf{s}} P[\mathbf{s}|\mathbf{r}]
$$

These detectors are called **Maximum A Posteriori Probability (MAP)** rule. 

Apply Bayes' theorem, we have

$$
\hat{\mathbf{s}} = \arg \max_{\mathbf{s}} \frac{P[\mathbf{r}|\mathbf{s}]P[\mathbf{s}]}{P[\mathbf{r}]} = \arg \max_{\mathbf{s}} P[\mathbf{r}|\mathbf{s}]P[\mathbf{s}]
$$

If we further assume that all symbols are equally likely, we have the **Maximum Likelihood (ML)** rule, which is given by

$$
\hat{\mathbf{s}} = \arg \max_{\mathbf{s}} P[\mathbf{r}|\mathbf{s}]
$$

**Remark:** The MAP and ML rules are all general rules, not necessary for AWGN channel and QPSK.

### Decision Region

**<font color= #d44375>Intuition:</font>** The decision region, in the constellation diagram perspective, is how we visualize the process of the detector in the vector space. Inituitively, we would think that the further a received signal is from a transmitted point on the constellation diagram, the lower the probability that this received point was actually transmitted from that constellation point.

The detector, not necessary MAP and ML detector, will partitions the signal space into decision regions based on the corresponding rule. 

For QPSK and AWGN channel, the decision regions ($\Gamma_{i}$, $i=1,2,3,4$) and the received signal (red dots) examples are shown in <a href="#fig3">Figure 3</a>. 

<figure id="fig3" style="text-align: center">   
    <img src="/img/CS/reMapper.svg" alt="QPSK Decision Region" style="width: 60%;">   	
    <figcaption>
    Figure 3: QPSK Decision Region
    </figcaption> 
</figure>  

Stop here and take a rethinking. Why we can represent the received signal as a single point on the constellation diagram? The reason is that , for AWGN channel, only the projection on the signal space, i.e., the constellation diagram, is needed. For point $A$ on the output space, the detector will think the corresponding transmitted signal is $00$, while it is of high probability that the transmitted signal is actually $10$ due to the noise (But the transmitted signal can, of course, be any other symbol! Here for the detector, we can only use the word **"the most likely one"** to describe the transmitted signal).

### Optimum Receiver for AWGN Channel

For AWGN channel, we can apply the memoryless property of the channel, MAP ruleand the fact that the noise is Gaussian to derive the optimum receiver.

$$
\hat{s} = \arg \max_{s} (P[r|s]P[s]) = \arg \max_{s} \left(\ln P[s] - \frac{|r-s|^2}{2\sigma^2}\right)
$$

Note that here we use the expression of the PMF of AWGN channel:

$$
p(r|s) = \frac{1}{\sqrt{2\pi\sigma^2}} e^{-\frac{(r-s)^2}{2\sigma^2}}
$$

If we assume that all symbols are equally likely, we can simplify the expression to get the minimum Euclidean distance rule:

$$
\hat{s} = \arg \min_{s} |r-s|^2,
$$

which is the same as our intuition in <a href="#fig3">Figure 3</a>.

## Monte Carlo method

**<font color= #d44375>Intuition:</font>** For simulation, we need to get the probability of error, say Bit Error Rate, we can do simulation with random transmitted signals and the random noise. Additionally, we always want to get the significant result with statistical confidence, so we can think of the method that we collect enough errors and do more trials to get the stable result. 

The basic performance is Bit Error Rate (BER), which can be expressed as $\Pr [\mathbf{u} \neq \hat{\mathbf{u}}]$. In an implementation point-of-view, we need to find an approach with statistical confidence for simulation, taking into account the random transmitted signals and the random noise. 

The Signal-to-Noise Ratio (SNR) is a fundamental parameter for the tramsmitted signal because it can affect the answer of this question **"how much can the noise affect the received signal"**.

An excellent solution is [Monte Carlo method](https://en.wikipedia.org/wiki/Monte_Carlo_method) below. With the simulated results, we can conduct performance verification by comparing simulated and theoretical results.

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
- An outer loop that tests different SNR ($E_s/N_0$) points
- An inner Monte Carlo loop that runs until we have statistical confidence in our results

A critical part for Monte Carlo method is to specify the maximum number of errors we want to collect. Undoubtedly, the more errors we collect, the more confident we are about the result. However, it is not realistic to set the value too high because the simulation time will be too long. 

We will discuss the simulation more in the next blog.

Any contributive advice is welcome by email.

