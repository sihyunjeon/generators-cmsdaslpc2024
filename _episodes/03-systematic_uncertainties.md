---
title: "3 - Analysis and systematic uncertainties"
teaching: 10
exercises: 20
questions:
- "How can we use generator information for quick studies?"
- "What are systematic uncertainties coming from theory inputs?"
- "How can we estimate these uncertainties?"
objectives:
- "Use NanoGEN for exploratory studies and to gain experience with generator related uncertainties (PDF choice, scale, strong coupling constant)"
- "Create plots using pre-installed python based tools"
- "Study some theory related systematic uncertainties"
keypoints:
- "PDF uncertainties can be estimated from a set of different per-event weights. The method depends on the type of PDF set that is used (hessian, MC replicas)"
- "Scale and alphaS variations are another source of uncertainty in the prediction of a simulated sample and can be used to estimate systematic uncertainties"
- "We differentiate between total uncertainties and acceptance / shape uncertainties"
---

# Analysis of generator truth samples

In the following we will explore a bit the content of NanoGEN samples, and how they can be used for doing first studies for a potentially interesting physics analysis.
The NanoGEN sample we've previously created contains several trees.
You can open the file in root to explore the content, e.g.

~~~bash
cd $CDGPATH/CMSSW_12_4_7/src; cmsenv
root -l $CDGPATH/CMSSW_10_6_9/src/NanoGEN.root
~~~
{: .source}

Within root, you can use `.ls` to list the content.
If you're interested in the branches contained in a certain tree you can 
~~~bash
.ls
Events->GetListOfBranches()->ls()
~~~
{: .source}

The NanoGEN sample contains, amongst others, collections of all the generated particles (GenPart), jets with different cone size parameters (GenJet, GenJetAK8), generated missing transverse momentum (GenMET) and generated leptons (GenDressedLepton).
In the exercise we will be using GenMET and GenDressedLepton to calculate the transverse mass of the lepton+MET system, MT, which is often used in analysis with leptonically decaying W bosons.

## Where do systematic uncertainties come from?

Several choices of parameters and settings in the event generators have an impact on the predicted overall event yield as well as the acceptance and object kinematics.
The most prominent examples are the choice of the parton distribution function (PDF), the chosen values of the renormalization and factorization scales and the value of the strong coupling constant.

## Estimating systematic uncertainties

Several different PDF sets are usually stored in samples that are centrally produced in CMS.
In order to save space, only one PDF set is stored in NanoAOD and NanoGEN.
For the UL campaign, the NNPDF3.1 NNLO PDF set is used ([LHAPDF](https://lhapdf.hepforge.org/pdfsets.html)), which contains 103 per-event weights.
The stored weights called `LHEPdfWeight` are ratios w.r.t. the central value, therefore, the weight with index 0 should usually be 1 (if not it means that different PDFs were used for the central ME and the computed variations and extra care needs to be taken).
Indices 1-100 are different, linear independent PDF sets that can be used to estimate systematic uncertainties, while indices 101 and 102 correspond to the up and down variations of the strong coupling constant.

> ## Hessian and MC replicas
> Two different approaches for PDF sets exist: MC replicas and hessian eigenvectors.
> While the hessian sets used in CMS in (most) UL samples allows for estimation of the total uncertainty by using the squared sum of the individual variations,
> the situation is more ambigous for MC replicas.
{: .callout}

Similar sets of variations are available for the renormalization and factorization scales, `LHEScaleWeight`.
You can find the details about the variations in the branch documentation within the root file (see above on how the get the list of branches in root).

## Estimate systematic uncertainties using generator information

The [script](https://github.com/danbarto/gen-cmsdas-2023/blob/main/analysis/plot_mt.py) `plot_mt.py` in `$CDGPPATH/gen-cmsdas-2023/analysis/` calculates MT for events with exactly one generated electron or muon.
It can be run after setting the proper CMSSW environment to load all the necessary packages
~~~bash
cd $CDGPATH/CMSSW_12_4_7/src; cmsenv
cd $CDGPATH/gen-cmsdas-2023/
python3 analyis/plot_mt.py --input INPUTFILE.root
~~~
{: .source}
where you can replace `INPUTFILE.root` with the path to your NanoGEN.root file, or use `/FIXME/NanoGEN.root` which is a slightly larger file to produce smoother distributions.
In order to produce the MT histogram we reweight the sample to an integrated luminosity corresponding to the 2018 data taking (60/fb).
The different scale and PDF weights are contained in branches `LHEScaleWeight` and `LHEPdfWeight`.

Plotting the impact of varying the factorization scale up and down while keeping the renormalization scale constant is already implemented.
A common approach to assign the uncertainty in CMS is to use the envelope of all the variations as systematic uncertainty, neglecting the anticorrelated case.
First, look at which weight indices you would need to get histograms of all the variations.
You can then get the maximum and minimum variations in every bin of the MT distribution and obtain the systematic uncertainty.

> ## Inclusive and acceptance / shape effects
> With the above approach we obtain systematic uncertainties for the total effect of varying the used scales.
> In cases where we already assign an uncertainty on the inclusive cross section of a process we would double count a large fraction of the uncertainty.
> Therefore, we would keep the inclusive normalization for each variation unchanged w.r.t. the nominal value, and only account the acceptance and shape effects.
{: .callout}

Bonus exercise: Compare the overall scale uncertainty with just the acceptance / shape uncertainty.
What do you observe? Is this expected for this example?
