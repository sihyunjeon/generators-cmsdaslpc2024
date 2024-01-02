---
title: "3 - Optional (MadSpin and BSM UFO model)"
teaching: 10
exercises: 20
questions:
- "What is MadSpin used for?"
- "How do we customize the BSM parameters in the UFO model?"
objectives:
- "Understand the role of MadSpin."
- "Customize BSM parameters for gridpacks."
keypoints:
- "PDF uncertainties can be estimated from a set of different per-event weights. The method depends on the type of PDF set that is used (hessian, MC replicas)"
- "Scale and alphaS variations are another source of uncertainty in the prediction of a simulated sample and can be used to estimate systematic uncertainties"
- "We differentiate between total uncertainties and acceptance / shape uncertainties"
---

# Decay of resonant particles

We will learn some advanced use cases of MadGraph which is MadSpin.
MadSpin, as we saw earlier, is one of the modules that runs through MadGraph interface which handles the decay of resonant particles.
For example, we ran the physics process defined as 
~~~
generate p p > z, z > e+ e-
~~~
{: .output}
Here, `z` is the reonsant particle and hence when one chooses to use MadSpin, above process can be split into two where we first do
~~~
generate p p > z
~~~
{: .output}
and then decay `z` into the electron pair using MadSpin.

But the question still remains, why is MadSpin in any case useful?
The answer lies in NLO calculations in QCD or loop-induced processes.
Let's launch MadGraph prompt shell again.
~~~bash
cd ${GENTUTPATH}/standalone-tut/MG5_aMC_v3_5_2/
./bin/mg5_aMC
~~~
{: .source}

Now try making another simple example that is top pair production.
~~~
import model sm
generate p p > t t~ [QCD]
~~~
{: .output}
It would be not so difficult to realize `[QCD]` has been added in the process definition.
This is a flag which tells MadGraph that you wish to do the calculations at NLO in QCD.

Before going further, try concatenating top decay as below as we did with `Z->ee` example.
~~~
generate p p > t t~, t > w+ b [QCD]
generate p p > t t~ [QCD], t > w+ b
exit
~~~
{: .output}

You will find neither of these working and instead MadGraph complains that such syntaxes are not possible for NLO calculations.
This is where MadSpin becomes necessary, for such cases where resonant particle cannot be decayed can be ddecayed thorugh MadSpin.
Now lets get back to sane example.
~~~
import model sm
generate p p > t t~ [QCD]
output TopPair
launch
4 # "madspin=ON" will work as well
0
~~~
{: .output}

Press `tab` to turn off timer.

<<prompt>>

In the following we will explore a bit the content of NanoGEN samples, and how they can be used for doing first studies for a potentially interesting physics analysis.
The NanoGEN sample we've previously created contains several trees.
You can open the file in root to explore the content, e.g.

~~~bash
cd $CDGPATH/CMSSW_12_4_7/src; cmsenv
root -l $CDGPATH/CMSSW_10_6_19/src/NanoGEN.root
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
For the UL campaign, the [NNPDF3.1](https://nnpdf.mi.infn.it) NNLO PDF set is used ([LHAPDF](https://lhapdf.hepforge.org/pdfsets.html)), which contains 103 per-event weights.
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
python3 analysis/plot_mt.py --input /eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/NanoGEN.root
~~~
{: .source}
where we have produced a NanoGEN root file `/eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/NanoGEN.root` which is a slightly larger file to produce smoother distributions.
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
