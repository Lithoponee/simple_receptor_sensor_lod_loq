# Noise Resolution LOD and LOQ Tutorial

This folder contains a self-contained teaching example showing how one receptor can behave as a sensor and how analytical performance emerges from the combination of receptor binding, signal gain, calibration slope, random noise and instrument digitization.

The main conclusion is:

> A sensor is not defined by gain, noise or ADC resolution alone. Designing for the lowest credible LOD means minimizing concentration-equivalent uncertainty, $\sigma_Y/|S|$, using the complete measurement procedure.

## Files

- [simple_receptor_sensor_lod_loq.ipynb](simple_receptor_sensor_lod_loq.ipynb) — editable Jupyter notebook
- [build_notebook.py](build_notebook.py) — regenerates the notebook
- [CONSOLIDATED_NOTE_REVIEW.md](CONSOLIDATED_NOTE_REVIEW.md) — scientific audit and merge record for the external consolidated note
- `figures/` — separate PNG and SVG teaching figures

## Quick start

1. Open `simple_receptor_sensor_lod_loq.ipynb` in Jupyter.
2. Edit the cell named **Inputs — edit only this cell**.
3. Run all cells from top to bottom.
4. Read the calculated results and inspect the individual figures.

The default demonstration uses a receptor with a dissociation constant of 100 nM, a baseline signal of 50 a.u. and a maximum signal change of 300% at complete occupancy.

## 1. From target concentration to receptor occupancy

For a simple 1:1 receptor under the free-target approximation:

$$
\theta=\frac{C}{K_D+C}
$$

where:

- $C$ is the free target concentration;
- $K_D$ is the dissociation constant;
- $\theta$ is the fraction of receptors occupied.

$K_D$ belongs to the receptor-target interaction under specified experimental conditions. At $C=K_D$, the expected occupancy is 50%. It is not an LOD and does not directly describe the instrument.

When $C$ is much smaller than $K_D$:

$$
\theta\approx\frac{C}{K_D}
$$

This is why the low-concentration portion of a binding curve can look approximately linear.

### Free concentration versus total concentration in a closed chamber

The simple isotherm uses the **free** target concentration $C_f$. In a well-mixed closed chamber with a finite number of receptors, some target molecules leave solution when they bind. Define the equivalent receptor concentration:

$$
R_T=\frac{N_R}{N_A V}
$$

where $N_R$ is the number of accessible receptors, $N_A$ is Avogadro's constant and $V$ is chamber volume. For ideal independent 1:1 binding:

$$
\theta=\frac{C_f}{K_D+C_f},\qquad
C_T=C_f+R_T\theta
$$

For a required occupancy $\theta_*$, inversion gives:

$$
\boxed{
C_T
=K_D\frac{\theta_*}{1-\theta_*}
+R_T\theta_*
}
$$

The first term supplies the free concentration that drives equilibrium; the second supplies molecules bound to receptors. If an external reservoir maintains $C_f$, the free concentration is the relevant control variable and depletion may be negligible. In a sealed small volume, the total concentration or total molecule inventory must include both terms.

As $K_D\rightarrow0$, this ideal closed-system requirement approaches $R_T\theta_*$. That is a receptor-inventory floor for this specific equilibrium model—not a universal analytical LOD. It changes with accessible receptor number and chamber volume and does not include signal noise, transport, nonspecific binding or decision probabilities. For inaccessible receptors use the accessible receptor count in $N_R$; for cooperative, multivalent or non-1:1 binding, rederive the mass balance rather than using this expression.

## 2. From occupancy to signal

The notebook deliberately ignores the physical origin of the signal and uses a simple teaching transducer:

$$
I=I_U(1+\theta G)
$$

where:

- $I_U$ is the unbound baseline signal;
- $G$ is the maximum fractional signal change at complete occupancy.

For example, $G=300\%=3$. The fully bound signal is therefore:

$$
I_B=I_U(1+3)=4I_U
$$

This does not mean that every concentration produces a 300% change. The actual percentage change is:

$$
\frac{\Delta I}{I_U}\times100\%=\theta G\times100\%
$$

With $K_D=100$ nM and $C=1$ nM, occupancy is approximately 0.99%. A maximum gain of 300% therefore produces only about a 2.97% signal change at 1 nM.

## 3. Sensitivity is the calibration slope

Inside a selected linear range, the calibration is represented as:

$$
I=SC+b
$$

where $S$ is the sensitivity or slope.

At concentrations much smaller than $K_D$, the ideal slope of the teaching receptor model is approximately:

$$
S\approx\frac{I_U G}{K_D}
$$

Higher gain generally gives a steeper slope when everything else is unchanged. However, gain alone does not determine LOD. The slope must be compared with noise.

## 4. Define the random variable before choosing a noise

The noise used for an LOD calculation must be the SD of the quantity that will actually be reported. For a paired normalized electrochemical response, define:

$$
Y=100\%\times\frac{I_{after}-I_{before}}{I_{before}}
$$

Then calculate the blank noise from repeated target-free experiments:

$$
\sigma_{Y,blank}=\operatorname{SD}(Y_1,Y_2,\ldots,Y_n)
$$

The recommended notebook inputs are:

```python
BLANK_NOISE_MODE = "paired_processed_response"
BLANK_BEFORE_AU = np.array([...])
BLANK_AFTER_AU = np.array([...])
```

The notebook calculates every processed blank response and its SD. This is preferable to borrowing a percentage because it passes through the same acquisition, extraction and normalization used for real samples.

An empirical full-chain blank SD can already contain:

- analogue electronic noise;
- ADC quantization;
- reference variation;
- baseline-extraction variability;
- before/after normalization;
- residual drift and surface instability.

Do not add these components again if they are already present in the measured blank responses. Separate Johnson, ADC, molecular Poisson and amplifier calculations are useful for diagnosis and design limits, but they are not additional LOD inputs unless they have not already been included and are propagated into the same output units.

### Choose the blank population that matches the product

- For continuous monitoring on one sensor, repeat blanks on the same electrode over the intended assay timescale.
- For a fresh disposable sensor per sample, use independently prepared blank chips across relevant days and fabrication batches.
- If both questions matter, report within-sensor and between-sensor SD separately.

The notebook records this choice descriptively in:

```python
BLANK_POPULATION_DESCRIPTION = "same sensor over the intended assay timescale"
```

### Separate within-electrode and between-electrode variation

Repeated readings on one electrode are not independent replacements for additional electrodes. A useful random-effects description is:

$$
Y_{ij}=\mu+b_i+\varepsilon_{ij}
$$

where $b_i$ is the between-electrode effect and $\varepsilon_{ij}$ is within-electrode variation. For a balanced design with $m$ repeated measurements on each electrode, the classical one-way random-effects estimates are:

$$
\hat\sigma_{within}^2=MS_W
$$

$$
\hat\sigma_{between}^2
=\frac{MS_B-MS_W}{m}
$$

and the variance of an individual measurement made on a randomly selected electrode is:

