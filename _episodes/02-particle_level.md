---
title: "2 - Particle Level"
teaching: 10
exercises: 20
questions:
- "Why do we need parton shower?"
- "How do we produce NanoGEN samples"
objectives:
- "Perform parton shower with LHE file as an input"
- "Perform parton shower with gridpack as an input"
- "Inspect generator level information using NanoGEN files"
keypoints:
- "Pythia8 is the main tool used for parton showering in CMS" #FIXME
- "Events are not physical if it did not go through parton shower"
- "CMS detector simulation is the slowest part of the simulation chain, NanoGEN is a convenient shortcut to do quick physics studies"
- "Proper determination of the qcut is important for samples with additional partons included in the matrix element"
---

# Creating particle level samples from LHE files

As discussed earlier, LHE files itself are not enough to describe physical distributions.
In order to generate physics-wise sensible events, LHE files need to go through the parton shower.
Parton shower, in principle, is responsible for higher order corrections to the hard process (consider `q -> q g` or `e -> e gamma`).
Dominant contributions of such correction happen with collinear or soft emissions.
In CMS, one of the most widely used tool for parton shower is Pythia8 (however, do note that Pythia8 is a multipurpose generator that is able to calculate hard process for certain physics processes).
In this exercise, instead of compiling Pythia8 and running it in standalone mode as we did for MadGraph, we will take Pythia8 that is already compiled under CMSSW environment.

## (1) Running Pythia8 interface in CMSSW

Let's first check which release version of Pythia8 we will be using.

~~~bash
cd $GENTUTPATH/CMSSW_12_4_14_patch2/src
cmsenv
scram tool info pythia8
~~~
{: .source}

You can find out that we are now using Pythia8.306 version that is already compiled in `CMSSW_12_4_14_patch2`.

~~~
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Name : pythia8
Version : 306-494ded5c626b685d055d5b022e918c0c
++++++++++++++++++++

INCLUDE=/cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/pythia8/306-494ded5c626b685d055d5b022e918c0c/include
LIB=pythia8
LIBDIR=/cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/pythia8/306-494ded5c626b685d055d5b022e918c0c/lib
PYTHIA8DATA=/cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/pythia8/306-494ded5c626b685d055d5b022e918c0c/share/Pythia8/xmldoc
PYTHIA8_BASE=/cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/pythia8/306-494ded5c626b685d055d5b022e918c0c
ROOT_INCLUDE_PATH=/cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/pythia8/306-494ded5c626b685d055d5b022e918c0c/include
SYSTEM_INCLUDE+=1
USE=root_cxxdefaults cxxcompiler hepmc3 hepmc lhapdf
~~~
{: .output}

Now we will start building our parton shower fragment in our own directories in order to produce samples by ourselves.

