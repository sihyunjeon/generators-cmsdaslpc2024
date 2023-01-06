---
title: "2 - Particle Level"
teaching: 0
exercises: 0
questions:
- "Why do we need to do parton showering?"
- "How are simulated samples created in CMS?"
objectives:
- "Generate particle level samples in standard CMS data formats"
- "Inspect NanoGEN files"
keypoints:
- "Pythia8 is the main tool used for parton showering in CMS"
---

# Creating particle level samples from LHE files

To generate physical collision events, the LHE files we have created need to be showered.
Parton showering accounts for non-perturbative QCD effects with phenomenological models.
The most used tool for parton showering in CMS is Pythia 8 (P8).
While we could again run P8 completely on our own (you can download it from the P8 website, compile and run it on your laptop if you'd like), we are going to use the CMSSW GeneratorInterface in this exercise.
Simply stated, CMSSW can act as a wrapper around various generators and chain their outputs together.
This makes it quite easy to run large-scale production from a gridpack to the finished (Mini/Nano)AOD file.

## The CMS sample generation chain

For this exercise, we will use CMSSW_10_6_19 which should already be set up.
The central executable of the [CMSSW event processing model](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework) is `cmsRun` which is controlled through a python configuration file.
The `cmsDriver.py` tool can be used to create configuration files based on the campaign, data tiers and a configuration fragment for the parton shower.

> ## Test
> Note that in order for the fragments to be usable, they must be located in a CMSSW package directory structure like below and you must build your CMSSW release area after adding or changing the file.
> Many people have spent hours trying to figure out why their config fragment either does not work at all or does not what it should because of this.
> **Always remember the folder structure. Always remember to build.**
> 
{: .callout}

We will start with setting up the correct folder structure, copying the configuration fragment from the [gen-cmsdas-2023 repository](https://github.com/danbarto/gen-cmsdas-2023), building using scram and copying the previously produced LHE file.
~~~bash
CONFIG_PATH=${CDGPATH}/CMSSW_10_6_19/src/Configuration/GenProduction/python
mkdir -p ${CONFIG_PATH}
cp ${CDGPATH}/gen-cmsdas-2023/fragments/Hadronizer_TuneCP5_13TeV_generic_LHE_pythia8_cff.py ${CONFIG_PATH}/
cd ${CDGPATH}/CMSSW_10_6_19/src/
cmsenv
scram b
cp ${CDGPATH}/MG5_aMC_v2_6_5/wplustest_4f_LO/Events/run_01/unweighted_events.lhe .
~~~
{: .source}

Now it's time to use `cmsDriver.py` to produce a config file (called `config.py` in this example), and then run it with `cmsRun`.
There are many options, the most important ones defining the produced data tiers (GEN and LHE), the era and conditions, as well as the file names.
As a first step we want to just create the config file to run over 1000 events (`-n 1000`) and not immediately execute it (`--no_exec`).
Afterwards, we run with `cmsRun config.py`.
Some more details on the options are given on this [twiki page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookGenIntro).
~~~bash
cmsDriver.py Configuration/GenProduction/python/Hadronizer_TuneCP5_13TeV_generic_LHE_pythia8_cff.py \
    --mc \
    --eventcontent RAWSIM,LHE \
    --datatier GEN,LHE \
    --conditions 106X_upgrade2018_realistic_v4 \
    --beamspot Realistic25ns13TeVEarly2018Collision \
    --step GEN \
    --geometry DB:Extended \
    --era Run2_2018 \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --filein file:unweighted_events.lhe \
    --fileout file:GEN.root \
    --no_exec \
    --python_filename config.py \
    -n 1000
    
cmsRun config.py
~~~
{: .source}

As you can tell from the file extension the produced output file `GEN.root` is not plain text file anymore, but a root-based Event Data Model (EDM) file.
You can get a quick look at the content by running `edmDumpEventContent GEN.root`, or opening a root browser with `rootbrowse GEN.root`.
More information on generator data formats can be found [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideDataFormatGeneratorInterface).

FIXME: Add the old cmssw plot scripts?

In centrally produced CMS samples going from gridpack to GEN is just a small step towards the final end product, which is usually a MiniAOD or NanoAOD sample.
The GEN samples do not contain any pileup interactions and are not 
We can only look at the ''MC truth'', or generator information.
To obtain reconstructed samples we need to run the detector simulation (SIM), pileup premix, HLT, reconstruction and AOD - miniAOD - nanoAOD steps.
The SIM step is the most time intensive, where processing of every event takes about 1min.

### UL18 example for GEN-NanoAOD steps

FIXME FIXME

## Creating NanoAOD-like GEN samples: NanoGEN

For quick studies it can be beneficial to do quick studies on MC truth level.
To facilitate these studies one can quickly create flat root trees that are similar to NanoAOD, but only contain generator intormation, called NanoGEN.

In this example we will also run the LHE step in `cmsRun`, therefore directly using a pregenerated gridpack as input.
This implies that the fragment needs to have a `ExternalLHEProducer` block, like the one shown below.
~~~python
externalLHEProducer = cms.EDProducer("ExternalLHEProducer",
    args = cms.vstring('/eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/WJetsToLNu_HT-0toInf_slc7_amd64_gcc700_CMSSW_10_6_19_tarball.tar.xz'),
    nEvents = cms.untracked.uint32(5000),
    numberOfParameters = cms.uint32(1),
    outputFile = cms.string('cmsgrid_final.lhe'),
    scriptName = cms.FileInPath('GeneratorInterface/LHEInterface/data/run_generic_tarball_cvmfs.sh')
)
~~~
{: .source}

In the example case we have studied so far, parton radiation was modeled exclusively by P8.
MG5 created a matrix element prediction for production of only a W boson on its own, and P8 then took care of adding radiated jets.
We know that MG5 works well in the perturbative (hard / large momentum) regime, and P8 in the soft (low momentum) regime.
In order to get the best of both worlds for hard extra partons we can generate the full matrix elements not only for the W final state, but also for W + 1 jet, W + 2 jet, ..., W + N jet final states.
In this case, we will rely on P8 only to model jet multiplicities > N (and of course model the hadronization of outgoing partons).

You can copy the prepared fragment into your CMSSW release area and build with scram.
~~~bash
cp ${CDGPATH}/gen-cmsdas-2023/fragments/Hadronizer_TuneCP5_13TeV_MLM_5f_max2j_qCut10_LHE_pythia8_cff.py ${CONFIG_PATH}/
cd ${CDGPATH}/CMSSW_10_6_19/src/
cmsenv
scram b
~~~
{: .source}

The `cmsDriver.py` command for NanoGEN is very similar to the one for a EDM-based GEN configuration.
You can then run `cmsRun NanoGEN.py`.
~~~bash
cmsDriver.py Configuration/GenProduction/python/Hadronizer_TuneCP5_13TeV_MLM_5f_max2j_qCut10_LHE_pythia8_cff.py \
    --mc \
    --eventcontent NANOAODGEN \
    --datatier NANOAOD \
    --conditions auto:mc \
    --beamspot Realistic25ns13TeVEarly2018Collision \
    --step LHE,GEN,NANOGEN \
    --geometry DB:Extended \
    --era Run2_2018 \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --fileout file:NanoGEN.root \
    --no_exec \
    --python_filename NanoGEN.py \
    -n 1000
    
cmsRun NanoGEN.py
~~~
{: .source}

While the `cmsDriver` commands are very similar, the output files are quite different.
You can open the output file `NanoGEN.root` with root and inspect it - you'll see that the data is now arranged in flat root trees.

## Determine the qcut for jet matching

An intrinsic issue with generating the matrix element with extra partons is double counting.
Naively speaking, P8 will still "attach" additional radiation to all events coming out of MG, whether they have additional partons or not.
Therefore, events that have no additional partons in MG will end up having one or two or many jets after showering by P8.
At the same time, we will also get 1-jet events from MG events with one additional parton.
To remove this overlap and define more clearly the "division of labour" between the matrix element and parton shower, a matching procedure is used.
We will be using the so called MLM matching scheme, which uses a freely-chosen scale QCUT as the boundary between ME and PS.
After showering, jet clustering is performed and it is checked whether all jets with kT > QCUT are matched to a ME-level parton.
If not, the event is rejected.

In this exercise, we will see how to set up this method technically and validate the outcome.