$$
\sigma_{total}^2
=\sigma_{between}^2+\sigma_{within}^2
$$

For a disposable sensor, total or between-electrode variability is essential because a new device is used for each sample. For continuous monitoring on one electrode, within-electrode variation over the relevant time window may be the more direct quantity. Pooling all repeated readings as if they were independent creates pseudoreplication and overstates the effective degrees of freedom.

The balanced-ANOVA formulas are teaching approximations. A negative estimate of $\sigma_{between}^2$ should not be interpreted as negative physical variance; use an appropriate variance-component method such as REML and report the uncertainty. More electrodes usually improve estimation of between-electrode variance, while repeats per electrode are still needed to estimate within-electrode variance. The best allocation depends on pilot variance ratios, costs, batches and the intended product—not on a universal “25 × 4” or “10 × 10” rule.

### Every nonzero calibration point may also have different noise

The blank, low-concentration samples and high-concentration samples do not need to have the same SD. Variance that changes with concentration is called heteroscedasticity.

```python
CALIBRATION_NOISE_SD_AU = np.array([
    np.nan, 0.10, 0.12, 0.14, ...
])
```

Each nonzero entry corresponds to the concentration in the same position in `CALIBRATION_CONCENTRATIONS_NM`. The first entry is ignored because blank noise is obtained from the processed blank experiment.

A clean blank can coexist with noisy samples. The notebook therefore reports both blank-based and sample-aware teaching estimates. When `USE_WEIGHTED_REGRESSION=True`, points with more uncertain means receive less influence in the fit. The weighting model should be justified for a real method.

## 5. ADC resolution and noise-limited resolution are different

### ADC or digital resolution

An ADC converts an analogue signal into discrete numerical steps. For an ideal $N$-bit ADC:

$$
q=\frac{\text{full-scale signal range}}{2^N}
$$

$q$ is one least-significant-bit signal step. In an actual potentiostat, the effective input step also depends on the selected current range, analogue gain and electronics.

The notebook uses this direct input:

```python
INSTRUMENT_SIGNAL_STEP_AU = 0.10
```

The corresponding concentration step is:

$$
\Delta C_{ADC}=\frac{q}{|S|}
$$

### Noise-limited resolution

Random fluctuations may be much larger than one ADC step. Near a locally linear calibration, a concentration difference produces an expected signal difference:

$$
\Delta Y\approx |S|\Delta C
$$

If two signal levels are declared distinguishable when their expected separation is at least $k_{res}$ times the uncertainty of their difference, then:

$$
\boxed{
\Delta C_{resolution}
=\frac{k_{res}\sigma_{\Delta Y}}{|S|}
}
$$

Here, $k_{res}$ is a selected separation rule—not a universal physical constant—and $\sigma_{\Delta Y}$ must match what is being compared.

If one reference is treated as known exactly and the other measurement has SD $\sigma$, then $\sigma_{\Delta Y}=\sigma$ and the simplified expression is:

$$
\Delta C_{resolution}=\frac{k_{res}\sigma}{|S|}
$$

If two independent individual measurements have SDs $\sigma_1$ and $\sigma_2$, the uncertainty of their difference is:

$$
\sigma_{\Delta Y}=\sqrt{\sigma_1^2+\sigma_2^2}
$$

Therefore:

$$
\boxed{
\Delta C_{resolution}
=\frac{k_{res}\sqrt{\sigma_1^2+\sigma_2^2}}{|S|}
}
$$

For two replicate means based on $n_1$ and $n_2$ measurements, use their standard errors:

$$
\sigma_{\Delta\bar Y}
=\sqrt{\frac{s_1^2}{n_1}+\frac{s_2^2}{n_2}}
$$

#### Numerical resolution example

Suppose the local sensitivity is $S=2.0$ nA/nM. Two individual concentration levels have signal SDs $\sigma_1=0.30$ nA and $\sigma_2=0.40$ nA, and the chosen separation rule is $k_{res}=2$.

First calculate the SD of the difference:

$$
\sigma_{\Delta Y}
=\sqrt{0.30^2+0.40^2}
=0.50\ \mathrm{nA}
$$

Then:

$$
\Delta C_{resolution}
=\frac{2\times0.50\ \mathrm{nA}}
{2.0\ \mathrm{nA/nM}}
=0.50\ \mathrm{nM}
$$

This means the selected two-SD rule requires approximately 0.50 nM separation under these assumptions. It is not an LOD and does not establish a particular false-positive or false-negative probability unless the statistical rule is defined accordingly.

The notebook's blank-versus-low-sample demonstration uses:

$$
\sigma_{separation}
=\sqrt{\sigma_{blank}^2+\sigma_{low}^2}
$$

and:

$$
\Delta C_{sample-aware}
=\frac{k_{res}\sigma_{separation}}{|S|}
$$

For the simplified teaching calculation:

$$
\Delta C_{practical}
\approx\max\left(\Delta C_{ADC},\Delta C_{sample-aware}\right)
$$

If a stable signal fluctuates across many digital steps, the measurement is mainly noise-limited. If the signal is stable but changes only in obvious numerical jumps, it may be ADC-limited.

## 6. Electronic noise sources: Johnson, current-shot, ADC and op-amp noise

These four terms occur at different stages of the electrochemical measurement chain:

$$
\text{electrode current}
\longrightarrow
\text{transimpedance amplifier}
\longrightarrow
\text{ADC}
\longrightarrow
\text{processed response}
$$

They are primarily fast, within-measurement terms. They are different from slow monolayer drift, fouling, reference drift, matrix effects and electrode-to-electrode variation.

### There is no universal direction

There is no general law that says higher signal always produces higher noise, that a larger electrode is always noisier, or that higher waveform frequency always gives higher total noise. The direction depends on the physical noise mechanism and on whether **absolute noise** or **relative noise** is being compared.

#### Higher signal can mean higher absolute noise but better precision

| Noise mechanism | Approximate absolute-noise behavior as signal current $I$ increases | Approximate relative noise $\sigma/I$ |
|---|---:|---:|
| Fixed electronic or thermal floor | approximately constant | improves as $1/I$ |
| Current-shot noise | $\sigma\propto\sqrt{I}$ | improves as $1/\sqrt{I}$ |
| Proportional or multiplicative noise | $\sigma=rI$ | remains approximately $r$ |
| Fixed ADC input step | approximately constant until the range changes | improves as $1/I$ |

For example, a signal of 1 with noise 0.1 has 10% relative noise. A signal of 100 with noise 1 has more absolute noise but only 1% relative noise. Therefore, the statement “the higher signal is noisier” is incomplete unless the units and the relative precision are also specified.

#### A bigger electrode can produce both more signal and more noise

As a first approximation, faradaic signal often scales with electrode area $A$:

$$
I_{signal}\propto A
$$

