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
Luckily, these different processes factorize which allows us to separate the treatment of processes happening at different momentum transfer scales. #FIXME

The hard scatter of the incoming partons happens at the highest involved scale, and can be treated perturbatively.
Soft processes that finally lead to the formation of the observed final state hadrons cannot yet be calculated from first principles and therefore need to be modeled. #FIXME
Secondary interactions of other constituent partons of the colliding hadrons are called underlying event.
Although the hard and soft process are distinct, they are connected by an evolutionary Markov process that leads to parton showering.
The partons produced in this process eventually participate in the hadron formation (hadronization) where color singlet states are formed.
Monte Carlo techniques can be used for simulating the Markov process, efficient integration of the high dimensional hard scatter problem, and the hadronization models. #FIXME


## Using Madgraph to simulate the hard scatter process

In the first part of the exercise, we will use the matrix element generator MadGraph5 _aMC@NLO, or in short MG5.
MG5 can perform automatic matrix element predictions for many processes at leading and next-to-leading order accuracy in QCD.
Because of its ease of use for processes both in and beyond the standard model, it is one of the most widely used software tools to model the hard interaction. #FIXME

We will first use the interactive prompt of MG5 to generate proton proton collision events that produce W bosons.
First, log in to a new session on the LPC cluster (`ssh -Y USER@cmslpc-sl7.fnal.gov`).
Make sure you have completed the setup steps!
Then, start the interactive prompt of Madgraph: #FIXME
~~~bash
cd ${GENTUTPATH}/standalone-tut/MG5_aMC_v3_5_2/
cp -r ${GENTUTPATH}/generators-cmsdaslpc2024-git/standalone ./
./bin/mg5_aMC standalone/setup.config
~~~
{: .source}

Before proceeding further, take a look at `input/mg5_configuration.txt` and change `# text_editor = None` by removing `#` and replacing `None` with your favorite text editor. For example, `text_editor = vim`.

Now let's try with the simplest example.
~~~mgshell
import model sm

generate p p > z, z > e+ e-

output ZtoEE
~~~
{: .output}

First line imports the UFO model to use for matrix element calculations.
Second line defines which physics process to generate, this particular example generates the process where two quarks produce a Z boson and then decays into a pair of electrons.

Now start the computation of this process.
~~~bash
launch
~~~
{: .source}

Press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 60 seconds).

You can see that MadGraph asking you several questions as shown below.
~~~mgshell
/================ Description =================|=========== values ===========|====== other options ======\
| 1. Choose the shower/hadronization program   |   shower = Not Avail.        |   Please install module   |
| 2. Choose the detector simulation program    | detector = Not Avail.        |   Please install module   |
| 3. Choose an analysis package (plot/convert) | analysis = Not Avail.        |   Please install module   |
| 4. Decay onshell particles                   |  madspin = OFF               |   ON|onshell|full         |
| 5. Add weights to events for new hypp.       | reweight = OFF               |   ON                      |
\=========================================================================================================/
~~~
{: .output}
As we did not install any other `shower`, `detector`, or `analysis` tools, all are in `Not Avail.` state.
We will learn later how showering will be done under CMSSW and run brief analysis code to analyze the events we produce from this tutorial.
`madspin` will be demonstrated later using top pair process example.
`reweight` is out of scope for this tutorial although it is quite useful for certain BSM scenarios.

Let's move on by pressing `ENTER`.
Press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 90 seconds).

You can see that MadGraph asking you several questions as shown below.
~~~mgshell
Do you want to edit a card (press enter to bypass editing)?
/------------------------------------------------------------\
|  1. param : param_card.dat                                 |
|  2. run   : run_card.dat                                   |
\------------------------------------------------------------/
 you can also
   - enter the path to a valid card or banner.
   - use the 'set' command to modify a parameter directly.
     The set option works only for param_card and run_card.
     Type 'help set' for more information on this command.
   - call an external program (ASperGE/MadWidth/...).
     Type 'help' for the list of available command
 [0, done, 1, param, 2, run, enter path][90s to answer] 
