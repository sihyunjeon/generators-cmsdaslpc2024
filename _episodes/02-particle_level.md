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
{: .output}

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
{: .output}

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
{: .output}

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
{: .output}


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

Previously, we saw the histogram of dilepton system's transverse momentum using LHE information.
And we claimed it only being populated at 0GeV was not a physical distribution.
After parton shower, using the NanoGEN sample, let's see how the distribution changed.
Due to time constraints, tutors prepared samples with 40000 events in `/tmp/GENTUTORIAL/drellyan-mll50.root` for plotting purposes.

~~~bash
cd $GENPLOTPATH
python3 nanogen-plotter.py
~~~
{: .source}

> ## How did the distribution change?
> 
> Where did the dilepton system acquire transverse momentum from?
> 
> > ## Solution
> > Incoming partons from protons also go through parton shower which is named "initial state radiation (ISR)".
> > 
> {: .solution}
>
> ## What is GenDressedLepton?
>
> What happens to leptons during parton shower?
> Are leptons kept stable during parton shower as it does not participate in strong interactions?
>
> > ## Solution
> > Parton shower, despite its choice of the naming, "parton", also includes QED shower such as `e -> e gamma`.
> > Dressed leptons (`GenDressedLepton` collection in NanoGEN) is an object formed of the charged lepton and photons that are close to it.
> {: .solution}
>
{: .challenge}


## (2) Jet merging samples

Hard process calculation has advantage in modeling of hard jets and heavy particle decays while parton shower is great for describing collinear and soft emissions.
For more realistic and reliable physics modeling of hard jets, for example in DY events, MadGraph can be used as below.

~~~
generate p p > e+ e- @0
add process p p > e+ e- j @1
~~~
{: .output}

With such syntaxes, MadGraph produces DY process with 0 and 1 hard jet in the event.
If this sample goes through parton shower, as some portion of events (dentoed with `@1`) readily involves hard jet, it would be better at describing DY process with hard jet.
However consider the event `@0` emitting QCD particles from initial state radiation that could possibly form a jet that is hard enough.
Such phase space inherently possesses a problem of double counting as "DY with hard jet" event could come from both `@0` and `@1`.
To mitigate such issues and remove double counting of phase space contributions, jet merging technique is used.
Jet merging is set up with an artificial cut threshold called jet merging scale.
This scale decides whether an event will be accepted or not from both `@0` and `@1`.
Finally, only accepted events from the two processes will be merged and form one sample.
Very roughly, jet merging scale can be thought as the momentum of a jet.
If a jet in the event is hard enough above the threshold, events from `@0` are rejected while only accepting from `@1`.
On the other hand, if a jet in the event is not too hard below the threshold, events from `@0` are only accepted while rejecting `@1`.

Now let's take a look at the gridpack we produced before we started the parton shower exercises.
~~~bash
ls $GENGRIDPACKPATH/bin/MadGraph5_aMCatNLO/drellyan-mll50-01j_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz
cd $GENSHOWERPATH
mkdir jet_merging
cd jet_merging
cp $GENGRIDPACKPATH/bin/MadGraph5_aMCatNLO/drellyan-mll50-01j_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz ./
tar -xvf drellyan-mll50-01j_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz
~~~
{: .source}

Take a look at the cards in `InputCards` directory.
Most notably, `run_card.dat` had a different setting compared to the other gridpacks we've produced.

~~~
#*********************************************************************
# Matching - Warning! ickkw > 1 is still beta
#*********************************************************************
  1     = ickkw ! 0 no matching, 1 MLM, 2 CKKW matching
~~~
{: .output}

This flag tells MadGraph that the LHE files we are going to produce will later be going through jet merging in order to avoid double countings.

~~~
#*********************************************************************
# Jet measure cuts                                                   *
#*********************************************************************
  10.0  = xqcut ! minimum kt jet measure between partons
~~~
{: .output}

When jet merging is turned on, `xqcut` needs to be set which presample the events for efficient jet merging.
Remember that some portion of events will be later discarded and never going to be used.
So there is no point of producing events that involve jets with too low energy scale at this LHE level since these will likely be removed.

Try producing 100 events using this gridpack as we did before with command `./runcmsgrid.sh 100 1 1`.

> ## What is the cross section?
> 
> MadGraph reported `# original cross-section: 2928.1100000000024`, 2928pb which is significantly larger than previous values that were below 2000pb.
> How can this be explained?
> 
> > ## Solution
> > The cross section reported from MadGraph is before we run the parton shower.
> > During parton shower, jet merging will be performed and thus some portion of events will be discared.
> > This will be reflected into overall normalization and the cross section will be smaller than what we now see.
> > 
> {: .solution}
{: .challenge}

