---
title: "2 - Particle Level"
teaching: 10
exercises: 20
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

## Producing NanoGEN samples for GEN level analysis

For this exercise, we will use CMSSW_10_6_19 which should already be set up.
The central executable of the [CMSSW event processing model](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookCMSSWFramework) is `cmsRun` which is controlled through a python configuration file.
The `cmsDriver.py` tool can be used to create configuration files based on the campaign, data tiers and a configuration fragment for the parton shower.

> ## Attention
> Note that in order for the fragments to be usable, they must be located in a CMSSW package directory structure like below and you must build your CMSSW release area after adding or changing the file.
> Many people have spent hours trying to figure out why their config fragment either does not work at all or does not what it should because of this.
> **Always remember the folder structure. Always remember to build.**
> 
{: .callout}

Lets get back to CMSSW release we first used in order to run the parton shower step with Pythia8.

~~~bash
cd $GENTUTPATH/CMSSW_12_4_14_patch2/src
cmsenv
mkdir -p Configuration/GenProduction/python/
cp $GENTUTPATH/generators-cmsdaslpc2024-git/fragment/zto2e_0j.py Configuration/GenProduction/python/
scram b
~~~
{: .source}

Now it's time to use `cmsDriver.py` to produce a config file (called `config.py` in this example), and then run it with `cmsRun`.
There are many options, the most important ones defining the produced data tiers (GEN and LHE), the era and conditions, as well as the file names.
As a first step we want to just create the config file to run over 1000 events (`-n 1000`) and not immediately execute it (`--no_exec`).
Afterwards, we run with `cmsRun config.py`.
Some more details on the options are given on this [twiki page](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookGenIntro).

~~~bash
cmsDriver.py Configuration/GenProduction/python/zto2e_0j.py \
    --python_filename run_zto2e_0j.py \
    --eventcontent NANOAOD \
    --datatier NANOAOD \
    --fileout file:zto2e_0j.root \
    --conditions auto:mc \
    --step LHE,GEN,NANOGEN \
    --no_exec \
    --mc \
    -n 100
cmsRun run_zto2e_0j.py
~~~
{: .source}

Let's analyze the events using simple pyROOT script.

For quick studies it can be beneficial to do quick studies on MC truth level.
To facilitate these studies one can quickly create flat root trees that are similar to NanoAOD, but only contain generator intormation, called NanoGEN.

