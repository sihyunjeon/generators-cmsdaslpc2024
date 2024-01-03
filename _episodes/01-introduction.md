---
title: "1 - Introduction"
teaching: 20
exercises: 40
questions:
- "What is Monte Carlo event generator?"
- "Why are we using simulated samples in CMS?"
- "How are the simulated samples produced in CMS?"

objectives:
- "Running standalone MadGraph with simple Z boson process"
- "Producing MadGraph gridpacks using CMS script"
- "Analyzing LHE level information"
  
keypoints:
- "MadGraph is one of the most widely used generator for the hard scattering computations"
- "Standalone MadGraph can run interactively on-the-fly or by importing the predefined text scripts"
- "Gridpacks are useful for large scale productions with consistency guaranteed"
- "LHE level information is not physical and parton shower is needed to describe full physics" 
---

# Introduction and first steps

Although quite old, [link](https://arxiv.org/pdf/1304.6677.pdf) is a great reading material to get a general overview of Monte Carlo event generators.
Monte Carlo event generators are essential components of almost all experimental analyses and are also widely used by theorists and experiments to make predictions and preparations for future experiments.
It is one of the topics where we CMS experimentalists and theorists have the closest connections to, theorists give us predictions and experimentalists verify them with the actual data.
Although Monte Carlo event generators are extremely important tools in HEP, they are often used as black boxes which we more or less treat them as "data".
Our aim is to get the minimal background of how these tools are working and analyze them using the generator level information.

Samples that are used by CMS experiments go through several steps of simulation :
1. Monte Carlo event generator
2. Detector simulation
3. Pileup mixing
4. Trigger emulation
5. Object econstruction

We focus on "1. Monte Carlo event generator" in this tutorial.
Monte Carlo event generator can be further divided into several subpieces as each steps can be factorized and can be handled through separate calculations :
1. Parton distribution function (PDF)
2. Hard scattering (matrix element calculation)
3. Parton shower & hadronization
First of all, LHC is a proton-proton collider, hence we need information on how partons (quarks and gluons) are distributed in the proton (PDF).
Hard scattering is the part where calculations can be treated perturbatively, interactions of incoming partons with the largest momentum transfer (usually the physics process we are interested in).
Parton shower & hadronization further describes how the particles involed in the hard scattering evolve, working downwards to lower momentum scales even to a point where perturbative calculations break down.

## (1) Standalone : DY to ee

In the first exercise, we will run one of the most widely used tool for hard scattering calculations, that is MadGraph5_aMCatNLO, in short MadGraph [link](https://launchpad.net/mg5amcnlo).
MadGraph can perform the calculations for many different physics processes (both SM and BSM) at LO or NLO in QCD.
Because of its easy user interface and flexibility with UFO models, you can test wide variety of physics modeling.
We will now first see how MadGraph runs interactively in standalone mode using simple `DYtoee` process as an example.

Before we proceed, make sure you have first completed the steps described in "Setup" section.
We ended with "Setup" section with commands below, configuring MadGraph with several settings.
~~~bash
cd ${GENTUTPATH}/standalone-tut/MG5_aMC_v2_9_18/
cp -r ${GENTUTPATH}/generators-cmsdaslpc2024-git/standalone ./
./bin/mg5_aMC standalone/setup.config
~~~
{: .source}
Through this, we restrict ourselves to using maximally 2 cores with `set nb_core 2`.
Otherwise MadGraph will interfere with other people's running jobs.
We also installed `ninja` and `collier` which are tools that MadGraph adopts for NLO calculations.

Before going further, go into `input/mg5_configuration.txt` and change `# text_editor = None` by removing `#` and replacing `None` with your favorite text editor. For example, `text_editor = vim`.

Now launch MadGraph prompt shell by doing :
~~~bash
./bin/mg5_aMC
~~~
{: .source}

Now let's try with the simplest `DYtoee` example.
~~~
import model sm
generate p p > e+ e-
output standalone-drellyan-mll50
~~~
{: .output}

First line tells MadGraph that you would like to use the UFO model named `sm` for calculations.
Second line defines which physics process to generate, and in this particular example you are asking for the process where two "quarks from proton" produce a Z/gamma* mediators and then decays into an electron and a positron.
Keep in mind that the calculations are performed on "two quarks" and not "two protons".
The information which translates "protons -> quarks" actually come from PDF.
Last line sets the output directory for the computation results, M50 is to indicate the dilepton mass phase space cut we are about to apply in a few minutes.

Now launch!
~~~
launch
~~~
{: .output}

After MadGraph found all the Feynman diagrams that you targetted, you can see that MadGraph asking you several questions as shown below.
Press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 60 seconds).
~~~
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
`madspin` will be demonstrated later using top pair process example in the third (optional) exercise.
`reweight` is out of scope for this tutorial although it is quite useful for certain BSM scenarios.

Let's move on by pressing `ENTER`.

You can see that MadGraph is asking you several questions as shown below.
Again, press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 90 seconds).
~~~
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
~~~
###################################
## INFORMATION FOR MASS
###################################
Block mass
    5 4.700000e+00 # MB 
    6 1.730000e+02 # MT 
   15 1.777000e+00 # MTA 
   23 9.118800e+01 # MZ 
   25 1.250000e+02 # MH

