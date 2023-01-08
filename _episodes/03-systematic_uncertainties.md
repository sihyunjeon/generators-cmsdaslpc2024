---
title: "3 - Analysis and systematic uncertainties"
teaching: 0
exercises: 0
questions:
- "How can we use generator information for quick studies?"
- "What are systematic uncertainties coming from theory inputs?"
- "How can we estimate these uncertainties?"
objectives:
- "Use NanoGEN for exploratory studies and to gain experience with generator related uncertainties (PDF choice, scale, strong coupling constant)"
- "Create plots"
keypoints:
- "PDF uncertainties ..."
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
In the exercise we will be using GenMET and GenDressedLepton to calculate the transverse mass MT which is often used in analysis with leptonically decaying W bosons.

## Where do systematic uncertainties come from?

Several choices of parameters and settings in the event generators have an impact on the predicted overall event yield as well as the acceptance and object kinematics.
The most prominent examples are the choice of the parton distribution function (PDF), the chosen values of the renormalization and factorization scales and the value of the strong coupling constant.

## Estimating systematic uncertainties

Several different PDF sets are usually stored in samples that are centrally produced in CMS.
In order to save space, only one PDF set is stored in NanoAOD and NanoGEN.
For the UL campaign, the NNPDF3.1 NNLO PDF set is used ([LHAPDF](https://lhapdf.hepforge.org/pdfsets.html) number 306000), which contains 103 per-event weights.
The stored weights are ratios w.r.t. the central value, therefore, the weight with index 0 should usually be 1 (if not it means that different PDFs were used for the central ME and the computed variations).
Indices 1-100 are different, linear independent PDF sets that can be used to estimate systematic uncertainties, while indices 101 and 102 correspond to the up and down variations of the strong coupling constant.

Similar sets of variations are available for the renormalization and factorization scales.
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
The different scale and PDF weights are contained in branch