~~~bash
mkdir -p Configuration/GenProduction/python/
cp $GENTUTPATH/generators-cmsdaslpc2024-git/fragment/*.py Configuration/GenProduction/python/
scram b
mkdir -p $GENSHOWERPATH
cd $GENSHOWERPATH
~~~
{: .source}

`cmsDriver.py` executable makes the full configuration file based on the optional arguments it is given with (data tier, campaign, etc.) using the parton shower fragment that is built.
We will create NanoGEN files that are flat ntuples that resembles the NanoAOD data tier but only stored with generator-level information related branches.
It skips the SIM and RECO steps in the middle which makes it convenient to do generator-level studies.
For more information, take a look at [link](https://twiki.cern.ch/twiki/bin/viewauth/CMS/NanoGen).

~~~bash
cmsDriver.py Configuration/GenProduction/python/drellyan-mll50.py \
    --python_filename run_drellyan-mll50.py \
    --eventcontent NANOAOD \
    --datatier NANOAOD \
    --fileout file:drellyan-mll50.root \
    --conditions auto:mc \
    --step LHE,GEN,NANOGEN \
    --no_exec \
    --mc \
    -n 100
~~~
{: .source}

You just created `run_drellyan-mll50.py` that can be executed with `cmsRun` command.
Take a look at `run_drellyan-mll50.py` with `less`, how it evolved from `Configuration/GenProduction/python/drellyan-mll50.py` through `cmsDriver.py`.
It will proudce LHE files, run parton shower to make GEN samples, and then finally convert it to NanoGEN format in one go by doing below.
Note that we will only test with 100 events (`-n 100`) due to time constraints.

~~~bash
cmsRun run_drellyan-mll50.py
~~~
{: .source}

LHE files are first produced using the gridpack we've just produced.

~~~
   ______________________________________     
         Running Generic Tarball/Gridpack     
   ______________________________________     
gridpack tarball path = /uscms/home/sjeon/nobackup/GENTUTORIAL/gridpack-tut/genproductions/bin/MadGraph5_aMCatNLO/drellyan-mll50_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz
%MSG-MG5 number of events requested = 100
%MSG-MG5 random seed used for the run = 234567
%MSG-MG5 thread count requested = 1
%MSG-MG5 residual/optional arguments = 
%MSG-MG5 number of events requested = 100
%MSG-MG5 random seed used for the run = 234567
%MSG-MG5 number of cpus = 1
%MSG-MG5 SCRAM_ARCH version = slc7_amd64_gcc10
%MSG-MG5 CMSSW version = CMSSW_12_4_8
WARNING: Developer's area is created for non-production architecture slc7_amd64_gcc10. Production architecture for this release is el8_amd64_gcc10
**** Following environment variables are going to be unset.
      CMSSW_FULL_RELEASE_BASE

Running MG5_aMC for the 1 time
produced_lhe  0 nevt  100 submitting_event  100  remaining_event  100
run.sh 100 2345670
Now generating 100 events with random seed 2345670 and granularity 1
~~~
{:. output}

Reweight with additional PDF sets given for possible systematic sources.

~~~
INFO: #***************************************************************************
#
# original cross-section: 1855.0899999999972
#     scale variation: +10.6% -11.6%
#     emission scale variation: + 0% - 0%
#     central scheme variation: +3.05e-09% -17.8%
# PDF variation: +1.32% -1.32%
#
#PDF NNPDF31_nnlo_as_0118_nf_4: 1854.1 +1.32% -1.32%
#PDF NNPDF30_nnlo_nf_4_pdfas: 1816.21 +2.13% -2.13%
#PDF NNPDF40_nnlo_nf_4_pdfas: 1854.4 +0.579% -0.579%
#PDF MSHT20nnlo_nf4: 1827.24 +1.16% -1.61%
#PDF PDF4LHC21_40_pdfas_nf4: 1841.73 +1.59% -1.59%
#PDF ABMP16_4_nnlo: 1833.82 +0.925% -0.925%
# dynamical scheme # 1 : 1749 +11.8% -12.8% # \sum ET
# dynamical scheme # 2 : 1749 +11.8% -12.8% # \sum\sqrt{m^2+pt^2}
# dynamical scheme # 3 : 1524.71 +14.7% -15.9% # 0.5 \sum\sqrt{m^2+pt^2}
# dynamical scheme # 4 : 1855.09 +10.6% -11.6% # \sqrt{\hat s}
# PDF 42930 : 1837.5192582008478
#***************************************************************************
~~~
{:. output}

And then Pythia8 is launched with the LHE file created given as an input.
It first prints out the LHE information as we saw directly in the LHE file.

~~~
 --------  PYTHIA Event Listing  (hard process)  -----------------------------------------------------------------------------------
 
    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m 
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13600.000  13600.000
     1       2212  (p+)               -12     0     0     3     0     0     0      0.000      0.000   6800.000   6800.000      0.938
     2       2212  (p+)               -12     0     0     4     0     0     0      0.000      0.000  -6800.000   6800.000      0.938
     3          2  (u)                -21     1     0     5     6   501     0      0.000      0.000     66.079     66.079      0.000
     4         -2  (ubar)             -21     2     0     5     6     0   501     -0.000     -0.000    -36.939     36.939      0.000
     5        -11  e+                  23     3     4     0     0     0     0     30.176    -10.240    -24.793     40.375      0.001
     6         11  e-                  23     3     4     0     0     0     0    -30.176     10.240     53.933     62.644      0.001
                                   Charge sum:  0.000           Momentum sum:      0.000      0.000     29.140    103.019     98.811
~~~
{:. output}

Starts the parton shower on top of the given LHE event.
See how much more information gets printed out.
Remember that parton shower goes lower and lower from the hard process until certain energy threshold (`q -> q g -> q g g g -> q q q g g -> ...`).

~~~
 --------  PYTHIA Event Listing  (complete event)  ---------------------------------------------------------------------------------
 
    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m 
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13600.000  13600.000
     1       2212  (p+)               -12     0     0   265     0     0     0      0.000      0.000   6800.000   6800.000      0.938
     2       2212  (p+)               -12     0     0   266     0     0     0      0.000      0.000  -6800.000   6800.000      0.938
     3          2  (u)                -21     7     7     5     6   501     0      0.000      0.000     66.079     66.079      0.000
     4         -2  (ubar)             -21     8     0     5     6     0   501     -0.000     -0.000    -36.939     36.939      0.000
     5        -11  (e+)               -23     3     4     9     9     0     0     30.176    -10.240    -24.793     40.375      0.001
     6         11  (e-)               -23     3     4    10    10     0     0    -30.176     10.240     53.933     62.644      0.001
     7          2  (u)                -42    12     0     3     3   501     0      0.000      0.000     66.079     66.079      0.000
     8         -2  (ubar)             -41    13    13    11     4     0   502     -0.000     -0.000    -47.025     47.025      0.000
     9        -11  (e+)               -44     5     5    14    14     0     0     25.135    -14.682    -35.225     45.696      0.001
    10         11  (e-)               -44     6     6    15    15     0     0    -30.850      9.646     42.888     53.704      0.001
    11         21  (g)                -43     8     0    16    16   501   502      5.715      5.036     11.392     13.704      0.000
    12          2  (u)                -41   154     0    17     7   503     0      0.000      0.000     78.601     78.601      0.000
    13         -2  (ubar)             -42   155   155     8     8     0   502     -0.000     -0.000    -47.025     47.025      0.000
    14        -11  (e+)               -44     9     9   156   156     0     0     24.576    -15.108    -32.070     43.136      0.001
    15         11  (e-)               -44    10    10   157   157     0     0    -36.010      5.715     44.528     57.551      0.001
    16         21  (g)                -44    11    11   158   158   501   502      4.374      4.014     12.596     13.925      0.000
    17         21  (g)                -43    12     0   159   159   503   501      7.060      5.379      6.521     11.013      0.000
    18         21  (g)                -31    65     0    20    21   505   504      0.000      0.000      2.971      2.971      0.000
    19         21  (g)                -31    66    66    20    21   504   506      0.000      0.000    -25.830     25.830      0.000
    20         21  (g)                -33    18    19    67    67   507   506     -3.816      2.200    -23.877     24.280      0.000
    21         21  (g)                -33    18    19    68    68   505   507      3.816     -2.200      1.018      4.521      0.000
~~~
{:. output}

After 1 event information is printed out, 100 events get processed and finally reports the cross section.

~~~
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
Overall cross-section summary 
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Process		xsec_before [pb]		passed	nposw	nnegw	tried	nposw	nnegw 	xsec_match [pb]			accepted [%]	 event_eff [%]
0		1.855e+03 +/- 1.773e+01		100	100	0	100	100	0	1.855e+03 +/- 1.773e+01		100.0 +/- 0.0	100.0 +/- 0.0
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
Total		1.855e+03 +/- 1.773e+01		100	100	0	100	100	0	1.855e+03 +/- 1.773e+01		100.0 +/- 0.0	100.0 +/- 0.0
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Before matching: total cross section = 1.855e+03 +- 1.773e+01 pb
After matching: total cross section = 1.855e+03 +- 1.773e+01 pb
Matching efficiency = 1.0 +/- 0.0   [TO BE USED IN MCM]
Filter efficiency (taking into account weights)= (100) / (100) = 1.000e+00 +- 0.000e+00
Filter efficiency (event-level)= (100) / (100) = 1.000e+00 +- 0.000e+00    [TO BE USED IN MCM]

After filter: final cross section = 1.855e+03 +- 1.773e+01 pb
After filter: final fraction of events with negative weights = 0.000e+00 +- 0.000e+00
After filter: final equivalent lumi for 1M events (1/fb) = 5.391e-01 +- 5.179e-03

=============================================
~~~
{:. output}


> ## How did the cross section change after parton shower?
> 
> MadGraph reported `# original cross-section: 1855.0899999999972`, 1855pb.
> After running parton shower with Pythia8, same cross section 1855pb is kept.
> Parton shower adds more and more vertices, but why does the cross section remain unchanged?
> 
> > ## Solution
> > Parton shower is unitary.
> > Sum of probability to branch (e.g `q -> q g`) and not branch is 1.
> > Hence, the cross sections is determined by the lowest order input (hard process).
> > 
> {: .solution}
{: .challenge}

## (2) Running Pythia8 interface in CMSSW




For this exercise, we will use CMSSW_10_6_19 which should already be set up.
The central executable of the [CMSSW event processing model](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework) is `cmsRun` which is controlled through a python configuration file.
The `cmsDriver.py` tool can be used to create configuration files based on the campaign, data tiers and a configuration fragment for the parton shower. #FIXME

> ## Attention
> Note that in order for the fragments to be usable, they must be located in a CMSSW package directory structure like below and you must build your CMSSW release area after adding or changing the file.
> Many people have spent hours trying to figure out why their config fragment either does not work at all or does not what it should because of this.
> **Always remember the folder structure. Always remember to build.** #FIXME
> 
{: .callout}

Lets get back to CMSSW release we first used in order to run the parton shower step with Pythia8.

~~~bash
cd $GENTUTPATH/CMSSW_12_4_14_patch2/src
cmsenv
mkdir -p Configuration/GenProduction/python/
cp $GENTUTPATH/generators-cmsdaslpc2024-git/fragment/*.py Configuration/GenProduction/python/
scram b
mkdir -p $GENSHOWERPATH
cd $GENSHOWERPATH
~~~
{: .source}

Now it's time to use `cmsDriver.py` to produce a config file (called `config.py` in this example), and then run it with `cmsRun`.
There are many options, the most important ones defining the produced data tiers (GEN and LHE), the era and conditions, as well as the file names.
As a first step we want to just create the config file to run over 1000 events (`-n 1000`) and not immediately execute it (`--no_exec`).
Afterwards, we run with `cmsRun config.py`.
Some more details on the options are given on this [twiki page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookGenIntro). #FIXME

~~~bash
cmsDriver.py Configuration/GenProduction/python/drellyan-mll50.py \
    --python_filename run_drellyan-mll50.py \
    --eventcontent NANOAOD \
    --datatier NANOAOD \
    --fileout file:drellyan-mll50.root \
    --conditions auto:mc \
    --step LHE,GEN,NANOGEN \
    --no_exec \
    --mc \
    -n 100
cmsRun run_drellyan-mll50.py
~~~
{: .source}

See how many more particles are coming out from a single event trough parton shower

~~~
 --------  PYTHIA Event Listing  (complete event)  ---------------------------------------------------------------------------------
 
    no         id  name            status     mothers   daughters     colours      p_x        p_y        p_z         e          m 
     0         90  (system)           -11     0     0     0     0     0     0      0.000      0.000      0.000  13600.000  13600.000
     1       2212  (p+)               -12     0     0   258     0     0     0      0.000      0.000   6800.000   6800.000      0.938
     2       2212  (p+)               -12     0     0   259     0     0     0      0.000      0.000  -6800.000   6800.000      0.938
     3          4  (c)                -21     7     7     5     6   501     0      0.000      0.000    640.636    640.636      0.000
     4         -4  (cbar)             -21     8     0     5     6     0   501     -0.000     -0.000     -2.746      2.746      0.000
     5        -11  (e+)               -23     3     4     9     9     0     0     40.120      6.785    397.043    399.123      0.001
     6         11  (e-)               -23     3     4    10    10     0     0    -40.120     -6.785    240.847    244.260      0.001
     7          4  (c)                -42    12    12     3     3   501     0      0.000     -0.000    640.636    640.636      0.000
     8         -4  (cbar)             -41    13     0    11     4     0   502      0.000      0.000     -6.150      6.150      0.000
     9        -11  (e+)               -44     5     5    14    14     0     0     46.962     13.127    482.093    484.553      0.001
    10         11  (e-)               -44     6     6    15    15     0     0    -28.891      3.624    102.986    107.023      0.001
    11         21  (g)                -43     8     0    16    16   501   502    -18.071    -16.751     49.407     55.211      0.000
    12          4  (c)                -42    18     0     7     7   501     0      0.000      0.000    640.636    640.636      0.000
    13         -4  (cbar)             -41    19    19    17     8     0   503     -0.000     -0.000    -10.317     10.317      0.000
    14        -11  (e+)               -44     9     9    20    20     0     0     48.906     15.431    513.822    516.375      0.001
    15         11  (e-)               -44    10    10    21    21     0     0    -25.702      7.404     83.280     87.470      0.001
    16         21  (g)                -44    11    11    22    22   501   502    -13.486    -11.316     22.717     28.740      0.000
    17         21  (g)                -43    13     0    23    23   502   503     -9.718    -11.519     10.500     18.368      0.000
    18         21  (g)                -41   115   115    24    12   501   504      0.000      0.000    841.803    841.803      0.000
    19         -4  (cbar)             -42   160   160    13    13     0   503      0.000      0.000    -10.317     10.317      0.000
    20        -11  (e+)               -44    14    14   257   257     0     0     55.623     16.577    519.721    522.951      0.001
    21         11  (e-)               -44    15    15   255   256     0     0    -24.588      7.594     84.493     88.325      0.001
    22         21  (g)                -44    16    16    97    97   501   502    -13.150    -11.259     23.163     28.918      0.000
    23         21  (g)                -44    17    17    95    96   502   503     -9.530    -11.487     10.796     18.421      0.000
    24         -4  (cbar)             -43    18     0   149   150     0   504     -8.354     -1.425    193.314    193.505      1.500
    25         21  (g)                -31    29    29    27    28   505   506      0.000      0.000   2689.184   2689.184      0.000
    26         21  (g)                -31    30     0    27    28   507   508      0.000      0.000     -3.370      3.370      0.000
    27         21  (g)                -33    25    26    31    31   505   508      0.657      8.705   2683.510   2683.524      0.000
    28         21  (g)                -33    25    26    32    32   507   506     -0.657     -8.705      2.304      9.029      0.000
~~~
{: .output}

Let's analyze the events using simple pyROOT script.

For quick studies it can be beneficial to do quick studies on MC truth level.
To facilitate these studies one can quickly create flat root trees that are similar to NanoAOD, but only contain generator intormation, called NanoGEN.