Before running the parton shower, let's look at the pythia fragment that should be used for the parton shower with jet merging.
Compare `$GENTUTPATH/CMSSW_12_4_14_patch2/src/Configuration/GenProduction/python/drellyan-mll50-01j.py` and the one we used earlier `$GENTUTPATH/CMSSW_12_4_14_patch2/src/Configuration/GenProduction/python/drellyan-mll50.py`.
You will notice huge block of new lines are added to `drellyan-mll50-01j.py`.

~~~
        processParameters = cms.vstring(
            'JetMatching:setMad = off',
            'JetMatching:scheme = 1',
            'JetMatching:merge = on',
            'JetMatching:jetAlgorithm = 2',
            'JetMatching:etaJetMax = 5.',
            'JetMatching:coneRadius = 1.',
            'JetMatching:slowJetPower = 1',
            'JetMatching:doShowerKt = off',
            'JetMatching:qCut = 19.',
            'JetMatching:nQmatch = 4',
            'JetMatching:nJetMax = 1',
            'TimeShower:mMaxGamma = 4.0'
        ),
~~~
{: .output}

Most of the lines could be treated as template for jet merging samples using MadGraph at LO and Pythia8 (for further information, (link)[https://pythia.org/latest-manual/JetMatching.html] and (link)[http://hep.ucsb.edu/people/cag/Matching.pdf] would be useful).
Here `JetMatching:qCut = 19.`, line defines the threshold to decide whether the event should be accepted or not.
Again, although not exact, one can think of this as the threshold for the momentum scale of a jet in the event.
If a jet momentum in the event is above 19GeV, event is only accepted from `p p > e+ e- j` type of events.
If a jet momentum in the event is below 19GeV, event is only accepted from `p p > e+ e-` type of events.

Now let's give `-n 1000` as an option to `cmsDriver.py`.
This will first create an LHE file with 1000 events and this will be given as an input for Pythia8.

~~~bash
cmsDriver.py Configuration/GenProduction/python/drellyan-mll50-01j.py \
    --python_filename run_drellyan-mll50-01j.py \
    --eventcontent NANOAOD \
    --datatier NANOAOD \
    --fileout file:drellyan-mll50-01j.root \
    --conditions auto:mc \
    --step LHE,GEN,NANOGEN \
    --no_exec \
    --mc \
    -n 1000
cmsRun run_drellyan-mll50-01j.py
~~~
{: .source}

Cross sections before and after jet merging will be reported as below.

~~~
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
Overall cross-section summary 
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Process		xsec_before [pb]		passed	nposw	nnegw	tried	nposw	nnegw 	xsec_match [pb]			accepted [%]	 event_eff [%]
0		1.833e+03 +/- 1.197e+01		467	467	0	623	623	0	1.374e+03 +/- 3.305e+01		75.0 +/- 1.7	75.0 +/- 1.7
1		1.099e+03 +/- 1.713e+01		159	159	0	377	377	0	4.636e+02 +/- 2.887e+01		42.2 +/- 2.5	42.2 +/- 2.5
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
Total		2.932e+03 +/- 2.090e+01		626	626	0	1000	1000	0	1.835e+03 +/- 4.673e+01		62.6 +/- 1.5	62.6 +/- 1.5
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Before matching: total cross section = 2.932e+03 +- 2.090e+01 pb
After matching: total cross section = 1.835e+03 +- 4.673e+01 pb
Matching efficiency = 0.6 +/- 0.0   [TO BE USED IN MCM]
Filter efficiency (taking into account weights)= (626) / (626) = 1.000e+00 +- 0.000e+00
Filter efficiency (event-level)= (626) / (626) = 1.000e+00 +- 0.000e+00    [TO BE USED IN MCM]

After filter: final cross section = 1.835e+03 +- 4.673e+01 pb
After filter: final fraction of events with negative weights = 0.000e+00 +- 0.000e+00
After filter: final equivalent lumi for 1M events (1/fb) = 5.449e-01 +- 1.388e-02

=============================================
~~~
{: .output}

First two lines, `Process` denoted `0` and `1` are indicators for `p p > e+ e-` and `p p > e+ e- j` processes, respectively.
For `0`, 623 events were `tried` and 467 passed, which means jet merging procedure accepted 467 events out of 623 events from `0`.
For `1`, jet merging procedure accepted 159 events out of 377 events.
Note that the sum of `tried` is 1000 which was the given input with `-n 1000` at the LHE level.
After jet merging has been done, events are discarded and the final cross section is reported `After filter: final cross section = 1.835e+03 +- 4.673e+01 pb` 1835pb which is dropped from the LHE given value `Before matching: total cross section = 2.932e+03 +- 2.090e+01 pb` 2932 pb.

## (3) Analyzing generator level information (after parton shower)