...

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
~~~
#*********************************************************************
# Number of events and rnd seed                                      *
# Warning: Do not generate more than 1M events in a single run       *
#*********************************************************************
  10000 = nevents ! Number of unweighted events requested
  0   = iseed   ! rnd seed (0=assigned automatically=default))

...

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

...

#*********************************************************************
# Standard Cuts                                                      *
#*********************************************************************
# Minimum and maximum pt's (for max, -1 means no cut)                *
#*********************************************************************
 10.0  = ptl       ! minimum pt for the charged leptons
 -1.0  = ptlmax    ! maximum pt for the charged leptons
 {} = pt_min_pdg ! pt cut for other particles (use pdg code). Applied on particle and anti-particle
 {}     = pt_max_pdg ! pt cut for other particles (syntax e.g. {6: 100, 25: 50})

...

#*********************************************************************
# Minimum and maximum invariant mass for pairs                       *
#*********************************************************************
 0.0   = mmll    ! min invariant mass of l+l- (same flavour) lepton pair
 -1.0  = mmllmax ! max invariant mass of l+l- (same flavour) lepton pair
 {} = mxx_min_pdg ! min invariant mass of a pair of particles X/X~ (e.g. {6:250})
 {'default': False} = mxx_only_part_antipart ! if True the invariant mass is applied only
                       ! to pairs of particle/antiparticle and not to pairs of the same pdg codes.

...

#*********************************************************************
# maximal pdg code for quark to be considered as a light jet         *
# (otherwise b cuts are applied)                                     *
#*********************************************************************
 4 = maxjetflavor    ! Maximum jet pdg code
~~~
{: .output}

Try editting the beam energy (`ebeam1` and `ebeam2`) `6500` to `6800` as we are now running at 13.6TeV beam energy.
When done with editting, escape after saving the changes in the text file.

MadGraph allows you to change settings by interactively typing in below as well.
~~~
set run_card nevents 5000
~~~
{: .output}

Take a look at the run card again and see if number of events to generate (`nevents`) is changed to `5000`.
And change it back to `10000` using same command and check again.

As shown above, there are several phase space cuts set by default (e.g. `10.0 = ptl`).
There is a handy command that removes all phase space cuts at once (instead of doing `set run_card ptl 0`, `set run_card ptj 0`, ... one by one by hand).
~~~
set no_parton_cut
~~~
{: .output}

Take a look at the card again and see if lepton pt cut (`ptl`) is changed to `0`.
Keep in mind that the cuts you give before doing `set no_parton_cut` will be removed by this command.
So don't forget to do `set no_parton_cut` before giving the cuts you wish to give.

As mentioned above, `mll50` in the output directory name stands for dilepton mass cut at 50GeV.

