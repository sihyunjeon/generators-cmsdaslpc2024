---
title: Setup
---
The following commands are tested on the LPC cluster, using the bash shell.
You can run `echo "$SHELL"` to check that you are indeed running bash.
~~~bash
export GENTUTPATH=${HOME}/nobackup/GENTUTORIAL
export GENMGPATH=${GENTUTPATH}/standalone-tut/MG5_aMC_v2_9_18
export GENGRIDPACKPATH=${GENTUTPATH}/gridpack-tut/genproductions
export GENSHOWERPATH=${GENTUTPATH}/shower-tut
export GENPLOTPATH=${HOME}/GENTUTORIAL/plotter-tut
~~~
{: .source}
First create the Generator Tutorial session directory.
~~~bash
mkdir -p ${GENTUTPATH}
cd ${GENTUTPATH}
git clone https://github.com/sihyunjeon/generators-cmsdaslpc2024-git.git
~~~
{: .source}
Let's start setting up CMS specific environments.
~~~bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
cmsrel CMSSW_12_4_14_patch2
cd CMSSW_12_4_14_patch2/src
cmsenv
cd ${GENTUTPATH}
~~~
{: .source}

Now we will download MadGraph v3.5.2 to run it standalone before discussing "gridpack" which is how CMS produces the samples.
This is to first demonstrate how MadGraph runs as it is before going deeper into CMS specifics.
~~~bash
mkdir -p ${GENTUTPATH}/standalone-tut
cd ${GENTUTPATH}/standalone-tut
wget https://cms-project-generators.web.cern.ch/cms-project-generators/MG5_aMC_v2.9.18.tar.gz
tar -xvf MG5_aMC_v2.9.18.tar.gz
rm MG5_aMC_v2.9.18.tar.gz
cd MG5_aMC_v2_9_18
~~~
{: .source}

As for the second step of tutorial, we will learn how "gridpacks" are how produced using MadGraph.
To do this we need to clone the CMS genproductions repository.
~~~bash
mkdir -p ${GENTUTPATH}/gridpack-tut/
cd ${GENTUTPATH}/gridpack-tut/
git clone --depth 1 https://github.com/cms-sw/genproductions.git
~~~
{: .source}

Return back to the home path to start with standalone tutorial.
~~~bash
cd ${GENTUTPATH}/standalone-tut/MG5_aMC_v2_9_18
cp -r ${GENTUTPATH}/generators-cmsdaslpc2024-git/standalone ./
./bin/mg5_aMC standalone/setup.config
./bin/mg5_aMC standalone/install.config &> install.log &
~~~
{: .source}

> ## When opening a new terminal screen
> 
> In case you escape from the current terminal and open a new one, always execute below before starting the exercises.
> ~~~bash
> cd ${HOME}/nobackup/GENTUTORIAL/generators-cmsdaslpc2024-git/
> source setup.sh
> ~~~
> {: .source}

{% include links.md %}
