## Model Formulation

Our system utlizes the Ccas/Ccar System as an optogenetic switch. When exposed to green light, the Ccas photoreceptor transfers its phosphate group to its binding partner, the Ccar molecule. The Ccar molecule then induces the expression of the gene of interest. Since the expression of MetE is crucial to the production of methionine, it is the optimal choice for our gene of interest. The production of methionine allows the cell to function and profilerate. In order to accurately measure the rate of proliferation, we have artifically inserted GFP, whose levels will be determined by a camera.

![Reaction Network Diagram](http://2018.igem.org/wiki/images/8/89/T--Waterloo--Model-ReactionNetwork.png)

##### Formally, our network is equivalent to:

\\[ \frac{\textrm{CcaS}_{a}(t)}{dt} =  k_1 \ell(t) + k\textrm{Prolif}(t)\\]

<center>The subscript a indicates that we are tracking the rate of activation of the mentioned molecule.</center>

\\[ \frac{\textrm{CcaR}_{a}(t)}{dt} =  k_2 \textrm{CcaS}_{a} (H(t-\tau_1)\cdot |t|) + k\textrm{Prolif}(t)\\]

<center>Note that a Heaviside function is introduced to account for input-response delay in the network. Bacteria will not immediately respond to influence, and internal mechanisms will introduce delay.</center>

\\[\frac{d\textrm{Prom}_{a}}{dt} = \alpha_0 + \alpha \frac{\textrm{CcaR}_{a} (t)}{k + \textrm{CcaR}(t)}\\]

\\[ \frac{d\textrm{MetE}(t)}{dt} = k\textrm{Prom}_{a}(t) \\]

\\[\textrm{Logistic}(t) =  \left( A+\frac{K-A}{(C+Qe^{-\beta t})^{1/\nu}} \right)\\]

<center>All parameters excluding \\(t\\) (being time) are parameters which are up to experiment. Some parameters (such as C and Q) can be determined *a priori* based on the goals of our experiment. Q specifically is related to our boundary conditions, while C can simply be set to 1. </center>

\\[ \frac{d\textrm{MetE}(t)}{dt} = \textrm{Logistic}\_{1} (t) \cdot \textrm{CcaR}_{a}(H(t - \tau_2)\cdot |t|) \\]

\\[ \frac{d\textrm{Met}(t)}{dt} =  \textrm{Logistic}\_{2} (t) \cdot \textrm{MetE}(H(t- \tau_3)\cdot |t|) + E_1 (t)\cdot k_3 \\]

\\[ \frac{d\textrm{GFP}(t)}{dt} =  \textrm{Logistic}\_{3} (t) \cdot \textrm{Met}(H(t- \tau_4)\cdot |t|)+ E_2 (t)\cdot k_4 \\]

\\[ \frac{d\textrm{Prolif}(t)}{dt} =  \textrm{Logistic}\_{4} (t) \cdot \textrm{Met}(H(t- \tau_5)\cdot |t|)+ E_3 (t)\cdot k_5 \\]

\\[\frac{dE_i}{dt} = \textrm{Prolif}(t) \cdot k_i\\]

<center>\\(W(t)\\) is a stochastic process - introducing functions like these into the model allows us to account for random/chaotic motion.</center>

\\[\frac{dp_i}{dt}=(k-d)\textrm{GFP}(t) + \textrm{D}(t)\\]

\\[\frac{dR}{dt} = \frac{d}{dt} \frac{p_1 (t)}{p_2 (t)} \\]

<center>\\(\frac{dR}{dt}\\) is the ratio which we are trying to control.</center>

We determined that this is the most accurate reaction network that could be derived in theory (though it doesn't account for all noise).

## Moving Horizon Estimate (MHE)
MHE is a state estimation method that utilizes multiple measurements over time.  These measurements contain noise and other random variations.  However, MHE will allow for the production of estimates of unknown variables and parameters in the measurements despite noise.  MHE necessitates an iterative approach relying on either linear programming or nonlinear programming solvers to find solutions.  This method is particularly useful for nonlinear or constrained dynamic systems for which few general models with established properties and parameters are available.  

Since the optogenetic system we are utilizing is not yet generally modelled with established properties and not much data was available prior to experimentation, MHE was used in this project due to its capacity to estimate unknown variables and parameters during optimization.

MHE works by adjusting initial conditions and parameters within a model to align measured and predicted values.  To do this it uses an internal dynamic model, an optimization cost function over the estimation horizon, and requires a history of past measurements.

## Error Dynamics of a Linearization of Our Model

For a control system such as ours, the error dynamics are critical to derive. We will assume the following linearization of our model:

(insert equations)

(description of variables)

We will use the following constrained convex optimization formulation of MHE:

\\[\min_{\hat { x }_{T - N | T } , \hat { W } _ { T - N | T } ^ { T - 1 } } \| \hat { x } _ { T - N | T } - x _ { T - N | T } \| ^ { 2 } - \| Y _ { T - N } ^ { T - 1 } - \mathcal { O } \hat { x } _ { T - N | T } - \overline { c b } U _ { T - N } ^ { T - 2 } \|^2 + \sum _ { k = T - N } ^ { T - 1 } \left\| \hat { w } _ { k } \right\| ^ { 2 } + \sum _ { k = T - N } ^ { T } \left\| \hat { v } _ { k } \right\| ^ { 2 },\\]

such that  \\(\hat { x } _ { k + 1 } = A \hat { x } _ { k } + B u _ { k } + G \hat { w } _ { k } , \quad \hat { y } _ { k } = C \hat { x } _ { k } + \hat { v } _ { k }​\\),

where \\(T​\\) is the current time, \\(x,y,u​\\) are the population ratio, light measurement, and output signal vectors of the system, \\(w, v​\\) are the process disturbance noises and the measurement noise respectively,

\\[Y _ { T - N } ^ { T } = \left[ y _ { T - N } ^ { T } , \dots , y _ { T } ^ { T } \right] ^ { T }​\\]

is the vector containing the past \\(N​\\) inputs at time \\(T​\\) with the \\(U​\\) variant defined analagously, and \\(Q \succeq 0, R \succeq 0, P_{T-N|T-1} \succeq 0​\\) are the covariances of \\(w,v,x​\\) assumed to be symmetric and time invariant for the steady state MHE. The matrix \\(\mathcal{O}​\\) is defined in Tenney (2002).

The error dynamics of this model are as follows:

\\(e_{t+1} = \sum_{i} Error_{i}e_{t} + D_{i} d\\). With the first term being the estimation error, and \\(D_{i}d\\) being the error introduced by disturbing the system during measurement.

We have the following equations which were derived in Voelker (2013):

\\[Error_i = MS_i F_e M^{-1},\\]

\\[D_i = MS_i\begin{bmatrix} F_w & F_v \end{bmatrix} \begin{bmatrix} W \\\ V \end{bmatrix},\\]

\\[M = \begin{bmatrix} A^N & g_T \\\ 0 & I\end{bmatrix},\\]

\\[S_i = (I+H^{-1}\hat{K}^\top_i(\hat{K}_i H^{-1} \hat{K}_i^\top)^{-1})\hat{K}_i)^{-1} \cdot (-I + H^{-1}\hat{K}_i^\top (\hat{K}_iH^{-1}\hat{K}_i)^{-1}\hat{K}_i)H^{-1},\\]

\\[H \approx \begin{bmatrix} 2 P^{-1} + 2ca^\top\cdot \mathrm{diag}(R^{-1})\cdot ca & 2ca^\top \cdot \mathrm{diag}(R^{-1}) \\\ 2 \cdot \mathrm{diag}(R^{-1}) \cdot ca & 2\mathrm{diag}(Q^{-1})+\mathrm{diag}(R^{-1}) \end{bmatrix},\\]

\\[ca = \mathrm{diag}(C)\cdot a,\\]

\\[c g = \operatorname { diag } ( C ) \cdot g,\\]

\\[\overline{cb} = \mathrm{ diag } ( C ) \cdot \overline { b},\\]

\\[\overline { b } = \begin{bmatrix} { 0 } & { 0 } & { \cdots } & { 0 } \\\ { B } & { 0 } & { \cdots } & { 0 } \\\ { A \cdot B } & { B } & { \cdots } & { 0 } \\\ { A ^ { 2 } \cdot B } & { A \cdot B } & { \cdots } & { 0 } \\\ { \vdots } & { } & { \ddots } & { } \\\ { A ^ { N - 2 }  \cdot B } & { A ^ { N - 3 } } \cdot { B } & \cdots &  { B } \end{bmatrix}\\]



\\[\overline {g} = \begin{bmatrix} 0 &  0 & { \cdots } & { 0 } \\\ { G } & { 0 } & { \cdots } & { 0 } \\\ { A \cdot G } & { G } & { \cdots } & { 0 } \\\ { A ^ { 2 } \cdot G } & { A \cdot G } & { \cdots } & { 0 } \\\ { \vdots } & { } & { \ddots } & { } \\\ { A ^ { N - 2 } \cdot G } & { A ^ { N - 3 } \cdot G } & { \cdots } & { G } \end{bmatrix},\\]

\\[g = \begin{bmatrix} { } & { \overline { g } } & { } & { } \\\ { A ^ { N - 1 } \cdot G } & { \cdots } & { G } \end{bmatrix},\\]

\\[g_T = \begin{bmatrix} A^{N-1} \cdot G & \cdots & G \end{bmatrix},\\]

\\[F_w  = \begin{bmatrix} 0 \\\ 2 \cdot \mathrm{diag}(Q^{-1}) \end{bmatrix},\\]

\\[F_v = \begin{bmatrix} -ca^\top \cdot \mathrm{diag}(R^{-1}) \\\ -cg^\top \cdot \mathrm{diag}(R^{-1}) \end{bmatrix},\\]

\\[F _ { e } = \begin{bmatrix} { 2 P ^ { - 1 } \cdot A } & { \left[ 2 P ^ { - 1 } \cdot G , 0 , \cdots , 0 \right] } \\\ { 0 } & { 0 } & { } \end{bmatrix},\\]

\\[\overline { a } = \left[ \begin{array} { c } { I } \\\ { A } \\\ { A ^ { 2 } } \\\ { \vdots } \\\ { A ^ { N - 1 } } \end{array} \right]\\]

\\[a = \left[ \begin{array} { c } { \overline { a } } \\\ { A ^ { N } } \end{array} \right]\\]

\\(\hat{K}_i, \hat{k}_i\\) are the respective Lagrange multipliers and corresponding matrices.



# Background Research
## Characterizing Gene Expression with Green and Red Light for CcaS-CcaR
The CcaS-CcaR system is activated by green light, and deactivated by red light.
To determine whether or not we should keep green or red light constant and modulate the other, and at what intensity this colour of light should be kept constant, literature data was used to fit parameters for hill functions.  The hill functions for the affect of red and green light on gene expression were then plot against eachother three-dimensionally.  From this plot, an optimal range of gene expression can be picked and the value of intensity of either constant green or constant red light can be determined.  Therefore, from this analysis we know whether to modulate green or red light while keeping the other at constant value, as well as what constant value should be maintained, to provide an optimal gene expression range.
#### Green Light
Steady state protein expression from the CcaS-CcaR system was plotted with an increasing intensity of green input light.  The data was obtained from literature experiments and during these experiments, red light was held constant at an intensity of 1.05 W/m<sup>2</sup>.<sup>2</sup>
To capture the trend of the data and assign a single function across the entire range, curve fitting was performed.  A negative exponential curve fits the data well with the following attributes and values:
Exponential function:
y= a + b<sup>exp(c<sup>x</sup>)</sup>

- R<sup>2</sup>=0.96
- Mean Squared Error=3.8
- a=76.6
- b=-71.2
- c=-0.005
  ![Hill function for green light](http://2018.igem.org/File:T--Waterloo--GreenLight_HillFxn.png)
#### Red Light
The same analysis was done with red-light intensity.  Currently data is not available for CcaS-CcaR, therefore expression values for the Cph8-OmpR optogenetic system were used in its place.<sup>2</sup>
Exponential function:
y= a + b<sup>exp(c<sup>x</sup>)</sup>
- R<sup>2</sup>=0.96
- Mean Squared Error=5.2
- a=23.98
- b=94.1
- c=-0.007
  ![Hill function for red light](http://2018.igem.org/File:T--Waterloo--HillFxn_RedLight.png)
#### 3-D Plot
![Hill Function 3d plot](http://2018.igem.org/File:T--Waterloo--Hillxn_3d.png)
## Setting the Frequency and Duty Cycle for Light Modulation

To alter gene expression of the optogenetic system, the modulation of light can be used to provide different degrees of gene expression. In turn, the modulation of light can be affected by changing the either the frequency or period of modulation, or the duty cycle. However, changing the frequency or period, or changing the duty cycle will have effects on observed gene expression.<sup>1</sup> For the purposes of our system, it was decided to alter the duty cycle and keep the frequency constant.
Previous studies using the same optogenetic system expressing GFP and controlled using pulse-width modulation (PWM) demonstrated that frequency largely plays are role in modifying the amplitude of gene expression (difference between the maximal and minimal fluorescence values). Where a longer period indicated a larger amplitude.<sup>1</sup> This means that changing the period of light modulation affected variability of gene expression observed between light pulses. Since the goal of the project is a fine-tuned controller for growth rate, it would be more beneficial to use a shorter and fixed period to minimize variability of expression of the optogenetic system.
To determine what fixed period to use, certain boundaries must be kept in mind. First, the implementation of a PWM control strategy require that switching between 'on' and 'off' light signals occurs more rapidly than the system can respond.<sup>1</sup> This allows the system to average the input and return an even output, despite light modulation. Secondly, at low input period, the system cannot respond fast enough to track input signals for induction and repression, which is required for PWM control.<sup>1</sup> To remain within these boundaries, previous research indicates using a period in the range of 1-10 minutes.<sup>1</sup>
Thus, to vary the amount of light being delivered to the system to provide a continuous rate of gene expression, the duty cycle must be varied. The duty cycle, in our case, is defined as the percentage ratio of time the light is on to time the light is off.<sup>1</sup> For example, if the period is 100 seconds and the light is on for 75 seconds. The duty cycle would be 75%. Intuitively, a longer period indicates a greater amount of observed gene expression and vice versa.<sup>1</sup> Thus, by adjusting the duty cycle, different levels of gene expression can be observed.

#### Summary:
 * Decided to use a fixed period in the range of 1-10 minutes to minimize variability of expression of the optogenetic system controlled with pulse-width modulation.
 * Will adjust the duty cycle of PWM to affect levels of gene expression of the optogenetic system.

#### References:

**1.** Davidson, E. A., Basu, A. S., & Bayer, T. S. (2013). Programming Microbes Using Pulse Width Modulation of Optical Signals. *Journal of Molecular Biology*, 425(22), 4161–4166. doi:10.1016/j.jmb.2013.07.036

**2.** Olson, E. J., Hartsough, L. A., Landry, B. P., Shroff, R., & Tabor, J. J. (2014). Characterizing bacterial gene circuit dynamics with optically programmed gene expression signals. Nature Methods, 11(4), 449-455. doi:10.1038/nmeth.2884

**3.** Voelker, Anna, Konstantinos Kouramas, and Efstratios N. Pistikopoulos. "Moving Horizon Estimation: Error Dynamics and Bounding Error Sets for Robust Control." Automatica 49, no. 4 (2013): 943-48. Accessed October 16, 2018. doi:10.1016/j.automatica.2013.01.008.
