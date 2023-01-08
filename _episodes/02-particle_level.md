---
title: "2 - Particle Level"
teaching: 0
exercises: 0
questions:
- "Why do we need to do parton showering?"
- "How are simulated samples created in CMS?"
objectives:
- "Generate particle level samples in standard CMS data formats"
- "Inspect GEN and NanoGEN files"
keypoints:
- "Pythia8 is the main tool used for parton showering in CMS"
- "Events are unphysical before running hadronization and parton showering"
- "CMS detector simulation is the slowest part of the simulation chain, NanoGEN is a convenient shortcut to do quick physics studies"
- "Proper determination of the qcut is important for samples with additional partons included in the matrix element"
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

> ## Attention
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

An alternative way to inspect the output is to run the CMSSW ParticleListDrawer module, like in the prepared `record_cfg.py` config:
~~~bash
cmsRun ${CDGPATH}/gen-cmsdas-2023/configs/record_cfg.py inputFiles=file:${CDGPATH}/CMSSW_10_6_19/src/GEN.root
~~~
{: .source}
What differences do you notice compared to the LHE events you have seen before?
Can you find the final state leptons in the list?
Tip: There are often multiple copies of each particle.
To find out whether a particle is a final-state particle, status codes are used which are explained in the [pythia documentation](https://pythia.org/latest-manual/ParticleProperties.html).

Similarly, we can't plot distributions using MadAnalysis since EDM is a CMS internal data format that is not supported.
Instead, we will use simple CMSSW modules to plot basic quantities.
Copy the configuration input file to our configuration directory and build:

~~~bash
cp $CDGPATH/gen-cmsdas-2023/configs/WjetsAnalysis_cfi.py Configuration/GenProduction/python/
scram b
cmsRun $CDGPATH/gen-cmsdas-2023/configs/WjetsComparisons_cfg.py inputFiles=file:GEN.root
rootbrowse analyzed.root
~~~
{: .source}

In centrally produced CMS samples going from gridpack to GEN is just a small step towards the end product, which is usually a MiniAOD or NanoAOD sample.
The GEN samples do not contain any pileup interactions, simulation of the interaction of particles with the detector and the subsequent reconstruction.
We can only look at the ''MC truth'', or generator information.
To obtain reconstructed samples we need to run the detector simulation (SIM), pileup premix, HLT, reconstruction and AOD - miniAOD - nanoAOD steps.
The SIM step is the most time intensive, where processing of every event takes about 1min.

### BONUS: UL18 example for GEN --> NanoAOD steps

The only process (or sample) dependent steps in the sample generation chain are the gridpack generation and the gridpack -> LHE -> GEN steps that we've covered in the above example.
If you have a GEN file you can go all the way from GEN -> NanoAOD following the example below.
Note: Only do this if you have completed the other tasks of this exercise as the steps can be time consuming.

#### SIM step
The SIM step for central samples was setup in `CMSSW_10_6_17_patch1`, so we're going to use the same release.
This is the most time consuming step, running the entire CMS detector simulation with GEANT4.
The `cmsDriver.py` command is based on [this example](https://cms-pdmv.cern.ch/mcm/public/restapi/requests/get_test/TOP-RunIISummer20UL18SIM-00119) in [McM](https://cms-pdmv.cern.ch/mcm/requests?page=0).

~~~bash
cp $CDGPATH/CMSSW_10_6_19/src/GEN.root $CDGPATH/output_gen.root
cd $CDGPATH
cmsrel CMSSW_10_6_17_patch1
cd CMSSW_10_6_17_patch1/src
cmsenv
cmsDriver.py \
    --python_filename sim_cfg.py \
    --eventcontent RAWSIM \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --datatier GEN-SIM \
    --fileout file:output_sim.root \
    --conditions 106X_upgrade2018_realistic_v11_L1v1 \
    --beamspot Realistic25ns13TeVEarly2018Collision \
    --step SIM \
    --geometry DB:Extended \
    --filein file:$CDGPATH/output_gen.root \
    --era Run2_2018 \
    --runUnscheduled \
    --no_exec \
    --mc \
    -n 10

cmsRun sim_cfg.py
~~~
{: .source}
Watch out for the `-n 10` option - we will be running over just 10 events from the input file.
You can change this to `-n -1` if you want to run over all events, but be aware that this can be **very** slow.

#### Premix step
Pileup interactions are included in this step.
We can reuse the same CMSSW release.

> ## Attention
> This configuration queries a dataset that is used for pileup premixing.
> You therefore need a valid grid proxy from here on.
{: .callout}

~~~bash
cmsDriver.py \
    --python_filename premix_cfg.py \
    --eventcontent PREMIXRAW \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --datatier GEN-SIM-DIGI \
    --fileout file:output_premix.root \
    --pileup_input "dbs:/Neutrino_E-10_gun/RunIISummer20ULPrePremix-UL18_106X_upgrade2018_realistic_v11_L1v1-v2/PREMIX" \
    --conditions 106X_upgrade2018_realistic_v11_L1v1 \
    --step DIGI,DATAMIX,L1,DIGI2RAW \
    --procModifiers premix_stage2 \
    --geometry DB:Extended \
    --filein file:output_sim.root \
    --datamix PreMix \
    --era Run2_2018 \
    --runUnscheduled \
    --no_exec \
    --mc \
    -n 10

cmsRun premix_cfg.py
~~~
{: .source}

#### HLT step

The HLT simulation is done in an earlier release, we therefore need to setup a new CMSSW release.
This step is usually fairly fast.

~~~bash
cd $CDGPATH
cmsrel CMSSW_10_2_16_UL
cd CMSSW_10_2_16_UL/src
cmsenv

cmsDriver.py \
    --python_filename hlt_cfg.py \
    --eventcontent RAWSIM \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --datatier GEN-SIM-RAW \
    --fileout file:output_hlt.root \
    --conditions 102X_upgrade2018_realistic_v15 \
    --customise_commands 'process.source.bypassVersionCheck = cms.untracked.bool(True)' \
    --step HLT:2018v32 \
    --geometry DB:Extended \
    --filein file:$CDGPATH/CMSSW_10_6_17_patch1/src/output_premix.root \
    --era Run2_2018 \
    --no_exec \
    --mc \
    -n 10
    
cmsRun hlt_cfg.py
~~~
{: .source}

#### RECO to MiniAOD

We now have all the inputs needed to run the digitization and afterwards data-like reconstruction.
For convenience the steps producing intermediate RECO or AOD outputs are collapsed in one.

~~~bash
cd $CDGPATH
cmsrel CMSSW_10_6_20
cd CMSSW_10_6_20/src
cmsenv

cmsDriver.py step2 \
    --filein file:$CDGPATH/CMSSW_10_2_16_UL/src/output_hlt.root \
    --fileout file:miniAOD.root \
    --eventcontent MINIAODSIM \
    --datatier MINIAODSIM \
    --runUnscheduled \
    --conditions 106X_upgrade2018_realistic_v16_L1v1 \
    --step RAW2DIGI,L1Reco,RECO,RECOSIM,PAT \
    --procModifiers run2_miniAOD_UL \
    --nThreads 8 \
    --geometry DB:Extended \
    --era Run2_2018 \
    --python_filename maodv2_cfg.py \
    --no_exec \
    --mc \
    -n 10
    
cmsRun maodv2_cfg.py
~~~
{: .source}

#### MiniAOD to NanoAOD

Almost there!

~~~bash
cd $CDGPATH
cmsrel CMSSW_10_6_27
cd CMSSW_10_6_27/src
cmsenv

cmsDriver.py step3 \
    --mc \
    --filein file:$CDGPATH/CMSSW_10_6_20/src/miniAOD.root \
    --fileout file:nanoAOD.root \
    --conditions 106X_upgrade2018_realistic_v16_L1v1 \
    --step NANO \
    --era Run2_2018,run2_nanoAOD_106Xv2 \
    --eventcontent NANOAODSIM \
    --datatier NANOAODSIM \
    --customise_commands="process.add_(cms.Service('InitRootHandlers', EnableIMT = cms.untracked.bool(False)));process.MessageLogger.cerr.FwkReport.reportEvery=100" \
    --nThreads 8 \
    --python_filename nanov9_cfg.py \
    --no_exec \
    -n -1

cmsRun nanov9_cfg.py
~~~
{: .source}

Congrats, you have generated a CMS sample!
You can easily open the NanoAOD root file with root and inspect the flat root TTrees.

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

The `cmsDriver.py` command for NanoGEN is very similar to the one for a EDM-based GEN configuration.
You can then run `cmsRun NanoGEN.py`.

~~~bash
cp $CDGPATH/gen-cmsdas-2023/fragments/Hadronizer_TuneCP5_13TeV_nanoGEN_pythia8_cff.py Configuration/GenProduction/python/
cd ${CDGPATH}/CMSSW_10_6_19/src/
cmsenv
scram b
cmsDriver.py Configuration/GenProduction/python/Hadronizer_TuneCP5_13TeV_nanoGEN_pythia8_cff.py \
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
    -n 100
    
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
To remove this overlap and define more clearly the "division of labor" between the matrix element and parton shower, a matching procedure is used.
We will be using the so called MLM matching scheme, which uses a freely-chosen scale QCUT as the boundary between ME and PS.
After showering, jet clustering is performed and it is checked whether all jets with kT > QCUT are matched to a ME-level parton.
If not, the event is rejected.

In this exercise, we will see how to set up this method technically and validate the outcome.

We have prepared an MG5 gridpack for a W+jet process, which you can find at this location:
`/eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/WJetsToLNu_HT-0toInf_slc7_amd64_gcc700_CMSSW_10_6_19_tarball.tar.xz`

You can look at the `proc_card.dat` and `run_card.dat` either by unpacking the gridpack, or by opening it directly with e.g. `vim /eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/WJetsToLNu_HT-0toInf_slc7_amd64_gcc700_CMSSW_10_6_19_tarball.tar.xz` (for non-vim users: <ESC>, `: q` to close).

Compare the input cards to the ones we used before.
What is different, what stayed the same? What can you learn from the log file?
How many subprocesses are produced?
What is the cross section?
How does all of this compare to the previous case?
Let's generate some events.
Again, we are going to use cmsDriver, but now with a different fragment.
Compare the fragment to the earlier one to see what is different now.
Before running it, make sure that it has the correct gridpack path in it.

~~~bash
cp ${CDGPATH}/gen-cmsdas-2023/fragments/Hadronizer_TuneCP5_13TeV_MLM_5f_max2j_qCut10_LHE_pythia8_cff.py ${CONFIG_PATH}/
cd ${CDGPATH}/CMSSW_10_6_19/src/
cmsenv
scram b

cmsDriver.py Configuration/GenProduction/python/Hadronizer_TuneCP5_13TeV_MLM_5f_max2j_qCut10_LHE_pythia8_cff.py \
    --mc \
    --eventcontent RAWSIM,LHE \
    --datatier GEN,LHE \
    --conditions 106X_mc2017_realistic_v6 \
    --beamspot Realistic25ns13TeVEarly2017Collision \
    --step LHE,GEN \
    --nThreads 1 \
    --geometry DB:Extended \
    --era Run2_2017 \
    --customise Configuration/DataProcessing/Utils.addMonitoring \
    --fileout file:w2jets_qcut10.root \
    -n 1000
~~~
{: .source}

At the end of the run, CMSSW automatically runs the GenXSecAnalyzer, which is a simple tool to calculate the sample cross-section, check the distribution of weights and give other useful insights.
In the case of jet matching, it also gives the matching efficiencies.
What are the matching efficiencies for each subprocess?
How does the cross-section after matching compare to the cross-section before matching?

If you want to generate matched samples, it is important to check that the matching performs well.
Since we have artificially split the physical process into energy regimes above and below QCUT, we need to make sure that the transition between the two regimes is smooth.
Optimally, it should be impossible to tell from the matched sample what value of QCUT we used.
To test this, we consider the distribution of the differential jet rates (DJRs).
The DJRs correspond to the kt separation of the final clustering step for a given jet multiplicity.
E.g. if we keep clustering an event until it has exaclty 2 jets left, and these 2 jets have a kt separation of 20 GeV, then DJR(1->2) = 20 GeV.
It is called "1->2" because as we decrease the cutoff scale from >20 GeV to <20 GeV, the event turns from a 1-jet into a 2-jet event.
In the genproductions repository, there is a macro that plots these quantities for a given EDM file:

~~~bash
root -l -b -q ${CDGPATH}/genproductions_mg265/bin/MadGraph5_aMCatNLO/macros/plotdjr.C\(\"w2jets_qcut10.root\",\"djr_qcut10.pdf\"\)
~~~
{:. source}

Look at the resulting PDF file and check out the distributions.
The contributions with different numbers of matrix element partons should sum up to give a smooth distribution.
To save on computing time in this exercise, we have pre-generated samples with different values of QCUT.
They are stored in the uscms eos area:

`/eos/uscms//store/user/cmsdas/2023/short_exercises/Generators/wjets_2j/`

What do the DJR distributions look like for the different values of QCUT?
Which one would you choose?
