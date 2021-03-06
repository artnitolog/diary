<!-- python -m readme2tex --output papers.md --nocdn --rerender papers_raw.md -->
# Конспекты статей

## R. Müller, S. Kornblith, G. Hinton [When Does Label Smoothing Help?](https://arxiv.org/pdf/1906.02629.pdf), 2019

> **General idea (introduced in [Rethinking the Inception Architecture for Computer Vision](https://arxiv.org/pdf/1512.00567.pdf))**: replace the hard (true) targets <img src="svgs/6217c8bb52c9d9f0adb29e37b52dad41.svg?invert_in_darkmode" align=middle width=15.325460699999988pt height=14.15524440000002pt/> in standard cross-entropy
> <p align="center"><img src="svgs/032ff852483955403ffd57c74bc1cb26.svg?invert_in_darkmode" align=middle width=174.499578pt height=48.18280005pt/></p>
> with smoothed (mixture) <img src="svgs/dc9d70b51a74e96eb2a41ecb9accb79a.svg?invert_in_darkmode" align=middle width=153.22870364999997pt height=24.7161288pt/>. The prior distribution over labels <img src="svgs/43f85be4d399d755291f3fbc15bb1db3.svg?invert_in_darkmode" align=middle width=16.67630249999999pt height=14.15524440000002pt/> is now taken uniform: <img src="svgs/a783edab9a82253acf61fa1f0dcf72d9.svg?invert_in_darkmode" align=middle width=53.240112749999994pt height=27.77565449999998pt/>. This regularization mechanism was called LSR (label-smoothing regularization).
The final loss:
> <p align="center"><img src="svgs/e33de40a1910464cfaf85b9ea3368ef2.svg?invert_in_darkmode" align=middle width=489.86565375000004pt height=48.18280005pt/></p>
> **So what?** The model is encouraged to be less confident. LSR prevent the largest logit from becoming much larger than the other. In fact, we reduce overfitting.

In the [paper]((https://arxiv.org/pdf/1906.02629.pdf)), the authors explain why this trick works on model calibration (and against distillation).

> Intuitively, <img src="svgs/97549fbd61ee3133a20edbe3fc217816.svg?invert_in_darkmode" align=middle width=259.13704905pt height=27.6567522pt/>, so each class has a template <img src="svgs/61046e5eadc702c4e6063bccd2508c6f.svg?invert_in_darkmode" align=middle width=19.03453859999999pt height=14.15524440000002pt/>; <img src="svgs/56c25d5a3ffb084ecbe3bbb320a4efc7.svg?invert_in_darkmode" align=middle width=32.38595414999999pt height=26.76175259999998pt/> does not depend on the class, <img src="svgs/563e233a6a64ccb2a677b3ba49b42a6b.svg?invert_in_darkmode" align=middle width=42.84741779999999pt height=26.76175259999998pt/> is usually constant across classes. So the logit <img src="svgs/6669d9c00a615559bfcae4a4e57aadd0.svg?invert_in_darkmode" align=middle width=38.78511779999999pt height=27.6567522pt/> can be thought of as a measure of distance between the penultimate layer and a template.

**Insight**: label smoothing encourages the activations <img src="svgs/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode" align=middle width=9.39498779999999pt height=14.15524440000002pt/> of the penultimate layer to be close to the template of the correct class <img src="svgs/61046e5eadc702c4e6063bccd2508c6f.svg?invert_in_darkmode" align=middle width=19.03453859999999pt height=14.15524440000002pt/> and equally distant to the templates of the incorrect class. This property is observed in authors' visualizations on the datasets CIFAR-10, CIFAR-100 and ImageNet with the architectures AlexNet, ResNet-56 and Inception-v4.

**Label smoothing prevents over-confidence. But does it improve the calibration of the model?**
The authors show that label smoothing reduces ECE and can be effective even without temperature scaling.

## Chuan Guo, Geoff Pleiss, [«On Calibration of Modern Neural Networks»](https://arxiv.org/abs/1706.04599), 2017

Authors explain basic concepts of calibration and compare different methods on real models and datasets. The main result is coolness of temperature scaling in all respects.

### Intro

* In «real-world», classification with high accuracy isn't always enough. In addition to class label, a network is expected to provide a *calibrated confidence* to its prediciton, which means that the score of the class should reflect the real probability. Motivation:
    1. Decision systems like self-driving cars or automated disease diagnosis.
    2. Model interpretability.
    3. Good probability estimates may be incorporated as inputs to other models.
* Modern neural networks are more accurate than they used to be, but less calibrated. Better accuracy doesn't mush better confidence.

### How to estimate (mis)calibration

* Perfect calibration: <img src="svgs/569c727a90a160ff5022fd05071d0751.svg?invert_in_darkmode" align=middle width=241.2473019pt height=24.65753399999998pt/>.
* Reliability histogram — accuracy as a function of confidence.
  1. We split confidences <img src="svgs/c01869f50281dbe87e64a3206b052bf7.svg?invert_in_darkmode" align=middle width=68.57101349999999pt height=23.744328300000017pt/> into bins (by size or length) <img src="svgs/7491fc5ae8253196bdf656ddd60cb334.svg?invert_in_darkmode" align=middle width=82.6106358pt height=22.465723500000017pt/>.
  2. <img src="svgs/fc20ba32921a145829af36aad7be648b.svg?invert_in_darkmode" align=middle width=238.5481461pt height=27.77565449999998pt/> (unbiased and consistent estimator of <img src="svgs/81b07d48a321802b94e1f9aa77bab597.svg?invert_in_darkmode" align=middle width=131.8023366pt height=24.65753399999998pt/>).
  3. <img src="svgs/ef940ac77fe806cf4796555c1ce34ec8.svg?invert_in_darkmode" align=middle width=201.48301139999998pt height=27.77565449999998pt/> — average confidence.
* Miscalibration measure is <img src="svgs/f98a3592cffdb3dd2e7f50c42924b3af.svg?invert_in_darkmode" align=middle width=172.99546109999997pt height=24.65753399999998pt/>. To estimate this value, ECE (expected calibration error) is used:
<p align="center"><img src="svgs/c97deb0f19c4075d0aee735622ae162e.svg?invert_in_darkmode" align=middle width=286.48078964999996pt height=57.32419935pt/></p>

* Sometimes we may wish to estimate only worst-case deviation (expectation is replaced with max). Approximation is MCE (maximum calibration error) — like ECE, but sum is replaced with max.
* Also NLL (negative log likelihood) as a standard measure of a probabilistic model’s quality can be used to estimate calibration.

### Reasons of miscalibration

1. **Model capacity.** Experiments showed that ECE metric grows substantially with increasing depth, filters per layer. But subjectively results seem incomplete.
2. **Batch normalization.**  Authors claim that models trained with Batch Normalization tend tobe more miscalibrated.
3. **Weight decay.** It is common to train models with little weight decay, but calibration is improved when more regularization is added.
4. **NLL**. Overfitting to NLL can be benificial to classification accuracy. At the expense of well-modeled probability.

### Calibration methods

#### Binary classification

* **Histogram binning.** We split (by size or length) uncalibrated confidences into bins and assign a substitute to each one. These substitutes are chosen to minimize bin-wise square loss.
* **Isotonic regression** — generalization of histogram binning: we optimize not only substitutes, but also bin boundaries. The loss function is the same. Optimization is now constrained with condition of monotone boundaries and substitutes.
* **Bayesian binning into quantiles.** Instead of producing a single binning scheme, BBQ performs Bayesian averaging of the probabilities produced by each sheme.
* **Platt scaling.** Uncalibrated predictions are passed through trained sigmoid (shift and scale). In fact, it is a logistic regression with uncalibrated confidences as inputs.

#### Multiclass

* **Binning methods** can be incorparated with one-vs-all method.
* **Matrix scaling.** Before passing to softmax, we apply linear transformation to the logits. This linear transformation may be restricted to coordinate scaling (*vector scaling*) and even a single scalar parameter:
* **Temperature scaling.** Logits are only scaled with *temperature* T. Classification is not affected with this transormation. T is optimized (like Platt scaling) with respect to NLL on validation set (can this be done with folding?)

#### Experiment results

* Authors used state-of-the-art (2017) NN models and different datasets (image classification and document classification).
* The main result is *surprising* effectiveness of temperature scaling. Vector scaling after fitting produced nearly constant parameters, which made it no different than a scalar transformation. So, authors concluded «network miscalibration is intrinsically low dimensional».
* The concept «simple is better than complex» worked in histogram methods too: just histogram binning outperformed BBQ and isotonic regression.
* As for computation time, one-parameter temperature scaling is the fastest one. It is followed by histogram methods.

## J. Platt. [«Probabilistic outputs for support vector machines and comparison to regularized likelihood method»](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.41.1639), 2000

* The general idea of getting conditional probabilities of classes is to make post-processing step: algorithm outputs (margins) are transformed with fitted sigmoid (2 parameters)
* Parameters can be found with MLE. The issues are the choice of calibration set and the method to avoid overfitting
* Using the same set (whole train) to fit the sigmoid can give extremely biased fits, so special hold-out set is preferred (cross-validation will also five a lower variance estimate for sigmoid parameters)
* Overfitting of sigmoid is still possible. Suggested methods:
    1. Regularization (requires a prior model for parameters)
    2. Using smoothed targets (for MAP estimates) instead of {0, 1}

## Zadrozny В., Elkan C. [«Obtaining calibrated probability estimates from decision trees and naive bayesian classifiers»](https://cseweb.ucsd.edu/~elkan/calibrated.pdf), 2001

* Applying **m-estimation** (generalized Laplace smoothing) to correct most distant probabilities and shift towards the base rate(standard one, based on [rule of succession](https://en.wikipedia.org/wiki/Rule_of_succession) adjusts estimates closer to 1/2 which is not reasonable in unbalanced classes). [details](https://www.researchgate.net/publication/220838515_Estimating_Probabilities_A_Crucial_Task_in_Machine_Learning)
* \[DT\] **Curtailment** - taking into account not only class frequencies of the leaves but also of the closest ancestors. can be implemented with unconventional pruning
* **Binning** (histogram method): training examples are sorted by scores into subsets of equal size, and the _estimated corrected probability_ for the test object is now the "accuracy" inside the bin it belongs to
* **Evaluation metrics** of calibration quality: MSE and log-loss are more suitable than lift charts and "profit achieved" (specific problem-related metric)

* DTs' _predict proba_ is just the raw training frequency of final leaf. That's not reliable:
    1. such frequencies are usually shifted towards 0 or 1 since DTs strive to have homogeneous leaves;
    2. without pruning, leaf «capacity» can be small
* Standard DT pruning (maximizing accuracy) methods do not improve quality of probability estimation
