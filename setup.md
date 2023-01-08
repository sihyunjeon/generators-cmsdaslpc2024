---
title: Setup
---
The following commands are tested on the LPC cluster, using the bash shell.
You can run `echo "$SHELL"` to check that you are indeed running bash.
Add the following lines to your `~/.bash_profile` file
~~~bash
export CDGPATH=${HOME}/nobackup/cmsdas_2023_gen
source /cvmfs/cms.cern.ch/cmsset_default.sh
~~~
{: .source}
Run `bash -l` to restart bash, or alternatively `source ~/.bash_profile` to activate your change. You don't need to rerun this command next time you re-login to the LPC cluster.

We start with pulling and installing the required CMS software (CMSSW).
We use two versions: the older 10.6.19 is used for sample generation,
while we will make use of the more up-to-date tools installed in 12.4.7 when we analyze the samples.
~~~bash
mkdir -p ${CDGPATH}

cd ${CDGPATH}
cmsrel CMSSW_10_6_19
cmsrel CMSSW_12_4_7
~~~
{: .source}

Downloading MadGraph5 v2.6.5 from a CMS mirror, which is the MG version that is currently used in CMS for Run 2 Ultra Legacy and Run 3 samples.
~~~bash
wget https://cms-project-generators.web.cern.ch/cms-project-generators/MG5_aMC_v2.6.5.tar.gz
tar xf MG5_aMC_v2.6.5.tar.gz
rm MG5_aMC_v2.6.5.tar.gz
~~~
{: .source}

Cloning the CMS generator repository:
~~~bash
git clone -b mg265UL https://github.com/cms-sw/genproductions.git genproductions_mg265
~~~
{: .source}

You can check that the MadGraph steering cards of the example we will be using are actually there:
~~~bash
ls -rtl genproductions/bin/MadGraph5_aMCatNLO/cards/examples/wplustest_4f_LO/
~~~
{: .source}

~~~
total 20
-rw-r--r-- 1 dspitzba us_cms   223 Dec 21 08:10 wplustest_4f_LO_proc_card.dat
-rw-r--r-- 1 dspitzba us_cms 15996 Dec 21 08:10 wplustest_4f_LO_run_card.dat
~~~
{: .output}

Cloning extra scripts and config files
~~~bash
git clone https://github.com/danbarto/gen-cmsdas-2023.git
~~~
{: .source}


{% include links.md %}
