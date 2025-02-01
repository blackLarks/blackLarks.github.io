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

<a href="#fig1">Figure 1</a> shows the basic structure of a digital communication system (Proakis). In this blog, I skip the Source encoder and Channel encoder. For simplexity, I assume that the information source generates random sequence binary bits, the channel is Additive white Gaussian noise (AWGN) Channel and QPSK with Gray labelling is applied (<a href="#fig2">Figure 2</a>).

<figure id="fig2" style="text-align: center">   
    <img src="/img/CS/mapper.svg" alt="QPSK constellation diagram" style="width: 60%;">   	
    <figcaption>
    Figure 2: QPSK Constellation Diagram
    </figcaption> 
</figure>  

# Brief Principle

Here I give some brief explanations about the communication system.

## Mapping Bits to Symbols

As <a href="#fig2">Figure 2</a> shown, we can map every 2 consecutive bit sequence to a point on the constellation diagram. A confusing problem for me is about the normalization of the consetllation diagram.

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