A fixed amplifier or ADC floor can then become smaller relative to the signal, approximately as $1/A$. Current-shot noise may grow as $\sqrt{A}$ while its relative contribution improves as $1/\sqrt{A}$. However, double-layer capacitance often grows approximately as $A$, so op-amp voltage noise coupled through $C_{dl}$ can also grow with area and may remain approximately constant relative to an area-scaled signal. Surface heterogeneity, fouling, charging current and drift can change as well. Thus a larger electrode can have greater absolute noise while still providing better—or sometimes unchanged—signal-to-noise performance.

#### Higher frequency does not automatically mean higher total noise

White-noise rms depends on effective measurement bandwidth $B$:

$$
\sigma_{white}\propto\sqrt{B}
$$

If waveform frequency increases while bandwidth remains fixed, Johnson noise does not automatically increase. If the acquisition bandwidth is widened at the same time, white noise increases approximately as $\sqrt{B}$. For a relatively narrow band of width $B$ centred near a characteristic frequency $f$, the capacitive conversion of white op-amp voltage noise can be approximated by:

$$
i_{n,cap}\approx2\pi fC_{dl}e_n\sqrt{B}
$$

This is a **spot-frequency approximation**, not an integral from DC to $B$. The distinction is made explicit in the worked budget below.

Current-shot noise changes when the faradaic current changes with frequency, whereas low-frequency $1/f$ noise density may decrease at higher frequency. The analytical signal can also increase, decrease or change sign with frequency. Frequency should therefore be selected using the measured slope-to-noise ratio:

$$
\text{frequency performance}=\frac{|S(f)|}{\sigma_Y(f)}
$$

The key principle is that **absolute noise may increase while relative precision, resolution or LOD improves**. Always compare signal or calibration slope with the corresponding noise obtained using the same waveform, bandwidth, electrode and processing procedure.

### Diagnose noise on two separate axes

The dependence of noise magnitude on signal and the dependence of noise on time or frequency are different questions.

**Amplitude scaling:** sweep a variable that genuinely changes mean current or response and inspect the slope of $\log\sigma$ versus $\log I$:

| Approximate slope | Compatible behavior |
|---:|---|
| 0 | fixed additive floor or a fixed ADC step |
| 0.5 | ideal counting statistics, such as current-shot noise |
| 1 | proportional noise, or another process whose absolute size follows signal |

This slope is diagnostic evidence, not unique identification. Electrode area, concentration and gain can change several mechanisms simultaneously, and a current-range transition can create an apparent discontinuity.

**Temporal spectrum:** use repeated measurements over time, an averaging-time sweep, a power spectral density or an Allan-deviation analysis to distinguish approximately white noise from correlated drift and $1/f$-like behavior. “Multiplicative” describes how noise amplitude scales with signal; “white,” “$1/f$” and “drift” describe temporal correlation. A multiplicative process can be white or correlated, so an averaging experiment alone does not separate “multiplicative noise” from “$1/f$ noise.”

### Effective noise bandwidth is not the sampling rate

For approximately white noise with amplitude spectral density $e_n$ and effective noise bandwidth $B$:

$$
\sigma_{rms}\approx e_n\sqrt{B}
$$

The effective bandwidth is set by the complete analogue and digital filter chain, sampling window and integration procedure. It is not automatically equal to waveform frequency or ADC sampling rate. For an ideal boxcar average of duration $T$, a common approximation is:

$$
B\approx\frac{1}{2T}
$$

Therefore, for independent white noise:

$$
\sigma_{white}(T)
=\sigma_{ref}\sqrt{\frac{T_{ref}}{T}}
$$

The two statements are consistent: $\sigma\propto\sqrt B$ and, because $B\propto1/T$, $\sigma\propto1/\sqrt T$. Thus $\sigma\sqrt T$ is constant in this ideal model. Averaging improves the precision of one result, but takes proportionally more time; it does not provide unlimited information per unit time.

### Why the narrowest bandwidth is not automatically best

The signal has a timescale or spectrum too. A filter that is too narrow can reduce peak height, broaden a transient, move an extracted peak position or miss a binding change. For coloured noise, the formal matched-filter result weights the signal spectrum by the inverse noise power spectrum; it is not simply the rule $B=B_{signal}$.

The notebook uses a deliberately simple first-order low-pass demonstration. For a one-pole low-pass filter, the equivalent noise bandwidth and cutoff frequency are related by:

$$
B_{ENBW}=\frac{\pi}{2}f_c,
\qquad f_c=\frac{2B_{ENBW}}{\pi}
$$

The demonstration maps $B_{ENBW}(T)=1/(2T)$ onto this equivalent one-pole cutoff. For a characteristic signal frequency $f_{signal}$, the retained signal fraction is:

$$
g(T)=\frac{1}{\sqrt{1+[f_{signal}/f_c(T)]^2}}
$$

This is a teaching transfer function, not a universal SWV model. An actual system should use its waveform, sampling windows, filter transfer function and signal-extraction method.

### Drift eventually defeats ordinary averaging

The notebook treats a linear drift rate $d$ as a midpoint-referenced bias over a window $T$:

$$
b_{drift}(T)=\frac{dT}{2}
$$

Random SD and deterministic bias are different quantities. For a compact teaching comparison they are combined as a root-mean-square error:

$$
\varepsilon_{noise}(T)
=\sqrt{\sigma_{white}(T)^2+b_{drift}(T)^2}
$$

and the plotted signal-aware error is:

$$
\varepsilon_{usable}(T)=\frac{\varepsilon_{noise}(T)}{g(T)}
$$

This explains the shape of Figure 9: white noise falls at short times, drift rises at long times, and excessive filtering penalizes the useful signal. It is not an Allan-deviation calculation and does not claim that drift is always linear.

With the editable defaults—1.0% white-noise SD at 0.10 s, 0.10% per minute linear drift and $f_{signal}=0.10$ Hz—the noise-only optimum is about 41.6 s. After the illustrative signal-attenuation penalty, the useful optimum is about 3.2 s, corresponding to about 0.16 Hz equivalent noise bandwidth, a 0.10 Hz one-pole cutoff, 71% retained signal and 0.25% baseline-referenced error. These are demonstration values, not measured sensor specifications.

Edit these notebook inputs to explore a real protocol:

```python
AVERAGING_REFERENCE_TIME_S = 0.10
WHITE_SD_AT_REFERENCE_TIME_PERCENT = 1.0
LINEAR_DRIFT_RATE_PERCENT_PER_MIN = 0.10
SIGNAL_CHARACTERISTIC_FREQUENCY_HZ = 0.10
AVERAGING_TIME_MIN_S = 0.01
AVERAGING_TIME_MAX_S = 300.0
```

For real data, sweep the averaging or integration time and record both the processed blank SD $\sigma_Y(T)$ and calibration slope $S(T)$. Compare $\sigma_Y(T)/|S(T)|$ while also checking signal loss, peak shape, peak-position bias and total assay time. Correlated low-frequency noise generally does not decrease as $1/\sqrt N$; its behavior depends on its spectrum, observation duration and referencing or detrending procedure.

### Johnson thermal noise

Thermal motion produces voltage or current fluctuations in every resistance. For a feedback resistor $R_f$, the input-referred current-noise density is:

