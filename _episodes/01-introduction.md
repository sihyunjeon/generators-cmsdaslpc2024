---
title: "1 - Introduction"
teaching: 15
exercises: 30
questions:
- "What are Monte Carlo Generators?"
- "Why are we using simulated samples in CMS?"
- "How are simulated samples created in CMS?"
objectives:
- "Use the MadGraph generator in standalone mode and get familiar with the basic syntax"
- "Analyse the produced LHE files"
keypoints:
- "MadGraph is a widely used tool to generate matrix-element predictions for the hard scatter for SM and BSM processes."
- "MadGraph can be used interactively, or steered using text-based cards"
- "Gridpacks are used for large scale productions"
- "MadAnalysis is a tool that allows for quick checks of kinematic distributions"
---

# Introduction and first steps

Samples of simulated events originating from certain processes are essential in high energy physics.
They are used for studies of physics objects, background predictions or signal efficiency and acceptance determinations.
Processes at vastly different energy regimes are involved, from the hard scattering process to hadronization and parton showering.
Luckily, these different processes factorize which allows us to separate the treatment of processes happening at different momentum transfer scales.

The hard scatter of the incoming partons happens at the highest involved scale, and can be treated perturbatively.
Soft processes that finally lead to the formation of the observed final state hadrons cannot yet be calculated from first principles and therefore need to be modeled.
Secondary interactions of other constituent partons of the colliding hadrons are called underlying event.
Although the hard and soft process are distinct, they are connected by an evolutionary Markov process that leads to parton showering.
The partons produced in this process eventually participate in the hadron formation (hadronization) where color singlet states are formed.
Monte Carlo techniques can be used for simulating the Markov process, efficient integration of the high dimensional hard scatter problem, and the hadronization models.


## Using Madgraph to simulate the hard scatter process

In the first part of the exercise, we will use the matrix element generator MadGraph5 _aMC@NLO, or in short MG5.
MG5 can perform automatic matrix element predictions for many processes at leading and next-to-leading order accuracy in QCD.
Because of its ease of use for processes both in and beyond the standard model, it is one of the most widely used software tools to model the hard interaction.

We will first use the interactive prompt of MG5 to generate proton proton collision events that produce W bosons.
First, log in to a new session on the LPC cluster (`ssh -Y USER@@cmslpc-sl7.fnal.gov`).
Make sure you have completed the setup steps!
Then, start the interactive prompt of Madgraph:
~~~bash
cd $CDGPATH/MG5_aMC_v2_6_5/
./bin/mg5_aMC
~~~
{: .source}

Madgraph is configured and steered through text-based cards.
The process definitions can be stored in a card called `proc_card.dat`.
You can look at an example using the following command:
~~~bash
!cat $CDGPATH/gen-cmsdas-2023/cards/wplustest_4f_LO/wplustest_4f_LO_proc_card.dat
~~~
{: .source}

Note: the exclamation mark is used to execute shell commands within Madgraph, e.g. `!cat` in the above example.
Copy/paste the commands line-by-line and pay attention to the output.
~~~
import model sm-ckm

define ell+ = e+ mu+ ta+
define ell- = e- mu- ta-

generate p p > w+, w+ > ell+ vl @0

output wplustest_4f_LO -nojpeg
~~~
{: .output}

