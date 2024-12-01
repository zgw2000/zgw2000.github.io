---
title: "The contagious analysis of Preferential Attachment and Small-world networks based on the SIR model"
excerpt: "The number of confirmed and suspected cases in many European countries has risen rapidly since the outbreak of the new coronavirus. At present, no good treatment measures have been found, how to take effective preventive and control measures quickly is still an urgent task. Based on the infectious disease dynamics SIR model, this study studies the rate of spread of different intensity prevention and control measures in the two network models and provides suggestions for the policy makers of a new round of prevention and control strategies.<br/>![SIR](/images/model4.gif){: .align-center}"
collection: portfolio
---
<a id="top"></a>

***Guowang Zeng***

## Introduction

The public announcement of the outbreak of Coronavirus, 2019-nCov, Covid-19, NCP, SARS-Cov-2 on January 21, 2019, followed by the release of a large number of outbreak data showed the seriousness of the spread of the epidemic and caused a high degree of 20 world concern. Since then, a series of outbreak control measures, such as strengthening medical treatment, isolation, strengthening medical resources, research and treatment methods and vaccines, as well as later city closure and other prevention and control strategies, are related to the future development of the epidemic. At present, the epidemic is still developing, the national rescue and control operations are being carried out on a larger scale, the future trend forecast of the development of the epidemic is also playing a key role in the control of the epidemic on a larger scale. The epidemic spread model of infectious diseases is a traditional epidemiological and mathematical problem, which has important practical value. In recent years, the frequent outbreaks of large-scale outbreaks around the world have aroused the high attention of academic circles and all sectors of society to the transmission model and its application, and there have been some innovations in methods, which have played an important role in epidemic control, rescue operations, resource allocation, transportation, travel planning and so on. In the world's intellectual city construction and other technological innovation activities, combined with the core functions of the city to carry out research on the spread of infectious diseases model, can deepen the digital city model, improve the management level, improve emergency response capacity, is an important content of urban construction and development. This research can be based on the continuous maturity, combined with the future model of urban and social development, to improve the level of urban emergency management, to ensure the ultimate realization of the sustainable development goals of mankind.

## Methodology

Infectious diseases are a long-standing social and natural phenomenon that has become a turning point or a trigger point for some important changes in human history. It is for these reasons that the model of the spread of infectious diseases is also familiar to us and one of the important achievements of mankind in the prevention and control of infectious diseases. As a social phenomenon, many assumptions are usually made in research, but their effectiveness and rationality have been proven and play an important role in the prevention and control of infectious diseases worldwide today. Here we use the simplest possible model to meet current rapid estimates of the number of infections and overall trends in outbreaks, to help develop accurate patient assistance measures and outbreak control strategy studies.

## Models

**SIR model:**
<br/>
The SIR model we are familiar with is based on the following relationship between total number **N**, number of infections **I**, susceptible number **S**, number of patients recovered **R** and time t in endemic areas:

![SIR](/images/sir.png){: .align-center}

$$\frac{dS}{dt}=-\beta\frac{IS}{N}$$

$$\frac{dI}{dt}=\beta\frac{IS}{N}-\gamma I$$

$$\frac{dR}{dt}=\gamma I$$

The parameters in these equations β and γ constants, reflecting the characteristics of a particular outbreak.
<br/>
Early in the spread of the epidemic, since the first susceptible population is the total number of people, namely S ≈ N, we can simplify the relationship between the number of infected people I and time t:

$$\frac{dI}{dt}=\beta\frac{IS}{N}-\gamma I\approx\left(\beta-\gamma\right)I$$

such that:

$$\left(\beta-\gamma\right)>0\rightarrow\beta>\gamma\rightarrow\frac{\beta}{\gamma}>1$$

This gives an approximate solution to the number of infected persons:

$$I\left(t\right)=e^{(\beta-\gamma)t}$$

This relationship suggests that the approximate total number of infections is an exponential function of time. The constants and β here γ be determined according to the characteristics of the outbreak, such as the different networks (Preferential Attachment and Small-world), the intensity of the virus infection per se and the probability of exposure to infected persons and others. So that it can achieve an estimate of the number of people infected. Of course, epidemic prevention and control measures will also affect these parameters, which in turn reflect the effectiveness of prevention and control measures. These parameters are generally based on epidemiological statistics and are reflected during the epidemic. In other words, we can also determine these parameters based on the actual outbreak report.
<br/>
So we want the quantity Beta/Gamma to be less than 1 if we want the epidemic to slow down. Note that this ratio has a very important meaning in epidemiology: it is the **basic reproductive number or R0**, and it indicates the average number of infected by an infectious individual in a totally susceptible population. In the literature we found out that the following holds: If R0 < 1, then with probability 1, the disease dies out after a finite number of waves. If R0 > 1, then with probability greater than 0 the disease persists by infecting at least one person in each wave.