$$
i_{n,J}=\sqrt{\frac{4k_BT}{R_f}}
\quad[\mathrm{A}/\sqrt{\mathrm{Hz}}]
$$

For approximately white noise over effective bandwidth $B$:

$$
i_{J,rms}\approx\sqrt{\frac{4k_BT B}{R_f}}
$$

The electrolyte and electrode impedance also generate thermal noise. For a general impedance, the real part of its admittance determines the dissipative thermal-current contribution.

Johnson noise is affected by:

- **Temperature:** absolute noise grows approximately as $\sqrt{T}$.
- **Effective bandwidth:** rms noise grows as $\sqrt{B}$.
- **Feedback resistance:** input-referred resistor-current noise decreases as $1/\sqrt{R_f}$, although gain, stability and measurable-current range change at the same time.
- **Solution and electrode impedance:** only the dissipative real component generates Johnson noise.
- **Electrode area:** area changes signal and electrode impedance. A fixed additive Johnson-current floor becomes smaller relative to a signal that grows with area, but solution resistance and capacitance also change, so area is not a universal one-direction noise lever.

### Electronic current-shot noise

Faradaic charge is transferred in discrete electron events. Under the ideal independent-event approximation:

$$
i_{shot,rms}=\sqrt{2qIB}
$$

where $q$ is the elementary charge, $I$ is average faradaic current and $B$ is effective bandwidth.

Absolute shot noise increases as $\sqrt{I}$, but relative shot noise decreases:

$$
\frac{i_{shot,rms}}{I}=\sqrt{\frac{2qB}{I}}
$$

Current-shot noise is affected by:

- **Faradaic current:** larger current produces more absolute noise but better relative precision.
- **Bandwidth:** absolute and relative rms noise grow as $\sqrt{B}$.
- **Electrode area and active reporter population:** if current scales with area or reporter number, relative shot noise often improves approximately as $1/\sqrt{A}$.
- **Frequency and amplitude:** these change the faradaic current and therefore indirectly change shot noise. The relationship depends on electron-transfer kinetics and waveform response.

This electronic charge-shot term is not the same as molecular Poisson noise from a finite number of occupied receptors.

### Molecular binding or receptor-counting noise

Electronic shot noise counts electron-transfer events. Molecular binding noise counts how many receptors occupy the bound state. For $N_R$ independent receptors with occupancy probability $\theta$:

$$
N_B\sim\operatorname{Binomial}(N_R,\theta)
$$

$$
E[N_B]=N_R\theta,\qquad
\operatorname{Var}(N_B)=N_R\theta(1-\theta)
$$

For the measured occupancy fraction $\hat\theta=N_B/N_R$:

$$
\operatorname{Var}(\hat\theta)
=\frac{\theta(1-\theta)}{N_R},\qquad
\sigma_{\hat\theta}
=\sqrt{\frac{\theta(1-\theta)}{N_R}}
$$

Thus the bound-count SD grows as $\sqrt{N_R}$ while the occupancy-fraction SD improves as $1/\sqrt{N_R}$. If accessible receptor number scales with area, $N_R\propto A$, the ideal occupancy SD scales as $1/\sqrt A$.

At $\theta=0.5$ and $N_R=10^{10}$, $\sigma_{\hat\theta}=5\times10^{-6}$, or 0.0005 percentage points. At the same occupancy with only $N_R=100$, $\sigma_{\hat\theta}=0.05$, or 5 percentage points. Molecular counting noise is therefore usually negligible for a large independent receptor population but can matter for nanoelectrodes, small populations, single-molecule measurements or short observation times.

This model assumes independent identical receptors at equilibrium. Correlated binding, heterogeneous affinity, transport limitation, finite binding kinetics and temporal averaging change the result. Occupancy noise must also be propagated through the occupancy-to-signal model before it can be compared with current or voltage noise. Do not add it again to a processed blank SD if the empirical blank already contains the fluctuation.

### ADC quantization

The ADC converts the analogue front-end output into discrete codes. For an ideal $N$-bit converter:

$$
\mathrm{LSB}=\frac{\text{full-scale span}}{2^N}
$$

When quantization error samples many codes and behaves approximately as a uniform random variable:

$$
\sigma_q\approx\frac{\mathrm{LSB}}{\sqrt{12}}
$$

When the signal occupies only a few codes, the direct step is more informative than the random-noise approximation:

$$
\Delta C_{ADC}=\frac{\mathrm{LSB}}{|S|}
$$

#### Numerical example: a 16-bit ADC and a range of 40

A 16-bit ADC has:

$$
2^{16}=65\,536\ \text{codes}
$$

The meaning of “40” must be checked carefully. If 40 µA is the **total span**, for example 0 to 40 µA or $-20$ to $+20$ µA, then:

$$
\mathrm{LSB}
=\frac{40\ \mu\mathrm{A}}{65\,536}
=0.000610\ \mu\mathrm{A}
=0.610\ \mathrm{nA}
$$

The ideal uniform-quantization SD would be:

$$
\sigma_q
=\frac{0.610\ \mathrm{nA}}{\sqrt{12}}
=0.176\ \mathrm{nA}
$$

If the stated range is instead $\pm40$ µA, its total span is 80 µA:

$$
\mathrm{LSB}
=\frac{80\ \mu\mathrm{A}}{65\,536}
=1.221\ \mathrm{nA}
$$

and:

$$
\sigma_q=\frac{1.221}{\sqrt{12}}=0.352\ \mathrm{nA}
$$

For an illustrative current sensitivity of $S=2.0$ nA/nM, the direct ADC-limited concentration steps would be:

$$
\Delta C_{ADC}
=\frac{0.610\ \mathrm{nA}}{2.0\ \mathrm{nA/nM}}
=0.305\ \mathrm{nM}
$$

for a total 40 µA span, or:

$$
\Delta C_{ADC}
=\frac{1.221\ \mathrm{nA}}{2.0\ \mathrm{nA/nM}}
=0.611\ \mathrm{nM}
$$

for a $\pm40$ µA range. If “40” is expressed in nA, mA or volts instead, the calculation is identical but retains that unit. Nominal bit depth is not the same as effective resolution: analogue noise, effective number of bits, selected current range and front-end gain can all make the practical input-referred step larger.

ADC performance is affected by:

- **Bit depth or effective number of bits:** more effective bits give a smaller LSB.
- **Selected current range:** a wider range gives a larger input-referred LSB. Correct ranging is often more important than nominal ADC bit depth.
- **Signal magnitude:** the absolute LSB is unchanged, but its percentage of the measured peak decreases when the signal is larger.
- **Electrode area, waveform frequency and amplitude:** these may increase peak current and improve relative quantization, provided the input does not saturate or leave the linear range.
- **Averaging and oversampling:** these can improve effective resolution when quantization error and analogue noise provide sufficient code-to-code variation. They do not change the physical LSB.
- **ADC sampling rate and digital filtering:** these determine how many samples contribute and the effective bandwidth.
- **Autoranging:** a range change during an experiment can change both quantization and analogue gain. Record the range used for every trace.