> ## How should we set the dilepton mass cut?
>
> In MadGraph LO run card, the name of dilepton mass variable is `mmll`.
> How should we give 50GeV cut to this value?
>
> > ## Solution
> >
> > ~~~
> > set run_card mmll 50
> > ~~~
> {: .solution}
{: .challenge}

After you verified the desired dilepton mass cut is given, let's really start the computation by moving on, press `ENTER`.

> ## What is the cross section?
>
> Take a close look at what MadGraph logs tell you.
>
> > ## Solution
> >
> > ~~~
> >   === Results Summary for run: run_01 tag: tag_1 ===
> > 
> >      Cross-section :   1493 +- 1.315 pb
> >      Nb of events :  5000
> > ~~~
> {: .solution}
{: .challenge}

Type in `exit` in order to escape from MadGraph shell prompt.
We will take a look at the output LHE file.
~~~bash
less $GENMGPATH/standalone-drellyan-mll50/Events/run_01/unweighted_events.lhe.gz
~~~
{: .shell}

Scroll down to look at the first event (in order to exit, hit `q`).
~~~
<event>
 5      1 +1.4934000e+03 9.10903200e+01 7.54677100e-03 1.30023300e-01
        2 -1    0    0  501    0 +0.0000000000e+00 +0.0000000000e+00 +1.2430983507e+01 1.2430983507e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
       -2 -1    0    0    0  501 -0.0000000000e+00 -0.0000000000e+00 -1.6687025534e+02 1.6687025534e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
       23  2    1    2    0    0 +0.0000000000e+00 +0.0000000000e+00 -1.5443927183e+02 1.7930123884e+02 9.1090315443e+01 0.0000e+00 0.0000e+00
      -11  1    3    3    0    0 -2.3393803385e+01 -7.4187481776e+00 -1.5274153214e+02 1.5470062541e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
       11  1    3    3    0    0 +2.3393803385e+01 +7.4187481776e+00 -1.6977396902e+00 2.4600613435e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
~~~
{: .output}

> ## What does each column mean?
>
> > ## Solution
> > `ID`, `status`, `mother1`, `mother2`, `color`, `anticolor`, `px`, `py`, `pz`, `E`, `mass`, `life time`, and `spin`
> > ~~~output
> > -11  1    3    3    0    0 -2.3393803385e+01 -7.4187481776e+00 -1.5274153214e+02 1.5470062541e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
> > ~~~
> > {: .output}
> > This line tells you that a positron (`ID`) is an outgoing particle (`status`) with Z as its mother (`mother1` and `mother2` : 3rd particle is Z which is `ID=23`) with no color (`color` and `anticolor`), ...
> {: .solution}
{: .challenge}

## (2) Standalone : DY to ll (using particle containers)

Now let's learn about particle containers an easier way to deal with multiple particles.
Launch MadGraph shell prompt with `./bin/mg5_aMC` as we did above.

~~~
import model sm
display multiparticles
~~~
{: .output}

Above command will show several predefined particle containers as below.
~~~
Multiparticle labels:
p = g u c d s u~ c~ d~ s~
j = g u c d s u~ c~ d~ s~
l+ = e+ mu+
l- = e- mu-
vl = ve vm vt
vl~ = ve~ vm~ vt~
all = g u c d s u~ c~ d~ s~ a ve vm vt e- mu- ve~ vm~ vt~ e+ mu+ t b t~ b~ z w+ h w- ta- ta+
~~~
{: .output}

One can redefine or newly define the particle containers by doing :
~~~
define l+ = e+ mu+ ta+
define l- = e- mu- ta-
define myleptons+ = e+ mu+ ta+
define myleptons- = e- mu- ta-
define lpcdas = e+ mu- u d b~
display multiparticles
~~~
{: .output}

Now let's try making the same DY process events but this time allowing all lepton flavors.
Previously we only did electron pair, now we are about to use particle containers collect all possible dilepton contributions including muons and taus.
~~~
generate p p > l+ l-
output standalone-drellyan-mll50-inclusive
launch
0
set run_card nevents 5000
set ebeam1 6800
set ebeam2 6800
set no_parton_cut
set mmll 50
set use_syst False
0
~~~
{: .output}

