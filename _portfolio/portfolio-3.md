---
title: "The contagious analysis of Preferential Attachment and Small-world networks based on the SIR model"
excerpt: "The number of confirmed and suspected cases in many European countries has risen rapidly since the outbreak of the new coronavirus. At present, no good treatment measures have been found, how to take effective preventive and control measures quickly is still an urgent task. Based on the infectious disease dynamics SIR model, this study studies the rate of spread of different intensity prevention and control measures in the two network models and provides suggestions for the policy makers of a new round of prevention and control strategies.<br/>![SIR](/images/sir.gif){: .align-center}"
collection: portfolio
---
<a id="top"></a>

**Guowang Zeng**

## Introduction

```
The public announcement of the outbreak of Coronavirus, 2019-nCov, Covid-19, NCP, SARS-Cov-2 on January 21, 2019, followed by the release of a large number of outbreak data showed the seriousness of the spread of the epidemic and caused a high degree of 20 world concern. Since then, a series of outbreak control measures, such as strengthening medical treatment, isolation, strengthening medical resources, research and treatment methods and vaccines, as well as later city closure and other prevention and control strategies, are related to the future development of the epidemic. At present, the epidemic is still developing, the national rescue and control operations are being carried out on a larger scale, the future trend forecast of the development of the epidemic is also playing a key role in the control of the epidemic on a larger scale. The epidemic spread model of infectious diseases is a traditional epidemiological and mathematical problem, which has important practical value. In recent years, the frequent outbreaks of large-scale outbreaks around the world have aroused the high attention of academic circles and all sectors of society to the transmission model and its application, and there have been some innovations in methods, which have played an important role in epidemic control, rescue operations, resource allocation, transportation, travel planning and so on. In the world's intellectual city construction and other technological innovation activities, combined with the core functions of the city to carry out research on the spread of infectious diseases model, can deepen the digital city model, improve the management level, improve emergency response capacity, is an important content of urban construction and development. This research can be based on the continuous maturity, combined with the future model of urban and social development, to improve the level of urban emergency management, to ensure the ultimate realization of the sustainable development goals of mankind.
```

## Methodology

Infectious diseases are a long-standing social and natural phenomenon that has become a turning point or a trigger point for some important changes in human history. It is for these reasons that the model of the spread of infectious diseases is also familiar to us and one of the important achievements of mankind in the prevention and control of infectious diseases. As a social phenomenon, many assumptions are usually made in research, but their effectiveness and rationality have been proven and play an important role in the prevention and control of infectious diseases worldwide today. Here we use the simplest possible model to meet current rapid estimates of the number of infections and overall trends in outbreaks, to help develop accurate patient assistance measures and outbreak control strategy studies.

## Models

![SIR](/images/sir.png){: .align-center}

$$\frac{dS}{dt}=-\beta\frac{IS}{N}$$

$$\frac{dI}{dt}=\beta\frac{IS}{N}-\gamma I$$

$$\frac{dR}{dt}=\gamma I$$

The parameters in these equations β and γ constants, reflecting the characteristics of a particular outbreak.
Early in the spread of the epidemic, since the first susceptible population is the total number of people, namely S ≈ N, we can simplify the relationship between the number of infected people I and time t:

$$\frac{dI}{dt}=\beta\frac{IS}{N}-\gamma I\approx\left(\beta-\gamma\right)I$$

such that:

$$\left(\beta-\gamma\right)>0\rightarrow\beta>\gamma\rightarrow\frac{\beta}{\gamma}>1$$

This gives an approximate solution to the number of infected persons:

$$I\left(t\right)=e^{(\beta-\gamma)t}$$

This relationship suggests that the approximate total number of infections is an exponential function of time. The constants and β here γ be determined according to the characteristics of the outbreak, such as the different networks (Preferential Attachment and Small-world), the intensity of the virus infection per se and the probability of exposure to infected persons and others. So that it can achieve an estimate of the number of people infected. Of course, epidemic prevention and control measures will also affect these parameters, which in turn reflect the effectiveness of prevention and control measures. These parameters are generally based on epidemiological statistics and are reflected during the epidemic. In other words, we can also determine these parameters based on the actual outbreak report.
So we want the quantity Beta/Gamma to be less than 1 if we want the epidemic to slow down. Note that this ratio has a very important meaning in epidemiology: it is the **basic reproductive number or R0**, and it indicates the average number of infected by an infectious individual in a totally susceptible population. In the literature we found out that the following holds: If R0 < 1, then with probability 1, the disease dies out after a finite number of waves. If R0 > 1, then with probability greater than 0 the disease persists by infecting at least one person in each wave.

`R0 > 1: diffusion of infection (worst case)`
`R0 < 1: extinction of infection (best case)`





$$p_i=\frac{k_i}{\sum_{j} k_j}$$

$$P\left(k\right)~k^{-\alpha}$$

$$L\propto logN$$








> Click to see more info at [`zgw2000/risc_game`](https://github.com/zgw2000/risc_game "see it on Github").

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