### Op-amp and analogue-front-end noise

The potentiostat transimpedance amplifier converts electrode current into voltage. The amplifier contributes input-voltage noise, input-current noise, feedback-component noise and low-frequency $1/f$ noise.

A simplified voltage-noise contribution through the double-layer capacitance is:

$$
i_n(f)\approx2\pi fC_{dl}e_n
$$

where $e_n$ is op-amp input-voltage-noise density. This shows why a large capacitive electrode can amplify voltage noise at high frequency.

For flat $e_n$ and purely capacitive coupling integrated over a band from $f_1$ to $f_2$, the corresponding RMS current is:

$$
i_{n,cap,rms}
=2\pi C_{dl}e_n
\sqrt{\frac{f_2^3-f_1^3}{3}}
$$

For a zero-to-$f_{BW}$ band this becomes $2\pi C_{dl}e_nf_{BW}^{3/2}/\sqrt3$. This broadband integral and the spot-frequency approximation answer different questions and must not be interchanged.

Op-amp and front-end noise are affected by:

- **Op-amp voltage- and current-noise density:** lower-noise components reduce the corresponding terms.
- **Double-layer capacitance:** $C_{dl}$ generally increases with geometric area, electrochemically active area and roughness.
- **Waveform frequency:** the capacitive conversion of voltage noise increases with frequency. Electron-transfer signal may also change with frequency, so the final SNR must be measured rather than inferred from noise alone.
- **Measurement bandwidth and integration window:** wider bandwidth passes more noise; longer integration and suitable filtering reduce white noise.
- **Feedback resistor and capacitor:** they set gain, bandwidth, stability and resistor noise.
- **Input capacitance, cabling and layout:** long cables and parasitic capacitance can increase noise gain and pickup.
- **Shielding and grounding:** poor layout allows mains and electromagnetic interference to enter the front end.
- **Potential amplitude:** amplitude does not directly change intrinsic op-amp noise density, but it can increase faradaic and charging currents, change the baseline and move the electronics toward saturation or nonlinearity.

### How electrode area affects the four terms

Assume, only as a first approximation, that faradaic signal scales as $I_{signal}\propto A$ and double-layer capacitance scales as $C_{dl}\propto A$.

| Contribution | Approximate absolute scaling with area | Approximate relative effect |
|---|---:|---:|
| Fixed ADC input step | $A^0$ | improves roughly as $1/A$ until range changes |
| Fixed amplifier current noise | $A^0$ | improves roughly as $1/A$ |
| Current-shot noise | $\sqrt{A}$ | improves roughly as $1/\sqrt{A}$ |
| Voltage noise through $C_{dl}$ | $A$ | may remain approximately constant relative to an $A$-scaled signal |
| Molecular counting noise | approximately $\sqrt{A}$ in count | relative occupancy-counting noise improves roughly as $1/\sqrt{A}$ |

These are scaling guides, not universal laws. Larger area also changes solution resistance, mass transport, surface heterogeneity, charging current, current range and fabrication variability. A larger electrode can improve some additive-noise terms while leaving capacitance-related noise unchanged in relative units.

### How SWV frequency affects the terms

SWV frequency and electronic noise bandwidth are not automatically the same quantity. Frequency sets the waveform timing; effective bandwidth is determined by sampling windows, filters and integration.

- With fixed bandwidth, ideal Johnson and ADC step sizes do not inherently grow with SWV frequency.
- Current-shot noise changes when the faradaic current changes with frequency.
- Op-amp voltage noise coupled through $C_{dl}$ generally becomes more important at higher frequency.
- E-AB signal gain can rise, fall or change sign with frequency because reporter electron-transfer kinetics are frequency-dependent.
- If the acquisition bandwidth is deliberately increased in proportion to frequency, white rms noise also grows approximately as $\sqrt{B}$.

Therefore, measure both gain and processed blank SD at every waveform frequency:

$$
\mathrm{SNR}(f)=\frac{|S(f)|}{\sigma_Y(f)}
$$

Do not select frequency from the largest signal gain alone.

### How potential amplitude affects the terms

Increasing SWV amplitude can increase peak current or signal contrast, which may improve relative ADC and fixed additive-noise performance. However, excessive amplitude can also:

- increase capacitive and background current;
- distort peak shape;
- activate unwanted electrochemical reactions;
- broaden the effective potential region;
- drive the front end into a wider current range or saturation;
- change the apparent E-AB gain nonlinearly.

Amplitude should therefore be optimized using the measured ratio of calibration slope to processed-response noise, not peak height alone.

### Comparing magnitudes

Convert every component into the same units before comparing it. For raw current:

$$
\text{relative current noise (\%)}
=100\%\times\frac{i_{rms}}{|I_{peak}|}
$$

For analytical calculations, propagate raw current noise through SWV differencing, peak extraction and normalization to obtain $\sigma_Y$. If independent component estimates are genuinely in the same output units, they may be combined as:

$$
\sigma_{independent,total}
=\sqrt{\sigma_1^2+\sigma_2^2+\cdots}
$$

Do not use this root-sum-square expression for correlated terms without including covariance. Most importantly, do not add calculated component terms to an empirical processed blank SD that already contains them. A fitted response-domain $\sigma_0$ can also contain baseline-proportional drift, extraction error and between-sensor variation, so disagreement with the electronic bars is not by itself evidence that the electronics calculation is wrong.

### Worked electronic-noise budget with numbers

It is possible to calculate illustrative magnitudes, but there is no universal value for any electronic-noise term. Every result belongs to a specified circuit, current, bandwidth and processing procedure. The following example uses the same editable defaults as Figure 7:

| Parameter | Illustrative value |
|---|---:|
| Temperature $T$ | 298.15 K |
| Effective noise bandwidth $B$ | 10 Hz |
| Feedback resistance $R_f$ | 1.0 MΩ |
| Mean faradaic current $I$ | 1.0 µA |
| Op-amp current-noise density $i_n$ | 2.0 fA/$\sqrt{\mathrm{Hz}}$ |
| Op-amp voltage-noise density $e_n$ | 8.0 nV/$\sqrt{\mathrm{Hz}}$ |
| Double-layer capacitance $C_{dl}$ | 10 nF |
| Characteristic frequency $f$ | 100 Hz |
| Separate upper edge for broadband capacitive integral $f_{BW}$ | 10 Hz |
| ADC total current span | 2.0 µA |
| ADC effective bit depth | 18 bit |

#### Johnson thermal contribution

For the feedback resistor:

$$
i_{J,rms}
=\sqrt{\frac{4k_BTB}{R_f}}
=0.406\ \mathrm{pA}
$$

#### Current-shot contribution

For a 1.0 µA mean faradaic current:

$$
i_{shot,rms}
=\sqrt{2qIB}
=1.790\ \mathrm{pA}
$$

#### Op-amp front-end contribution

The illustrative input-current-noise term is:

$$
i_{op,current}
=i_n\sqrt B
=0.0063\ \mathrm{pA}
$$