> ## What is the cross section?
>
> We added 2 new Feynman diagrams (decaying to muon pair and tau pair).
> How should the cross section be adding up from previous value 1493pb?
>
> > ## Solution
> >
> > ~~~
> >   === Results Summary for run: run_01 tag: tag_1 ===
> > 
> >      Cross-section :   4473 +- 5.708 pb
> >      Nb of events :  5000
> > ~~~
> {: .solution}
{: .challenge}

Another way to generate multiple Feynman diagrams is by using `add process` as below.

~~~
import model sm
generate p p > z, z > e+ e-
add process p p > z, z > mu+ mu-
add process p p > z, z > ta+ ta-
~~~
{: .output}

There is one more cool trick to use MadGraph.
Take a look at `standalone/drellyan-mll10.config` file.

~~~
generate p p > e+ e-
output standalone-drellyan-mll10
launch
set nevents 10000
set no_parton_cut
set mmll 10
set use_syst False
0
~~~
{: .output}

Nothing has changed except that `set mmll 50` from above now became `set mmll 10`.
We loosened the dilepton mass cut in this script.

Try this :
~~~bash
./bin/mg5_aMC standalone/drellyan-mll10.config
~~~
{:.source}

MadGraph reads the line and automatically passes the command to MadGraph prompt shell.

Try this again :
~~~bash
./bin/mg5_aMC standalone/drellyan-mll4.config
~~~
{:.source}

> ## What are the cross sections?
>
> How are the cross sections changing when compared to the case where 50GeV cut was given?
>
> > ## Solution
> >
> > ~~~
> > Cross sections get larger as we loosen the cuts drastically (we will later see this through a plot).
> > ~~~
> {: .solution}
{: .challenge}


## (3) Gridpack : DY to ee producing gridpacks for CMS sample production)

