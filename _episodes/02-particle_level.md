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

To generate physical collision events, the LHE files we have created need to be showered.
Parton showering accounts for non-perturbative QCD effects with phenomenological models.
The most used tool for parton showering in CMS is Pythia 8 (P8).
While we could again run P8 completely on our own (you can download it from the P8 website, compile and run it on your laptop if you'd like), we are going to use the CMSSW GeneratorInterface in this exercise.
Simply stated, CMSSW can act as a wrapper around various generators and chain their outputs together.
This makes it quite easy to run large-scale production from a gridpack to the finished (Mini/Nano)AOD file. #FIXME

## Producing NanoGEN samples for GEN level analysis

~~~bash
cd $GENTUTPATH/CMSSW_12_4_14_patch2/src
cmsenv
mkdir -p $GENPLOTPATH
cd $GENPLOTPATH
cp $GENTUTPATH/generators-cmsdaslpc2024-git/plotter/*.py ./
python3 lhe-root-plotter.py
~~~{: .source}



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