`R0 > 1: diffusion of infection (worst case)`
`R0 < 1: extinction of infection (best case)`

**Preferential Attachment network:**
<br/>
A preferential attachment process is any of a class of processes in which some quantity, typically some form of wealth or credit, is distributed among a number of individuals or objects according to how much they already have, so that those who are already wealthy receive more than those who are not. "Preferential attachment" is only the most recent of many names that have been given to such processes. They are also referred to under the names "Yule process", "cumulative advantage", "the rich get richer", and, less correctly, the "Matthew effect". The application of preferential attachment to the growth of the World Wide Web was proposed by Barabási[^1] and Albert[^2] in 1999. Barabási and Albert also coined the name "preferential attachment" by which the process is best known today and suggested that the process might apply to the growth of other networks as well.
<br/>
<br/>
The network begins with an initial connected network of m0 nodes. New nodes are added to the network one at a time. Each new node is connected to m ≤ 𝑚0 existing nodes with a probability that is proportional to the number of links that the existing nodes already have. Formally, the probability pi that the new node is connected to node i is:

$$p_i=\frac{k_i}{\sum_{j} k_j}$$

where ki is the degree of node and the sum is made over all pre-existing nodes j (i.e. the denominator results in twice the current number of edges in the network). Heavily linked nodes ("hubs") tend to quickly accumulate even more links, while nodes with only a few links are unlikely to be chosen as the destination for a new link. The new nodes have a "preference" to attach themselves to the already heavily linked nodes.
<br/>
Note that the preferential attachment mechanism is the one used to generate the so- called scale-free networks. A scale-free network is a network whose degree distribution follows a power law. That is, the fraction P(k) of nodes in the network having k connections to other nodes goes for large values of k as:

$$P\left(k\right)~k^{-\alpha}$$

where alpha is a parameter whose value is typically in the range (2,3).
<br/>
The idea of scale-free networks is that, within the system, there are few important nodes that give robustness to the structure, while the majority of nodes are not as indispensable.
<br/>
<br/>
**Small-world network:**
<br/>
A small-world network is a type of mathematical graph in which most nodes are not neighbors of one another, but the neighbors of any given node are likely to be neighbors of each other and most nodes can be reached from every other node by a small number of hops or steps. Specifically, a small-world network is defined to be a network where the typical distance L between two randomly chosen nodes (the number of steps required) grows proportionally to the logarithm of the number of nodes N in the network, that is:

$$L\propto logN$$

A certain category of small-world networks was identified as a class of random graphs by Duncan Watts and Steven Strogatz[^3] in 1998. They noted that graphs could be classified according to two independent structural features, namely the clustering coefficient, and average node-to-node distance (also known as average shortest path length). Purely random graphs built according to the Erdős–Rényi (ER) model, exhibit a small average shortest path length (varying typically as the logarithm of the number of nodes) along with a small clustering coefficient. Watts and Strogatz measured that in fact many real-world networks[^4] have a small average shortest path length, but also a clustering coefficient significantly higher than expected by random chance. Watts and Strogatz then proposed a novel graph model, currently named the Watts and Strogatz model, with (i) a small average shortest path length, and (ii) a large clustering coefficient. 

## Assumption

The “lab” environment I’m going to consider is a population of 100 individuals, numbered from 0 to 99. Regarding the type of network, I decided to use the “small world” network represents the trajectory of human activity in a city, while “Preferential attachment” network” represents the trajectory of human keeping in social distance when m=1 or gathering activity in a city when m=3 (bar, parade, concert, club, football match, etc).
<br/>

| ![model1](/images/model1.png) | ![model2](/images/model2.png) | ![model3](/images/model3.png) |
|:---:|:---:|:---:|
| Figure 1: Watts Strogatz graph (small-world). With n=100, k=4, p=0.6 | Figure 2.1: Barabasi Albert graph. With n=100, m=1 | Figure 2.2: Barabasi Albert graph. With n=100, m=3 |

<br/>
**Hypothesis1 (Diffusion of infection):**
<br/>
Fixed R0[^5] > 1 (Beta/Gamma stay constant), set 1 infected individual randomly. The propagation speed of the Preferential Attachment model will be higher than that of the Small-world model when m increased. Namely, the PA model of contagious will strengthen the rate of transmission of the virus when m increased. We set an assumption that with Barabasi Albert graph when n=100, m=1 as a situation that each individual is keeping social distance and lockdown measures. And m=3 regard as high clustering activities. While small-world model with k=4 and p=0.6 regard as the situation closest to reality which stands for each individual has most 4 connected relations (parents and best friends).
<br/>
<br/>
**Hypothesis2 (Extinction of infection):**
<br/>
Fixed R0 < 1 (Beta/Gamma stay constant), set 10 infected individuals randomly. The reduction of R0 can effectively reduce the rate of infection of the disease. When R0 <1, the number of people infected reaches to a peak and gradually vanished. There are several measures can reduce Beta and increase Gamma, such as giving better treatment to the patient and wearing masks etc.

