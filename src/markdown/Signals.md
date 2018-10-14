## Overview
The signals team has two jobs - writing software to automatically analyze incoming data from the lab, and writing software to
generate signals in reaction to the incoming data.

## Input
Images are taken of bacterial samples in the lab - the challenge is to extract information about the populations from the colors seen
in the image. Since the LED we use to control the bacteria is blue, we will look at the 'redness' of the image rather than the 
amount of green, as blue light bleeds into the green spectrum a little bit.
In order to do this, we use scipy to read the rgb values off of the image and track the intensity of red. This provides us with a
good approximation of the fluorescence.

## Output

Model Predictive Control is an optimization technique where a mathematical model of the system (alonside data)
is used to generate a strategy to optimize a certain value. In order to keep the co-culture at a certain population ratio
we need to come up with a sequence of light signals that keep the bacteria growing at a certain pace.

We define a few parameters for this algorithm:

Manipulated variable - The parameter output by the program in order to control the ratio. In this case the MV is the light pulse width

Controlled variable - The parameter controlled by the program as a consequence of the MV value. In this case the CV is the population ratio.

The MV and CV are related by the following formulas:

![Formula1](http://latex2png.com/output//latex_d8c96bc3f73569bb6bf860bc0a18d7fe.png)

![Formula2](http://latex2png.com/output//latex_3081dcad4e3ae7dfd459bce86ea82bb2.png)

![Formula3](http://latex2png.com/output//latex_33768bfcf8de3d833674b7ce86171395.png)

1. A- lower asymptote of expected growth curve of light

2. K- upper asymptote of expected growth curve of light

3. C- typically 1

4. ν- controls behavior near the asymptote closest to the maximum growth

5. Q- related toY(0)

6. k- linear factor by which GFP and general protein growth are related

7. d- linear factor by which GFP decay and general protein usage are related

8. W- Stochastic Process for Noise (optional)

9. D- some function that determines the growth/denaturation of general proteins independent of GFP (optional)