As we learned above, running standalone MadGraph is not so difficult.
And it is often useful to do quick tests if you are curious about certain physics processes and its cross sections.
However, CMS relies on billions of events (we produce more than 50B events per year) for physics analysis.
Could we handle all the necessary statistics by interactively running standalone MadGraph?
What if the person who first produced 10M events for `Z->ee` process decides to leave CMS and we decide to make 40M events more?
Can we ensure all the physics settings (which PDF set was chosen, how are kinematic cuts given, etc.) are all kept consistently?
To mitigate such issues, CMS has developed a workflow called gridpacks which is maintained in [link](https://github.com/cms-sw/genproductions).
Gridpacks are precompiled library that contains all necessary executables from MadGraph to produce LHE events.
It is particularly useful for physics processes that require higher multiplicity of particles (we've only tried with `2->2` physics process, think of more complex physics processes e.g. `pp->eejjjj` which is `2->6` with 4 additional QCD particles denoted with `j`) as the precompilation greatly reduces the computing time.
Here, instead of running MadGraph interactively, we will only give inputs to the CMS developed workflow and produce gridpacks and then see how they are the same or different compared to the standalone exercise.

Before we begin, we first need to unset CMSSW environment settings (or open a new terminal) as it might interfere with the scripts in genproductions repository.
~~~bash
eval `scram unsetenv -sh`
~~~
{: .source}

Now lets go into genproductions to try out the gridpack production.
~~~bash
cd $GENGRIDPACKPATH/bin/MadGraph5_aMCatNLO
cp -r ${GENTUTPATH}/generators-cmsdaslpc2024-git/gridpack ./
~~~
{: .source}

Take a look at the cards in `gridpack/drellyan-mll50/` directory.
There are two `.dat` files which are minimal inputs to make gridpacks.
~~~bash
less gridpack/drellyan-mll50/drellyan-mll50_proc_card.dat
less gridpack/drellyan-mll50/drellyan-mll50_run_card.dat
~~~
{: .source}

You would notice that the `proc_card.dat` defines the physics process that we want to calculate is the same as the first example with standalone exercise.
From the `run_card.dat` you would notice that PDF choice that did not exist in the exercise above is now showing up.
~~~
#*********************************************************************
# PDF CHOICE: this automatically fixes also alpha_s and its evol.    *
#*********************************************************************
  'lhapdf'    = pdlabel     ! PDF set                                  
$DEFAULT_PDF_SETS = lhaid
$DEFAULT_PDF_MEMBERS = reweight_PDF     ! if pdlabel=lhapdf, this is the lhapdf number
~~~
{: .output}
`$DEFAULT_PDF_SETS` and `DEFAULT_PDF_MEMBERS` are parsed later automatically through `Utilities/gridpack_helpers.sh` in genproductions.
This is CMS specific part of the code to keep consistent PDF setup among different CMS samples.

As there are many people running the tutorial at once, let's restrict the core usage to 2 again and then start the gridpack production.
~~~bash
export NB_CORE=2
./gridpack_generation.sh drellyan-mll50 gridpack/drellyan-mll50/ pdmv
~~~
{: .source}

Note `pdmv` is only there to restrict the number of cores to use to 2 set with `NB_CORE`, normally you just need to execute it with `./gridpack_generation.sh <process name> <path to card>` without `pdmv`.

Keep in mind that everything is exactly the same as the standalone tutorial except that `gridpack_generation.sh` is merely replacing every interactive commands that we were giving to MadGraph prompt shell.

You can see that MadGraph is downloaded from the web,
~~~
/uscms/home/sjeon/nobackup/GENTUTORIAL/gridpack-tut/genproductions/bin/MadGraph5_aMCatNLO
WARNING: In non-interactive mode release checks e.g. deprecated releases, production architectures are disabled.
WARNING: In non-interactive mode release checks e.g. deprecated releases, production architectures are disabled.
--2024-01-01 17:21:56--  https://cms-project-generators.web.cern.ch/cms-project-generators/MG5_aMC_v2.9.13.tar.gz
Resolving cms-project-generators.web.cern.ch (cms-project-generators.web.cern.ch)... 2001:1458:d00:4e::100:3c0, 188.184.74.207
Connecting to cms-project-generators.web.cern.ch (cms-project-generators.web.cern.ch)|2001:1458:d00:4e::100:3c0|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 26561088 (25M) [application/gzip]
Saving to: 'MG5_aMC_v2.9.13.tar.gz'

     0K .......... .......... .......... .......... ..........  0%  228K 1m54s
    50K .......... .......... .......... .......... ..........  0%  456K 85s
   100K .......... .......... .......... .......... ..........  0%  181M 57s
   150K .......... .......... .......... .......... ..........  0%  301M 42s
~~~
{: .output}

then applying several patches (to mitigate several bugs that are discovered after the release),
~~~
patching file models/loop_qcd_qed_sm/restrict_lepton_masses_no_lepton_yukawas.dat
patching file models/loop_sm/restrict_ckm_no_b_mass.dat
patching file models/sm/restrict_ckm_lepton_masses.dat
patching file models/sm/restrict_ckm_lepton_masses_no_b_mass.dat
patching file models/sm/restrict_ckm_no_b_mass.dat
patching file models/sm/restrict_lepton_masses_no_b_mass.dat
patching file Template/NLO/SubProcesses/MCmasses_PYTHIA8.inc
patching file madgraph/interface/loop_interface.py
patching file madgraph/various/systematics.py
patching file Template/NLO/Source/make_opts.inc
patching file madgraph/iolibs/export_v4.py
patching file madgraph/iolibs/template_files/pdf_opendata.f
patching file madgraph/iolibs/template_files/pdf_wrap_lhapdf.f
~~~
{: .output}

then finding the desired Feynman diagram defined in `proc_card.dat`,
~~~
import model sm
INFO: load particles 
INFO: load vertices 
INFO: Restrict model sm with file MG5_aMC_v2_9_13/models/sm/restrict_default.dat . 
INFO: Run "set stdout_level DEBUG" before import for more information. 
INFO: Change particles name to pass to MG5 convention 
Defined multiparticle p = g u c d s u~ c~ d~ s~
Defined multiparticle j = g u c d s u~ c~ d~ s~
Defined multiparticle l+ = e+ mu+
Defined multiparticle l- = e- mu-
Defined multiparticle vl = ve vm vt
Defined multiparticle vl~ = ve~ vm~ vt~
Defined multiparticle all = g u c d s u~ c~ d~ s~ a ve vm vt e- mu- ve~ vm~ vt~ e+ mu+ t b t~ b~ z w+ h w- ta- ta+
generate p p > e+ e-
INFO: Checking for minimal orders which gives processes. 
INFO: Please specify coupling orders to bypass this step. 
INFO: Trying process: g g > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: u u~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
~~~
{: .output}

and finally giving out the computed cross sections.
~~~
  === Results Summary for run: pilotrun tag: tag_1 ===

     Cross-section :   1836 +- 6.442 pb
     Nb of events :  0
~~~
{: .output}

> ## Why did the cross section change?
>
> You would notice that the cross section has changed from 1493pb to 1836pb.
> What would be the reasons although we ran on exact same physics process?
>
> > ## Solution
> >
> > Most importantly, different PDF set has been used (check by looking at `drellyan-mll50/drellyan-mll50_gridpack/work/gridpack/process/madevent/Cards/run_card.dat`).
> > This will give you totally different assumptions on parton distributions in a proton hence difference in the results.
> > Other minor reasons for the difference could be the use of different MadGraph release version or perhaps different random seed.
> > 
> {: .solution}
{: .challenge}

Now try the same with different gridpack cards named `drellyan-mll50-5fs`.
~~~bash
./gridpack_generation.sh drellyan-mll50-5fs gridpack/drellyan-mll50-5fs/ pdmv
~~~
{:. shell}

This will add a new contribution to the process that is `b b~ > e+ e-` to the calculation as it uses different UFO model that is `sm-no_b_mass`.
Strictly speaking, we are using the same UFO model but adding a restriction (bottom quarks are treated massless) to the model.
Take a look at [restriction_card_tutorial](https://indico.cern.ch/event/239005/attachments/402547/559634/13_02_16_tutomg.pdf) from slide 24 for more information.

~~~
import model sm-no_b_mass
INFO: load particles 
INFO: load vertices 
INFO: Restrict model sm-no_b_mass with file MG5_aMC_v2_9_13/models/sm/restrict_no_b_mass.dat . 
INFO: Run "set stdout_level DEBUG" before import for more information. 
INFO: Change particles name to pass to MG5 convention 
Defined multiparticle p = g u c d s u~ c~ d~ s~
Defined multiparticle j = g u c d s u~ c~ d~ s~
Defined multiparticle l+ = e+ mu+
Defined multiparticle l- = e- mu-
Defined multiparticle vl = ve vm vt
Defined multiparticle vl~ = ve~ vm~ vt~
Pass the definition of 'j' and 'p' to 5 flavour scheme.
Defined multiparticle all = g u c d s b u~ c~ d~ s~ b~ a ve vm vt e- mu- ve~ vm~ vt~ e+ mu+ t t~ z w+ h w- ta- ta+
generate p p > e+ e-
INFO: Checking for minimal orders which gives processes. 
INFO: Please specify coupling orders to bypass this step. 
INFO: Trying process: g g > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: u u~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
INFO: Trying process: u c~ > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: c u~ > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: c c~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
INFO: Trying process: d d~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
INFO: Trying process: d s~ > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: d b~ > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: s d~ > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: s s~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
INFO: Trying process: s b~ > e+ e- WEIGHTED<=4 @1  
INFO: Process u~ u > e+ e- added to mirror process u u~ > e+ e- 
INFO: Process c~ c > e+ e- added to mirror process c c~ > e+ e- 
INFO: Process d~ d > e+ e- added to mirror process d d~ > e+ e- 
INFO: Trying process: d~ b > e+ e- WEIGHTED<=4 @1  
INFO: Process s~ s > e+ e- added to mirror process s s~ > e+ e- 
INFO: Trying process: s~ b > e+ e- WEIGHTED<=4 @1  
INFO: Trying process: b b~ > e+ e- WEIGHTED<=4 @1  
INFO: Process has 2 diagrams 
INFO: Process b~ b > e+ e- added to mirror process b b~ > e+ e- 
5 processes with 10 diagrams generated in 0.032 s
Total: 5 processes with 10 diagrams
output drellyan-mll50-5fs
~~~
{:. output}

Now you would get the following, somewhat increased cross section 1903pb compared to the previous run 1836pb.
~~~
  === Results Summary for run: pilotrun tag: tag_1 ===

     Cross-section :   1903 +- 6.669 pb
     Nb of events :  0
~~~
{:. output}

> ## Why did the cross section change?
>
> Did your `proc_card.dat` add any new processes?
>
> > ## Solution
> >
> > We now have 5 flavor quarks in the proton which was 4 in the previous example by adding up the bottom quark contributions.
> > So we have a new contribution that is `b b~ > e+ e-`.
> > 
> {: .solution}
>
> ## But why did it not scale up so much?
>
> When we ran `p p > e+ e-` and `p p > l+ l-` with standalone MadGraph, the cross sections was roughly 3 times larger.
> When we consider 4 (udcs) and 5 (udcsb) flavor schemes of proton should it not be 5/4 times larger?
>
> > ## Solution
> > 
> > No, keep in mind that proton consists of two up quarks and one down antiquark which are valence quarks.
> > The rest are sea quark contributions, smaller in PDF.
> > Hence, the amount of increment coming from bottom quark contributions are not so large.
> >
> {: .solution}
{: .challenge}

Let's recall why gridpack is a useful package to use.
It is a precompiled library to make the LHE files faster for the given process.
CMS uses gridpacks to produce official samples as it is much easier to keep consistency of the sample.
Now it's time to find out how we make LHE files from gridpacks.
To start with, copy and paste the gridpack to a new temporary directory and untar it.

~~~bash
mkdir test
cp ../drellyan-mll50-5fs_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz ./
tar -xvf drellyan-mll50-5fs_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz
~~~
{:. shell}

You can see that several files were compressed into one tarball.
Among many files, running `runcmsgrid.sh` will take the precompiled library to generate events for the LHE file.
Inputs are given in following order, `./runcmsgrid.sh <nevents> <random seed> <ncores>`.
To make 50 events with random seed 1 using 1 core, execute below.

~~~bash
./runcmsgrid.sh 50 1 1
~~~
{:. shell}

After the run has finished, LHE file with a name `cmsgrid_final.lhe` has been produced.
It's easy to reproduce more statistics as much as we need than running from scratch with standalone MadGraph.

The first step of generator tutorial has finished.
Before moving on to the next step, let's first run below as it takes a bit more time to finish.
~~~bash
cd -
./gridpack_generation.sh drellyan-mll50-01j gridpack/drellyan-mll50-01j/ pdmv
~~~
{:. shell}


> ## Important note : When asking questions to MadGraph authors in Launchpad [link](https://launchpad.net/mg5amcnlo)
>
> Quite often, there are people asking questions "I am working for CMS experiment, I tried to make a gridpack using my awesome BSM UFO file with this awesome BSM particle predictions to use for my analysis. But the script I used in genproductions does not give me functional gridpacks. Please help me."
> 
> Never do this! Setups in genproductions is not MadGraph authors turf!
> 1. They have no responsibility to make our gripdack script work.
> 2. They also have no idea (perhaps some idea) on what we do in genproductions
> 3. They do not share the same computing environment.
>
> Please first consult with GEN conveners or experts through CMS talk [link](https://cms-talk.web.cern.ch/c/physics/gen/102).
> For more constructive iterations and feedbacks, provide your inputs (`.dat` files you used to generate gridpacks) and all possible error logs.
> Also as you all now know how to run standalone MadGraph, test your gridpack inputs with standdalone MadGraph first.
> If it works in standalone run but not in gridpack, it likely could be genproductions issue.
> If it does not work, it likely could be the core MadGraph issue.
> 
{: .callout}


{% include links.md %}