The plotted Figure 7 voltage-noise term is the **narrow-band spot approximation** for a 10 Hz band centred near 100 Hz:

$$
i_{op,voltage}
=2\pi fC_{dl}e_n\sqrt B
=0.159\ \mathrm{pA}
$$

It is not the integral from 0 to 10 Hz. Under the alternative assumptions of flat $e_n$, purely capacitive coupling and integration from DC to an upper frequency of 10 Hz:

$$
i_{op,voltage,0\rightarrow f_{BW}}
=2\pi C_{dl}e_n\sqrt{\frac{f_{BW}^3}{3}}
=0.0092\ \mathrm{pA}
$$

The values differ because they represent different frequency bands. The spot calculation samples the much larger capacitive conversion near 100 Hz; the broadband calculation integrates only from 0 to 10 Hz. The upper integration edge $f_{BW}$ is entered separately because a filter's equivalent noise bandwidth and its upper cutoff are not generally identical.

Combining those two independent example terms gives:

$$
i_{op,total}
=\sqrt{i_{op,current}^2+i_{op,voltage}^2}
\approx0.159\ \mathrm{pA}
$$

The notebook reports both capacitive values but uses the explicitly labelled 100 Hz spot model in the plotted op-amp bar. Neither simplified value replaces a complete potentiostat noise-gain calculation, which may also require the feedback capacitor, source impedance, input capacitance, frequency-dependent op-amp spectra and the full analogue transfer function.

#### ADC quantization contribution

For a 2.0 µA total span and 18 effective bits:

$$
\mathrm{LSB}
=\frac{2.0\ \mu\mathrm{A}}{2^{18}}
=7.629\ \mathrm{pA}
$$

If the quantization error behaves approximately as a sampled uniform variable:

$$
\sigma_q
=\frac{\mathrm{LSB}}{\sqrt{12}}
=2.202\ \mathrm{pA}
$$

If the signal occupies only a few codes or is highly stable between codes, report the 7.629 pA LSB directly rather than assuming the 2.202 pA random-noise model.

#### Comparison in common units

| Contribution | Input-referred RMS current | Relative to 1.0 µA |
|---|---:|---:|
| Johnson thermal | 0.406 pA | 0.406 ppm = 0.0000406% |
| Current shot | 1.790 pA | 1.790 ppm = 0.000179% |
| Op-amp front end, plotted 100 Hz spot model | 0.159 pA | 0.159 ppm = 0.0000159% |
| Capacitive term integrated from 0 to 10 Hz, diagnostic alternative | 0.0092 pA | 0.0092 ppm = 0.00000092% |
| ADC quantization | 2.202 pA | 2.202 ppm = 0.000220% |
| Independent RSS total | **2.871 pA** | **2.871 ppm = 0.000287%** |

These numbers are not PalmSens specifications and should not be presented as measured sensor noise. They demonstrate the calculation and scale for one hypothetical configuration. To compare them with a normalized response such as $Y=100(I_{after}-I_{before})/I_{before}$, propagate every component through the same differencing, peak extraction and normalization used for the experimental result.

The parameter scaling also shows why quoted electronic-noise values cannot be universal:

| Parameter change | Approximate effect under this simplified model |
|---|---|
| Bandwidth multiplied by 100 | white RMS terms multiplied by 10 |
| Faradaic current multiplied by 100 | absolute shot-noise SD multiplied by 10; relative shot noise divided by 10 |
| $C_{dl}$ multiplied by 10 | voltage-noise-through-capacitance term multiplied by 10 |
| Characteristic frequency multiplied by 10 | same simplified capacitive op-amp term multiplied by 10 |
| ADC span multiplied by 10 | LSB and ideal quantization SD multiplied by 10 |
| Effective ADC resolution increased by 1 bit | LSB and ideal quantization SD divided by 2 |

### What these four bars do not include

The four electronic bars do not by themselves describe:

- baseline-model bias;
- slow reference drift;
- monolayer reorganization or reporter loss;
- biofouling and nonspecific adsorption;
- matrix-dependent effects;
- electrode-to-electrode and batch-to-batch variation;
- temperature-driven changes in chemistry;
- concentration-dependent sample variance.

These effects may dominate the final processed-response SD even when the electronic bars are individually small.

### Editable electronic-noise example in the notebook

Figure 7 compares all four terms after converting them to the same input-referred rms-current unit. Change these teaching inputs in Section 0 of the notebook:

```python
TEMPERATURE_K
EFFECTIVE_NOISE_BANDWIDTH_HZ
FEEDBACK_RESISTANCE_OHM
MEAN_FARADAIC_CURRENT_A

OPAMP_CURRENT_NOISE_DENSITY_A_PER_SQRT_HZ
OPAMP_VOLTAGE_NOISE_DENSITY_V_PER_SQRT_HZ
DOUBLE_LAYER_CAPACITANCE_F
CHARACTERISTIC_FREQUENCY_HZ
CAPACITIVE_INTEGRATION_UPPER_HZ

ADC_FULL_SCALE_CURRENT_SPAN_A
ADC_EFFECTIVE_BITS
```

The plotted op-amp value is the root-sum-square of the input-current-noise contribution and the explicitly labelled narrow-band spot voltage-noise contribution. The notebook diagnostic table separately reports the zero-to-$f_{BW}$ integral; it is not included in the bar or added to the spot value. The dashed line is the independent root-sum-square total of the four displayed bars. These values are illustrative and are not specifications for PalmSens or any other particular instrument.

## 7. How to change resolution in the notebook

### Change the simulated ADC step

```python
INSTRUMENT_SIGNAL_STEP_AU = 0.10
```

A smaller value represents finer digital resolution. This improves the ADC-limited concentration step but helps little if experimental noise is already much larger.

### Change the real or simulated noise

Edit:

```python
BLANK_BEFORE_AU
BLANK_AFTER_AU
CALIBRATION_NOISE_SD_AU
```

The paired arrays determine the processed blank-response SD. The calibration array controls simulated noise at each nonzero level. Lower experimental noise improves noise-limited resolution, LOD and LOQ. In a real experiment, use measured replicates rather than choosing SD values merely to obtain a preferred result.

### Change the separation rule

```python
RESOLUTION_NOISE_FACTOR = 2.0
```

This changes how many standard deviations are required before two levels are called distinguishable. Lowering the factor makes the reported numerical resolution smaller, but it does not physically improve the sensor.

### Change the slope only when the sensor truly changes

The slope can increase through larger physical signal gain, lower $K_D$ in the relevant concentration region, improved surface chemistry or another genuine transduction improvement. Editing the simulated gain changes the model; it does not by itself improve a real device.

## 8. LOD and LOQ

A conventional blank-threshold estimate is:

$$
C_{3\sigma}=\frac{3\sigma_{blank}}{|S|}
$$

and the analogous ten-sigma concentration is:

$$
C_{10\sigma}=\frac{10\sigma_{blank}}{|S|}
$$