## Results

**Diffusion of infection:**
<br/>
By using the EoN (Epidemics on Networks) module in Python3 to apply the SIR model simulation.

![model1](/images/model1.gif){: .align-center}

<div align="center">Barabasi Albert graph. With n=100, m=1. Iteration 0, iteration 5, iteration 10 and iteration 15</div>
<br/>
As we can see, from iteration 0 to iteration 5, The spread of the virus has slowed down. At the final iteration, there are still certain proportion of 100 individuals remain susceptible and the virus has stopped spreading.

![model2](/images/model2.gif){: .align-center}

<div align="center">Watts Strogatz graph (small-world). With n=100, k=4, p=0.6, set 1 infected node randomly. Iteration 0, iteration 1, iteration 2 and iteration 3</div>
<br/>
As we can see, from the SIR curve on the right, the spread of the virus is exponential, and under this model, almost 95% of the 100 individuals have been infected at iteration 3.

![model3](/images/model3.gif){: .align-center}

<div align="center">Barabasi Albert graph. With n=100, m=3, set 1 infected node randomly. Iteration 0, iteration 1, iteration 2 and iteration 3</div>
<br/>
As we can see, under this model, the virus spreads very quickly. All of them were infected at iteration 2, and the SIR curve on the right, the spread of virus is dramatically increasing.
<br/>

**Extinction of infection:**
<br/>
By using the EoN (Epidemics on Networks) module in Python3 to apply the SIR model simulation.

![model4](/images/model4.gif){: .align-center}

<div align="center">Watts Strogatz graph (small-world). With n=100, k=4, p=0.6, set 10 infected nodes randomly. Iteration 0, iteration 1, iteration 4 and iteration 8</div>
<br/>
As we can see, at iteration 1, the infected individuals reached to a peak. After iteration 1, the infected individuals began to decrease. No more infected individuals at iteration 8 and remain large proportion of 100 individuals remain susceptible and the virus has stopped spreading.

## Conclusion
> Unsurprisingly, the diffusion of infection shows that preventive measures like social distancing and lockdown are indeed effective. The Barabasi Albert graph with the lower m value is the ideal scenario to handle a pandemic with a high infection rate. The nodes have low connectivity, but they’re still connected and somewhat clustered. Simply put, the majority is following the social distancing guidelines and distancing their groups (family, workplace, dormitory, etc)
> 
> On top of that, the Watts Strogatz model (small-world) is the normal interactions, maybe with basic preventive measurements advised, but not mandate. The spread is slower compared to the prior model, but we also notice a very important point in utilizing them: They are only effective when everyone’s following them. When people don’t respect the guidelines given by health officials, they are also undermining the efforts of others who follow them.
We can consider the Barabasi Albert graph with the larger m value an environment where people are actively socializing. This could simulate an area with a younger demographic, where the nightlife and events are more extensively held. All the nodes are highly connected, and the spread of the virus is exponential. It only took 1 iteration for the virus to infect the majority of the population.
> 
> The extinction of infection model simulated by the EoN package can be considered the best-case scenario for this pandemic. Even with relative connectivity between nodes, the speed of the spread is slowed down; and at the end, when the infection stopped, there still exists susceptible nodes. These nodes are the product of all our efforts in applying preventive measurements. On top of protecting normal people, we’re also sheltering vulnerable individuals from Covid. Government and policymakers can consider applying the mentioned measures from the above assumptions.

## Reference

[^1]: Barabási, A.-L.; R. Albert (1999). "Emergence of scaling in random networks". Science. 286 (5439): 509–512.
[^2]: Albert, Réka; Barabási, Albert-László (2002). "Statistical mechanics of complex networks” Reviews of Modern Physics. 74 (47): 47–97.
[^3]: Watts DJ, Strogatz SH (June 1998). "Collective dynamics of 'small-world' networks". Nature. 393 (6684): 440–2.
[^4]: Networks, Crowds, and Markets: Reasoning about a Highly Connected World” by D. Easley and J. Kleinberg. Cambridge University Press, 2010.
[^5]: Paul L. Delamater, corresponding author Erica J. Street, Timothy F. Leslie, Y. Tony Yang, and Kathryn H. Jacobsen. "Complexity of the Basic Reproduction Number (R0)"



[<center>↑ BACK TO TOP ↑</center>](#top)

  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
  <script>
    $(document).ready(function() {
      $('a[href="#top"]').click(function() {
        $('html, body').animate({ scrollTop: 0 }, 'slow');
        return false;
      });
    });
  </script>
