---
title: "4 - CMS resources for samples and generators"
teaching: 5
exercises: 10
questions:
- "How do I find centrally produced samples and their status?"
- "How do I obtain a cross section to normalize my sample?"
objectives:
- "Leverage available tools for efficient analysis work"
keypoints:
- "DAS can be used to find samples and their files, number of events for a certain sample etc"
- "McM is used for sample generation management, and can be used by the user to obtain additional information about their samples, e.g. the root gridpack, fragments etc."
- "McM is also a good place to look for example cmsDriver commands"
- "Different sources for x-secs exist within CMS: a CMSSW analyzer and a database"
---

# CMS resources for simulated samples

## How to find samples and related information

Get configurations for a certain sample from [McM](https://cms-pdmv.cern.ch/mcm/requests?page=0&shown=127).
E.g. you want the inclusive W+jets sample, start from a DAS query (requires a valid grid certificate / proxy):
~~~bash
dasgoclient -query="/WJetsToLNu*/RunIISummer20UL18*/MINIAODSIM"
~~~
{: .source}

Alternatively there's also a web-based DAS client: [https://cmsweb.cern.ch/das/](DAS client).

~~~
/WJetsToLNu_012JetsNLO_34JetsLO_EWNLOcorr_13TeV-sherpa/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v4/MINIAODSIM
/WJetsToLNu_012JetsNLO_34JetsLO_EWNLOcorr_13TeV-sherpa/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1_ext1-v2/MINIAODSIM
/WJetsToLNu_0J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v2/MINIAODSIM
/WJetsToLNu_0J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_1J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_1J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_2J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_2J_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_HT-100To200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-100To200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_HT-1200To2500_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-1200To2500_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1_ext1-v1/MINIAODSIM
/WJetsToLNu_HT-1200To2500_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_HT-200To400_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-200To400_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-2500ToInf_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-2500ToInf_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1_ext1-v1/MINIAODSIM
/WJetsToLNu_HT-2500ToInf_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_HT-400To600_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-400To600_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1_ext1-v1/MINIAODSIM
/WJetsToLNu_HT-400To600_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-600To800_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-600To800_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1_ext1-v1/MINIAODSIM
/WJetsToLNu_HT-600To800_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-70To100_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-70To100_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-800To1200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_HT-800To1200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1_ext1-v1/MINIAODSIM
/WJetsToLNu_HT-800To1200_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-100To250_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-100To250_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-250To400_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-250To400_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-400To600_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-400To600_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-600ToInf_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Pt-600ToInf_MatchEWPDG20_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-PUForTRK_TRK_106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-pythia8/RunIISummer20UL18MiniAODv2-PUForTRKv2_TRKv2_106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM
/WJetsToLNu_Wpt-100to200_BPSFilter_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Wpt-200toInf_BPSFilter_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAOD-106X_upgrade2018_realistic_v11_L1v1-v1/MINIAODSIM
/WJetsToLNu_Wpt-200toInf_BPSFilter_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v2/MINIAODSIM
~~~
{: .output}

We want the inclusive LO sample with the latest MiniAOD version (MiniAODv2), hence we pick `/WJetsToLNu_TuneCP5_13TeV-madgraphMLM-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM`.
Plug this name into ''Output Dataset'' in McM, then click on the dataset name (`WJetsToLNu_TuneCP5_13TeV-madgraphMLM-pythia8`).
In ''Select View'' check ''Fragment'' and click on the expand icon under ''Fragment'' (rightmost column) for the request with a Summer20UL18wmLHEGS PrepId.
You can also filter the results directly by appending `?dataset_name=WJetsToLNu_TuneCP5_13TeV-madgraphMLM-pythia8&prepid=*Summer20UL18wmLHE*` to the requests address, `https://cms-pdmv.cern.ch/mcm/requests`.

## Status of samples

[GrASP](https://cms-pdmv.cern.ch/grasp/) is tool to conveniently track the status of your samples.
Just select the campaigns you're interested in (e.g. Run2 UL or Run3) and type the sample name.
You can also tag samples of your analysis so that they are easier to find and keep track of.

## Cross sections

### CMSSW analyzer

In the following, we will use a CMSSW analyzer called GenXSecAnalyzer to compute the cross section of samples.
The analyzer takes a list of EDM files as input (i.e., no NanoAOD or NanoGEN).
Make sure you are in a CMSSW environment
~~~bash
cd ${CDGPATH}/CMSSW_10_6_19/src/
cmsenv
~~~
{: .source}

You can then use the prepared configurations to obtain the cross section for a sample of your liking, e.g.  `/TTWJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-madspin-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM`
~~~bash
dasgoclient -query="file dataset=/TTWJetsToLNu_TuneCP5_13TeV-amcatnloFXFX-madspin-pythia8/RunIISummer20UL18MiniAODv2-106X_upgrade2018_realistic_v16_L1v1-v1/MINIAODSIM | grep file.name" > myfiles.txt
cmsRun $CDGPATH/gen-cmsdas-2023/configs/xsec_ana.py inputFiles="myfiles.txt" maxEvents=100000
~~~
{: .source}
In this example we restrict the maximum number of events to 100k.
This will give us a large enough sample for a reliable result, without running too long (the sample has 10.5M events, you can use DAS to verify this number with `dasgoclient -query="summary dataset=DATASET`).

The `inputFiles` option takes a range of options:
- A single file in your local area: `inputFiles="file:mylocalfile.root"`
- A single published file: `inputFiles="/store/mc/..."`
- Multiple files: `inputFiles="/store/mc/file1,/store/mc/file2"`
- A text file containing one filepath per line: `inputFiles="myfilelist.txt"`

Questions:
- What is the total cross section for your chosen sample? What is the relative uncertainty in this cross section that you obtained?
- Are there different processes listed in the summary? What could those different processes be?
- Does your sample have negative weights? If yes, what is the fraction of events with negative weights?
- The printout also mentions the equivalent luminosity. Do you understand what is meant by that?

### xsec DB

A central database is kept with approved x-secs for centrally produced samples, [XSDB](https://cms-gen-dev.cern.ch/xsdb/).

The CMS Generator's group Cross Section Database Tool (XSDB) is a tool for storing and looking up information related to a specific MC sample witihin CMS.
This tool is designed to complement DAS and MCM, with a direct link from DAS being available to a specific sample.
Anyone with a CERN login can view the XSDB and perform queries for sample information.
However, further action is restricted by e-group permissions.
There exist a user's, approver's, and admin e-groups.
The XSB users are CMS individuals that have permission to insert and modify documents for XSDB.
Approvers have the same user privileges as users, but are primarily tasked with approving records submitted by users.
The admins have the responsibility of maintaining and improving the tool for future use.

There is a large amount of information that can be stored in the database for each sample.
This information includes: cross section value, cross section uncertainty, hadronization tool, matrix element generator, sample contact, cuts used, DASprimary dataset name, and MCM prename, among other metadata.
This information can then be used to help with analysis.
In this exercise, we will simply try some searching through XSDB for a sample, looking at some information stored there and getting familiar with the XSDB.

We would like to search for a sample within XSDB. We'll look for an EXO sample used in the Contact Interaction qqbar to dimuon channel in the search for compositeness.
The sample can be found in DAS with the dataset name: `/CITo2Mu_M1300_CUETP8M1_Lam10TeVDesRR_13TeV-pythia8/RunIISummer16MiniAODv2-PUMoriond17_80X_mcRun2_asymptotic_2016_TrancheIV_v6-v1/MINIAODSIM`
- Query XSDB using: `DAS=CITo2Mu_M1300_CUETP8M1_Lam10TeVDesRR_13TeV-pythia8` in the query field and hitting either enter button on keyboard or clicking "Search"
- Take a minute to explore the items stored for the sample
- You can also choose which metadata are displayed by checking, or unchecking the appropriate boxes in between the search bar and the displayed results
- If we would like to see all of the Contact Interaction samples available we can search: `process_name=cito2mu*`
- Take some time to look through the samples and pagination at the bottom of the results page
- Repeat this exercise for `process_name=ttbar`. This will show a typical search for SM background samples.

It is possible to search for a substring of the item that one would like to look for.
It is important to note that wildcards are supported, however as long as the string is contiguous, it will be accepted by the XSDB query.
XSDB also supports boolean queries.
If we want to query the database for our original sample we could use the following: `process_name=cito2mu && total_uncertainty=21.42` You can also query for your favorite MC sample.
The XSDB twiki can be found here: [XSDB twiki](https://twiki.cern.ch/twiki/bin/viewauth/CMS/XSecDBTool).