These are useful teaching conventions, but they are not a complete detection-probability calculation and ten sigma is not automatically a validated LOQ.

### Fit additive and proportional noise separately

The notebook fits the replicate SDs in the signal domain with:

$$
\sigma_Y(C)=\sqrt{\sigma_0^2+\left[r\mu_{net}(C)\right]^2}
$$

Here, $\sigma_0$ is the component that is constant **in the processed net-response variable**, $r$ is the proportional CV and $\mu_{net}(C)$ is the mean signal above the blank. “Additive in net response” does not mean “electronic in the physical chain.” Noise proportional to the large baseline $I_U$, residual drift, normalization or extraction error and between-sensor variability can all become nearly constant after subtraction or normalization and therefore be absorbed into fitted $\sigma_0$.

Consequently, do not compare fitted $\sigma_0$ directly with the Figure 7 electronic bars. First propagate the electronic terms through the same waveform, bandwidth, peak extraction, differencing and normalization, and ensure that both quantities describe the same timescale and replicate population. Do not fit one straight line to $\log\sigma$ versus $\log C$: this two-component model has a log-log slope near zero at low response and near one only when proportional noise dominates.

For a calibration fitted to level means, the statistical objective weight is:

$$
w_i=\frac{n_i}{s_i^2}
$$

The notebook passes its square root, $\sqrt{n_i}/s_i=1/\mathrm{SEM}_i$, to `numpy.polyfit`, because that function multiplies unsquared residuals by the supplied weight.

### Known-parameter detection approximation

For false-positive probability $\alpha$ and false-negative probability $\beta$, a useful teaching approximation solves:

$$
|S|C_D=z_{1-\alpha}\sigma_0+z_{1-\beta}\sigma_Y(C_D)
$$

The defaults $\alpha=0.05$ and $\beta=0.05$ give a 95/95 detection criterion. The first term defines the critical blank decision threshold; the second requires the analyte distribution to clear that threshold with the requested detection probability. If proportional noise is too large, a finite solution may not exist under the model.