~~~
{: .output}

Let's take a look at the cards and see how the values are set, press `1` and `ENTER` to investigate the parameter settings.
~~~mgshell
###################################
## INFORMATION FOR MASS
###################################
Block mass
    5 4.700000e+00 # MB 
    6 1.730000e+02 # MT 
   15 1.777000e+00 # MTA 
   23 9.118800e+01 # MZ 
   25 1.250000e+02 # MH 

###################################
## INFORMATION FOR DECAY
###################################
DECAY   6 1.491500e+00 # WT 
DECAY  23 2.441404e+00 # WZ 
DECAY  24 2.047600e+00 # WW 
DECAY  25 6.382339e-03 # WH 
~~~
{: .output}

Let's take a look at the cards and see how the values are set, press `2` and `ENTER` to investigate the run settings.
~~~mgshell
#*********************************************************************
# Number of events and rnd seed                                      *
# Warning: Do not generate more than 1M events in a single run       *
#*********************************************************************
  10000 = nevents ! Number of unweighted events requested
  0   = iseed   ! rnd seed (0=assigned automatically=default))

#*********************************************************************
# Collider type and energy                                           *
# lpp: 0=No PDF, 1=proton, -1=antiproton,                            *
#                2=elastic photon of proton/ion beam                 *
#             +/-3=PDF of electron/positron beam                     *
#             +/-4=PDF of muon/antimuon beam                         *
#*********************************************************************
     1        = lpp1    ! beam 1 type
     1        = lpp2    ! beam 2 type
     6500.0     = ebeam1  ! beam 1 total energy in GeV
     6500.0     = ebeam2  ! beam 2 total energy in GeV

#*********************************************************************
# Standard Cuts                                                      *
#*********************************************************************
# Minimum and maximum pt's (for max, -1 means no cut)                *
#*********************************************************************
 10.0  = ptl       ! minimum pt for the charged leptons
 -1.0  = ptlmax    ! maximum pt for the charged leptons
 {} = pt_min_pdg ! pt cut for other particles (use pdg code). Applied on particle and anti-particle
 {}     = pt_max_pdg ! pt cut for other particles (syntax e.g. {6: 100, 25: 50})

#*********************************************************************
# Minimum and maximum invariant mass for pairs                       *
#*********************************************************************
 0.0   = mmll    ! min invariant mass of l+l- (same flavour) lepton pair
 -1.0  = mmllmax ! max invariant mass of l+l- (same flavour) lepton pair
 {} = mxx_min_pdg ! min invariant mass of a pair of particles X/X~ (e.g. {6:250})
 {'default': False} = mxx_only_part_antipart ! if True the invariant mass is applied only
                       ! to pairs of particle/antiparticle and not to pairs of the same pdg codes.

#*********************************************************************
# maximal pdg code for quark to be considered as a light jet         *
# (otherwise b cuts are applied)                                     *
#*********************************************************************
 4 = maxjetflavor    ! Maximum jet pdg code
~~~
{: .output}

Try editting the beam energy (`ebeam1` and `ebeam2) `6500` to `6800` as we are running at 13.6TeV beam energy.
When done with editting, escape after saving the changes in the text file.

MadGraph allows you to change settings by interactively typing in below.
~~~mgshell
set run_card nevents 5000
~~~
{: .output}

Take a look at the card again and see if number of events to generate (`nevents`) is changed to `5000`.

As shown above, there are several phase space cuts set by default (e.g. `10.0 = ptl`).
There is a handy command that removes all phase space cuts at once.
~~~mgshell
set no_parton_cut
~~~
{: .output}

Take a look at the card again and see if lepton pt cut (`ptl`) is changed to `0`.
Now, let's really start the computation by moving on, press `ENTER`.

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
> > When prompted about the run_card, use `${CDGPATH}/gen-cmsdas-2023/cards/wplustest_4f_LO/wplustest_4f_LO_run_card.dat` again.
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