The two most important lines of this block are the model import (`import model sm-ckm`) and the instructions on the process to generate (`generate p p > w+, w+ > ell+ vl`).
Within the MG directory you can find a directory `models`, that contains different pre-installed models.
The most obvious one that we are using in the example is `sm` - the standard model at leading order in perturbative QCD.
Model parameters can be configured through ''restriction cards'', in this example `restrict_ckm.dat` loaded through the syntax `sm-ckm`.
This specific restriction card uses a non-diagonal [CKM matrix](https://en.wikipedia.org/wiki/Cabibbo–Kobayashi–Maskawa_matrix) (diagonal CKM is the default otherwise for simplification and faster running).
One great feature of MadGraph is it's flexibility in terms of physics models to use.
To generate a sample using a new physics model one can use the UFO interface. A database of models can be found in the [feynrules model database](http://feynrules.irmp.ucl.ac.be/wiki/ModelDatabaseMainPage).

The practically most relevant part is that MG figures out all relevant Feynman diagrams contributing to a process.
If you are trying to set up a new MC sample, looking at these Feynman diagrams is a great way to check that you actually get the physics you want.
To check them out you can open the individual plots in e.g. `wplustest_4f_LO/SubProcesses/P0_qq_wp_wp_lvl/matrix*.ps` with `gv`, `display` or `evince`.
You can also use the `ps2pdf` program to convert the post script files into PDFs.

Now that MG has figured out the feynman diagrams you can start the actual computation within the MG5 prompt with
~~~bash
launch
~~~
{: .source}

Hint: if you closed the interactive MG session for some reason you can still launch without rerunning the previous commands with
~~~bash
cd $CDGPATH/MG5_aMC_v2_6_5/
./bin/mg5_aMC
launch wplustest_4f_LO
~~~
{: .source}
MG will ask you a few more questions. The first one you can just skip by pressing \<RETURN\>.
Once asked about the `run_card`, one can either use a default run card by just inserting `2` and hitting \<RETURN\> to edit the default `run_card` by hand, or provide a path to a run card of one's choice.
Please provide the path to the pre-made run_card: `${CDGPATH}/gen-cmsdas-2023/cards/wplustest_4f_LO/wplustest_4f_LO_run_card.dat`

What is the cross section determined by Madgraph?

> ## Obtaining the cross section
>
> Define a process (e.g. from the process card above) and launch.
>
> > ## Solution
> >
> > ~~~bash
> > launch wplustest_4f_LO
> > ~~~
> > {: .source}
> >
> > ~~~
> > === Results Summary for run: run_02 tag: tag_1 ===
> >
> >      Cross-section :   2.757e+04 +- 36.81 pb
> >      Nb of events :  10000
> > 
> > INFO: No version of lhapdf. Can not run systematics computation
> > store_events
> > INFO: Storing parton level results
> > INFO: End Parton
> > reweight -from_cards
> > decay_events -from_cards
> > INFO: storing files of previous run
> > INFO: Done
> > ~~~
> > {: .output}
> > The cross section calculated by MG is `2.757e+04 +- 36.81 pb`.
> {: .solution}
{: .challenge}

The main output that MG produces is called ''LHE file''.
The LHE file (Les Houches Event file) is a standard file format that stores process and event information from parton-level event generators.
The documentation can be found [here](https://arxiv.org/pdf/hep-ph/0609017.pdf).

In general, the LHE file contains a header with description of the settings of the generator (e.g. process and run information),
and multiple event blocks (one for each event).
The LHE file is plain text, so it's usually a good idea to use some compression algorithm to save space - MG zips the output by default.

> ## Looking at the LHE output
> Find the LHE file produced by MG and find the first event block.
> > ## Example solution
> > Exit MG, then do
> > ~~~bash
> > find -name '*.lhe.gz'
> > gzip -d ./wplustest_4f_LO/Events/run_01/unweighted_events.lhe.gz
> > less ./wplustest_4f_LO/Events/run_01/unweighted_events.lhe
> > ~~~
> > {: .source}
> > 
> > ~~~
> > <LesHouchesEvents version="3.0">
> > <header>
> > ...
> > </header>
> > ...
> > <event>
> >  5      0 +2.7145900e+04 7.93095700e+01 7.54677100e-03 1.33102200e-01
> >         2 -1    0    0  501    0 +0.0000000000e+00 +0.0000000000e+00 +4.5829549845e+01 4.5829549845e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
> >        -1 -1    0    0    0  501 -0.0000000000e+00 -0.0000000000e+00 -3.4311969734e+01 3.4311969734e+01 0.0000000000e+00 0.0000e+00 1.0000e+00
> >        24  2    1    2    0    0 +0.0000000000e+00 +0.0000000000e+00 +1.1517580112e+01 8.0141519579e+01 7.9309573878e+01 0.0000e+00 0.0000e+00
> >       -13  1    3    3    0    0 -1.6845086581e+01 +2.2368564620e+01 -2.2614075432e+01 3.5993138689e+01 0.0000000000e+00 0.0000e+00 1.0000e+00
> >        14  1    3    3    0    0 +1.6845086581e+01 -2.2368564620e+01 +3.4131655543e+01 4.4148380890e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
> > <mgrwt>
> > <rscale>  0 0.79309574E+02</rscale>
> > <asrwt>0</asrwt>
> > <pdfrwt beam="1">  1        2 0.70507000E-02 0.79309574E+02</pdfrwt>
> > <pdfrwt beam="2">  1       -1 0.52787646E-02 0.79309574E+02</pdfrwt>
> > <totfact> 0.15019241E+05</totfact>
> > </mgrwt>
> > <rwgt>
> > <wgt id='1'> +2.3707085e+04 </wgt>
> > ...
> > </rwgt>
> > </event>
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## MadGraph syntax
> If you want to add another process, e.g. production of W- in the above example, you can add another process with `add process p p > w-, w- > ell- vl~`
>
> A detailed introduction to the syntax is given in [this documentation](https://cp3.irmp.ucl.ac.be/projects/madgraph/wiki/InputEx).
> Some very basic things to keep in mind:
>
> `generate p p > e+ e-` will generate any diagram that is compatible with the used model that produces an electron / positron pair
>
> `generate p p > e+ e- / Z` will exclude diagrams that contain a Z boson as internal paricle
>
> `generate p p > e+ e- $ Z` will exclude the Z boson from appearing in the s-channel (careful about gauge invariance)
>
> `generate p p > Z > e+ e-` will always include a Z boson in the s-channel (careful about gauge invariance)
>
> `generate p p > w+ QED=3` will include all QED contributions, otherwise the QED order is always set to its minimal value
{: .callout}

> ## Bonus: Obtaining the cross section of W boson production
>
> The exercise above only contained W+ bosons (only positive charge).
> Add production of the negatively charged W bosons and calculate the cross section.
> Before running MG, think about what result you would expect, i.e. by how much do you think the cross section should increase.
> Compare the results of W+ and W+/-. What do you conclude?
>
> > ## Solution
> >
> > ~~~bash
> > import model sm-ckm
> > 
> > define ell+ = e+ mu+ ta+
> > define ell- = e- mu- ta-
> > generate p p > w+, w+ > ell+ vl @0
> > add process p p > w-, w- > ell- vl~ @1
> > 
> > output wtest_4f_LO -nojpeg
> > launch 
> > ~~~
> > {: .source}
> >
> > When prompted about the run_card, use `${CDGPATH}/gen-cmsdas-2022/cards/wplustest_4f_LO/wplustest_4f_LO_run_card.dat` again.
> > ~~~
> >   === Results Summary for run: run_01 tag: tag_1 ===
> > 
> >      Cross-section :   4.732e+04 +- 57.08 pb
> >      Nb of events :  10000
> > ~~~
> > {: .output}
> > The cross section calculated by MG is `4.732e+04 +- 57.08 pb`.
> > While one would naiively expect the cross section to double by including W- bosons we only get a cross section that is ~40% larger.
> > The simplified explanation is that the initial state protons contain more up valence quarks than down valence quarks.
> {: .solution}
{: .challenge}


## Using the gridpack workflow

As mentioned previously, interactive running of MG5 is useful for developments and quick tests, but ultimately not practical for large-scale production.
To avoid having to use the interactive mode, one can make use the card structure of MG.
A fully automated workflow for running MG and producing gridpacks is maintained in the [genproductions repository](https://github.com/cms-sw/genproductions).
A gridpack is simply an archive file that contains all the executable MG5 code needed to produce LHE events for a given process, which can then be executed easily on many different grid workers (hence the name).
It has the advantage that once it is created, it is a one-button program to generate events, no thinking required.
In this part of the exercise, we will use the same input cards as before to create a gridpack, run it, and compare the results to before.

Gridpacks are generated using the gridpack_generation script, which we will run in local mode, i.e. on the machine we are currently logged in to.
Note that scripts are provided to run the gridpacks on other computing infrastructures such as the CERN batch system and CMSConnect, which is useful for more complicated processes.

To create a gridpack, we simply call gridpack_generation.sh and pass the process name and card location to it.

> ## Setting and unsetting the CMSSW environment
> You should not have activated a CMSSW environment in this exercise so far.
> However, if you did so before, you need to unset it in order to not interfere with the genproductions script.
> You can run the following command to unset the CMSSW environment, or log in to a clean new session.
> ~~~bash
> eval `scram unsetenv -sh`
> ~~~
> {: .source}
{: .callout}

We will be generating a gridpack with cards similar to the commands we've used in the standalone example above.
The cards are located in the MG section of the genproductions directory

~~~bash
cd $CDGPATH/genproductions_mg265/bin/MadGraph5_aMCatNLO
time ./gridpack_generation.sh wplustest_4f_LO cards/examples/wplustest_4f_LO local
~~~
{: .source}

> ## Naming conventions
> There are certain naming conventions for the input cards.
> For a given process name $NAME, the input cards must be named as $NAME_run_card.dat, $NAME_proc_card.dat, etc...
{: .callout}

The cards for Run2 UL samples that were generated with MG can be found in `bin/MadGraph5_aMCatNLO/cards/production/2017/13TeV/` of the [genproductions repo](https://github.com/cms-sw/genproductions/tree/master/bin/MadGraph5_aMCatNLO/cards/production/2017/13TeV).


~~~bash
mkdir work
cd work
tar xf ../wplustest_4f_LO_slc7_amd64_gcc700_CMSSW_10_6_19_tarball.tar.xz

NEVENTS=10000
RANDOMSEED=12345
NCPU=1
./runcmsgrid.sh $NEVENTS $RANDOMSEED $NCPU
~~~
{: .source}

This will produce an output LHE file called `cmsgrid_final.lhe`.

## Comparing the LHE output

There are multiple ways of analyzing an LHE file, each of which has its own advantages and disadvantages.
For the purpose of this exercise, we will use the most straightforward tool: MadAnalysis (MA).
MA is a tool designed to be used by theorists to analyze parton-level LHE files, particle-level HEPMC files or even events with DELPHES detector simulation.
We can install MA directly from the MG5 console.

~~~bash
cd $CDGPATH/MG5_aMC_v2_6_5/
./bin/mg5_aMC

~~~
{: .source}

You can then install MA from within MG
~~~
install MadAnalysis5 --madanalysis5_tarball=/eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/ma5_v1.9.13.tgz
~~~
{: .source}

> ## MadAnalysis mirror
> You might realize that we're installing MA from a local copy (`/eos/uscms/store/user/cmsdas/2023/short_exercises/Generators/ma5_v1.9.13.tgz`).
> We've experienced some issues with using a [direct download](https://madanalysis.irmp.ucl.ac.be/raw-attachment/wiki/MA5SandBox/ma5_v1.9.13.tgz),
> therefore, we've prepared the local copy.
{: .callout}

After finishing the installation quit MG and run MadAnalysis
~~~bash
cd ../CMSSW_12_4_7/src;cmsenv;cd -
./HEPTools/madanalysis5/madanalysis5/bin/ma5
~~~
{: .source}
Select 2 cores for compiling (remember that you're on a shared computing node!) - this only needs to be done once.
LHE files can be imported as different datasets:
~~~
import wplustest_4f_LO/Events/run_01/unweighted_events.lhe as STANDALONE
import ../genproductions_mg265/bin/MadGraph5_aMCatNLO/work/cmsgrid_final.lhe as GRIDPACK
~~~
{: .source}

Now we can use simple syntax to define the distributions we would like to look at and start the analysis.
~~~
plot PT(mu+) 20 0 100
plot PT(mu-) 20 0 100
plot PT(l+) 20 0 100
plot PT(mu+ vm) 20 0 100
plot M(mu+ vm) 40 40 120
plot M(e+ ve) 40 40 120
plot M(ta+ vt) 40 40 120
plot M(l+ vl) 40 40 120

set main.stacking_method = superimpose
submit
~~~
{: .source}

As you can see from the terminal output, MadAnalysis translates the plotting commands into a C++ analyzer and compiles it. If you want to do some analysis steps that are not supported in the simple command-line syntax we used here, you can also write a C++ analyzer directly (MA calls this "expert mode"). This takes a little more time but gives you much greater flexibility.
The analysis output can be viewed as HTML. Since there is no web browser on cmslpc-sl7, let's copy ANALYSIS_0/Output/ to a local machine and open the HTML output:

firefox ANALYSIS_0/Output/HTML/MadAnalysis5job_0/index.html

What observations do you make? Are the two datasets consistent? What are the shapes of the lepton pT distributions? What is the shape of the pT distribution of the W system? Are these shapes physical?
Feel free to experiment here and plot other quantities you find interesting. You can refer to the [user manual](https://arxiv.org/pdf/1206.1599.pdf) to learn about the syntax.

{% include links.md %}