This equation treats the blank mean, variances and calibration slope as known. It is therefore a linear-response, approximately normal-distribution teaching approximation—not a complete finite-calibration LOD. IUPAC defines the [critical value](https://goldbook.iupac.org/terms/view/08017) through a chosen false-positive probability and defines a [detection limit](https://goldbook.iupac.org/terms/view/D01629) using a stated probability of distinguishing a result from a suitable blank. The exact calculation for a real method should include the applicable calibration, variance estimation and validation protocol.

### An SNR crossing is useful but is not a formal LOD

The equation:

$$
|S|C=k\sigma_Y(C)
$$

finds the concentration where the mean net response equals $k$ times the noise at that same concentration. It is a valid signal-to-noise criterion, but it does not define a false-positive probability relative to the blank and therefore should not automatically be labelled an LOD.

For the additive-plus-proportional model in a linear range:

$$
\sigma_Y(C)=\sqrt{\sigma_0^2+(r|S|C)^2}
$$

the positive SNR-crossing solution is:

$$
C
=\frac{k\sigma_0/|S|}{\sqrt{1-(kr)^2}}
$$

and exists only when $kr<1$. This existence condition belongs to the selected SNR rule; it is different from the known-parameter detection-probability approximation above.

### Precision-only LOQ

The notebook also finds the lowest concentration satisfying:

$$
\frac{\sigma_Y(C)}{|S|C}\leq 10\%
$$

This is a precision-only LOQ candidate. A real LOQ must additionally pass predefined accuracy or recovery criteria using low-level samples.

### Blank uncertainty matters

The 95% confidence interval for the true blank SD is calculated from the chi-square distribution. With only three blanks, the interval is approximately $0.52s$ to $6.3s$, so a three-blank estimate is very uncertain. A small measured blank SD produces an optimistically low threshold; it is not an upper bound on the LOD.

There is a second finite-blank issue: the blank mean is also estimated. For one future measurement compared with the mean of $n$ blanks, a simple one-sided prediction threshold is:

$$
Y_{critical}
=\bar Y_{blank}
+t_{1-\alpha,n-1}s_{blank}
\sqrt{1+\frac{1}{n}}
$$

For $n=3$ and $\alpha=0.05$:

$$
t_{0.95,2}\sqrt{1+\frac13}
=2.920\times1.155
\approx3.37
$$

whereas the known-parameter normal factor is $z_{0.95}=1.645$. Thus three blanks can make the simple normal-threshold calculation optimistic through both an uncertain SD and an uncertain blank mean. The notebook reports this finite-blank prediction factor for the entered number of blanks.

This $t$ correction still treats the calibration slope and intercept as fixed. When their uncertainty matters, use an appropriate calibration prediction-interval approach, such as a correctly implemented Hubaux–Vos procedure, and follow the validation framework applicable to the method.

The notebook additionally displays a conservative sample-aware comparison:

$$
\mathrm{LOD}_{sample-aware}
=\frac{k_{LOD}\sqrt{\sigma_{blank}^2+\sigma_{low}^2}}{|S|}
$$

This comparison demonstrates why clean blanks do not guarantee reliable detection when low-level samples are imprecise. It is deliberately labelled a **heuristic** and is not a replacement for a finite-calibration detection method or experimental validation.

LOD concerns deciding whether analyte is distinguishable from the blank. LOQ concerns the higher concentration region where quantitative precision and accuracy are acceptable. Proposed values should be verified with real low-level samples.

### Reusable methods wording

> Replicate response variance was modeled as $\sigma_Y(C)=\sqrt{\sigma_0^2+[r\mu_{net}(C)]^2}$, where $\sigma_0$ is constant in the processed net-response variable and is not assumed to be a purely electronic floor. Calibration level means were fitted using objective weights $n_i/s_i^2$. The known-parameter teaching detection concentration was obtained from $|S|C_D=z_{1-\alpha}\sigma_0+z_{1-\beta}\sigma_Y(C_D)$ with $\alpha=\beta=0.05$; a finite-blank critical level was also calculated using $t_{1-\alpha,n-1}s_{blank}\sqrt{1+1/n}$. Calibration slope and intercept uncertainty require an appropriate prediction-interval method. The precision-only LOQ candidate was the lowest concentration with predicted relative concentration SD no greater than 10%; accuracy and recovery require separate experimental validation.

## 9. How the linear range is selected

The receptor response is hyperbolic rather than perfectly linear:

$$
I=I_U\left(1+G\frac{C}{K_D+C}\right)
$$

The linear range is therefore the concentration interval where a linear calibration is an acceptable approximation for the intended method.

In the input cell, change:

```python
LINEAR_RANGE_MAX_NM = 10.0
```

Then rerun the notebook and check:

1. whether the calibration points cover the proposed range;
2. whether residuals appear randomly distributed around zero;
3. whether there is systematic curvature;
4. whether replicate precision is acceptable at every level;
5. whether the slope remains suitable;
6. whether back-calculated concentrations meet the method's accuracy criteria.

$R^2$ is useful but does not define the range by itself. A high $R^2$ can still occur for a visibly curved response. The notebook therefore reports both $R^2$ and the maximum residual relative to the fitted response span.

The editable teaching checks are:

```python
MIN_R_SQUARED = 0.995
MAX_RESIDUAL_PERCENT_OF_RESPONSE_SPAN = 5.0
```

These are teaching choices, not universal analytical standards.

## 10. Central sensor-design question: how do I achieve the lowest LOD?

The design target is not maximum signal or minimum electronic noise separately. It is minimum **concentration-equivalent uncertainty**:

$$
\boxed{
\sigma_C(C)=\frac{\sigma_Y(C)}{|S(C)|}
}
$$

For a conventional low-level rule:

$$
\mathrm{LOD}\approx\frac{k\sigma_0}{|S|}
$$

Therefore, every design change must be evaluated using both the processed-response noise and the calibration slope measured under the same conditions.

| Design change | If everything else truly stays fixed | Important failure mode |
|---|---|---|
| Double slope $S$ | LOD is halved | noise may rise with gain or current |
| Halve processed-response noise $\sigma_0$ | LOD is halved | the apparent reduction may come from undersampling drift |
| Double both slope and noise | no LOD improvement | larger signal can look better while concentration performance is unchanged |
| Lower $K_D$ | increases the ideal low-concentration slope | saturation, slow release, transport limitation or loss of quantitation range |
| Increase electrode area | may increase signal and receptor count | capacitance, background, fouling and between-electrode variability may also rise |
| Average longer | reduces independent white noise | signal attenuation, drift and assay time eventually dominate |
| Add preconcentration | raises local target concentration | recovery variability and blank contamination may rise |

For the ideal receptor model at $C\ll K_D$:

$$
S\approx\frac{I_UG}{K_D}
$$

and therefore:

$$
\boxed{
\mathrm{LOD}\approx
\frac{k\sigma_0K_D}{I_UG}
}
$$

This is a design map, not a universal physical law. For a normalized response the baseline factor may cancel, and for a real sensor $G$, $K_D$, accessible receptor density, kinetics and noise may change together.

### Practical optimization sequence

1. Specify the matrix, required concentration range and acceptable false-positive and false-negative probabilities.
2. Choose affinity and kinetics that give appreciable response without complete saturation across the intended range.
3. For each electrode area, receptor density, waveform, amplitude, frequency, incubation time and averaging setting, measure both $S$ and $\sigma_Y$.
4. Minimize $\sigma_Y/|S|$ using matrix-matched blanks and low-level samples. Include between-sensor variability if each sample uses a new disposable sensor.
5. Compare the empirical processed-response floor with a consistently propagated electronic budget. Improve surface reproducibility, antifouling, transport and referencing first when they dominate.
6. Confirm the predicted LOD using independent low-concentration samples and the appropriate calibration prediction-interval or validation procedure.

The central design sentence is:

> The lowest credible LOD comes from the largest reliable low-concentration slope divided by the smallest full-procedure response uncertainty—not from the largest current, lowest $K_D$, finest ADC or longest averaging time alone.

## 11. Interpreting the default results

With the default settings, the notebook demonstrates that:

- the ADC-limited concentration step is smaller than the noise-limited resolution;
- the system is therefore mainly noise-limited;
- the additive floor and proportional noise term can be estimated from all replicate SDs;
- a conventional blank threshold, a known-parameter detection approximation, a finite-blank critical level and a precision-only LOQ are different quantities;
- a 10% maximum gain gives a much shallower low-concentration slope than a 300% maximum gain;
- different calibration points have visibly different SD values;
- the per-level noise figure separates the smooth simulated SD from the finite-replicate SD and shows uncertainty in every SD estimate;
- the electronic-noise budget separately shows Johnson, current-shot, op-amp and ADC contributions in the same rms-current units;
- the selected linear range should be judged from residuals as well as $R^2$;
- the bandwidth demonstration separates white-noise reduction, drift bias and loss of useful signal, producing an illustrative optimum rather than claiming that narrower is always better.

Exact values can change slightly because the calibration replicates are simulated with a fixed random seed.

## 12. Common mistakes

### Mistake 1: Gain equals LOD

Gain affects the response magnitude. LOD depends on slope relative to noise.

### Mistake 2: ADC resolution equals analytical resolution

The ADC step is only one limitation. Noise, slope and data processing determine whether the corresponding concentration difference is actually distinguishable.

### Mistake 3: A clean blank guarantees a good LOD

Low-level samples may be much noisier than the blank. Check their replicate precision.

### Mistake 4: Lowering a sigma factor improves the sensor

Changing the factor changes the declaration rule, not the physical measurement.

### Mistake 5: High R squared proves linearity

Always inspect residuals, curvature, precision and back-calculated accuracy.

### Mistake 6: Kd is the detection limit

$K_D$ describes receptor affinity. LOD describes the performance of the complete analytical procedure.

### Mistake 7: Add every named noise source to the measured blank SD

The processed blank SD already contains all components that survive the full measurement chain. Adding separate ADC, amplifier or drift estimates again double-counts them. Component budgets are diagnostic alternatives when full-chain data are unavailable. Likewise, fitted $\sigma_0$ is a net-response floor—not automatically an electronic floor—and cannot be compared directly with raw-current component bars.

### Mistake 8: Borrow a literature noise percentage

Noise belongs to a specified observable, protocol, timescale, electrode population, instrument range and processing method. Until it is measured on the intended procedure, report a noise sweep and model thresholds—not an experimental LOD.

### Mistake 9: Treat $3\sigma/S$ as the only definition of LOD

It is a conventional blank-threshold estimate. A formal detection limit also specifies acceptable false-positive and false-negative probabilities and accounts for the sample distribution at the detection concentration.

### Mistake 10: Call $10\sigma/S$ a validated LOQ without checking samples

Quantitation requires acceptable precision and accuracy or recovery at low concentrations. A numerical ten-sigma threshold alone does not establish that performance.

### Mistake 11: Treat a few blanks and the fitted slope as exact

A $z$-based threshold assumes known parameters. With a finite blank series, prediction of a future measurement includes uncertainty in the blank mean and SD; inverse calibration additionally includes slope and intercept uncertainty. Three blanks are insufficient for a precise experimental LOD even if their observed scatter looks small.

## Figures

The notebook exports each figure separately in PNG and SVG formats:

- Additional binding-range figure: `00b_kd_useful_dynamic_range`
1. `01_receptor_occupancy`
2. `02_receptor_as_sensor`
3. `03_blank_noise`
4. `04_calibration_lod_loq`
5. `05_noise_by_concentration`
6. `06_additive_proportional_noise_model`
7. `07_electronic_noise_budget`
8. `08_linearity_residuals`
9. `09_bandwidth_averaging_tradeoff`

## Scope

This is a teaching simulation. For a real analytical method, determine LOD, LOQ, resolution and linear range using an appropriate validation protocol, sufficient independent replicates, matrix-matched samples, accuracy, precision and recovery measurements.
