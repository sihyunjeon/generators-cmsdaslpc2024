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


## (1) Standalone : Z boson to electron pair

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

Launch MadGraph prompt shell by doing:
~~~bash
./bin/mg5_aMC
~~~
{: .source}

Now let's try with the simplest example.
~~~output
import model sm

generate p p > z, z > e+ e-

output ZtoEE
~~~
{: .output}

First line imports the UFO model to use for matrix element calculations.
Second line defines which physics process to generate, this particular example generates the process where two quarks produce a Z boson and then decays into a pair of electrons.

Now start the computation of this process.
~~~output
launch
~~~
{: .output}

Press `tab` to turn off the timer (otherwise, MadGraph will move on by itself after 60 seconds).

You can see that MadGraph asking you several questions as shown below.
~~~output
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
~~~output
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
~~~output
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
~~~output
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

Try editting the beam energy (`ebeam1` and `ebeam2`) `6500` to `6800` as we are running at 13.6TeV beam energy.
When done with editting, escape after saving the changes in the text file.

MadGraph allows you to change settings by interactively typing in below.
~~~output
set run_card nevents 5000
~~~
{: .output}

Take a look at the card again and see if number of events to generate (`nevents`) is changed to `5000`.

As shown above, there are several phase space cuts set by default (e.g. `10.0 = ptl`).
There is a handy command that removes all phase space cuts at once.
~~~output
set no_parton_cut
~~~
{: .output}

Take a look at the card again and see if lepton pt cut (`ptl`) is changed to `0`.
Now, let's really start the computation by moving on, press `ENTER`.

> ## What is the cross section?
>
> Define a process (e.g. from the process card above) and launch.
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
~~~output
less $GENMGPATH/ZtoEE/Events/run_01/unweighted_events.lhe.gz
~~~
{: .output}

Scroll down to look at the first event and exit with `q`.
~~~output
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


## (2) Standalone : Z boson to lepton pair (using particle containers)

Now let's learn about particle containers, launch MadGraph shell prompt.

~~~output
import model sm

display multiparticles
~~~
{: .output}

Above command will show several predefined particle containers as below.

~~~output
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

~~~output
define l+ = e+ mu+ ta+
define l- = e- mu- ta-
define myleptons+ = e+ mu+ ta+
define myleptons- = e- mu- ta-

display multiparticles
~~~
{: .output}

Now let's try making Z boson to dilepton pair events (previously we only did electron pair).

~~~output
generate p p > z, z > l+ l-

output ZtoLL
launch
0
set run_card nevents 5000
set ebeam1 6800
set ebeam2 6800
set no_parton_cut
0
~~~
{: .output}

> ## What is the cross section?
>
> We added 2 new Feynman diagrams (Z to muon pair and tau pair). How should the cross section add up from previous value 1493pb?
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

~~~output
import model sm

generate p p > z, z > e+ e-
add process p p > z, z > mu+ mu-
add process p p > z, z > ta+ ta-
~~~
{: .output}


## (2) Gridpack : Z boson to electron pair (producing gridpacks with CMSSW)

As mentioned previously, interactive running of MG5 is useful for developments and quick tests, but ultimately not practical for large-scale production.
To avoid having to use the interactive mode, one can make use the card structure of MG.
A fully automated workflow for running MG and producing gridpacks is maintained in the [genproductions repository](https://github.com/cms-sw/genproductions).
A gridpack is simply an archive file that contains all the executable MG5 code needed to produce LHE events for a given process, which can then be executed easily on many different grid workers (hence the name).
It has the advantage that once it is created, it is a one-button program to generate events, no thinking required.
In this part of the exercise, we will use the same input cards as before to create a gridpack, run it, and compare the results to before. #FIXME

Gridpacks are generated using the gridpack_generation script, which we will run in local mode, i.e. on the machine we are currently logged in to.
Note that scripts are provided to run the gridpacks on other computing infrastructures such as the CERN batch system and CMSConnect, which is useful for more complicated processes.

To create a gridpack, we simply call gridpack_generation.sh and pass the process name and card location to it. #FIXME

As the CMSSW environment that has been already set from above could interfere with the genproductions script you would need to execute below to remove CMSSW environment settings (or open a new terminal).

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

Take a look at the cards in `gridpack/ztoee-0j/` directory.
These two files are minimal inputs to make gridpacks.

~~~bash
less gridpack/ztoee-0j/ztoee-0j_proc_card.dat
less gridpack/ztoee-0j/ztoee-0j_run_card.dat 
~~~
{: .source}

You would notice that the `proc_card.dat` defines the physics process that we want to calculate is the same as the first example with standalone tutorial.
From the `run_card.dat` you would notice that PDF choice block is different like below.

~~~output
#*********************************************************************
# PDF CHOICE: this automatically fixes also alpha_s and its evol.    *
#*********************************************************************
  'lhapdf'    = pdlabel     ! PDF set                                  
$DEFAULT_PDF_SETS = lhaid
$DEFAULT_PDF_MEMBERS = reweight_PDF     ! if pdlabel=lhapdf, this is the lhapdf number
~~~
{: .output}

`$DEFAULT_PDF_SETS` and `DEFAULT_PDF_MEMBERS` are parsed later automatically through `Utilities/gridpack_helpers.sh`.
This is to keep consistent PDF setup among different CMS samples.

As there are many people running the tutorial at once, let's restrict the core usage to 2 and then start the gridpack production.

~~~bash
export NB_CORE=2
./gridpack_generation.sh ztoee-0j gridpack/ztoee-0j/ pdmv
~~~
{: .source}

Note `pdmv` is only there to restrict the number of cores to use to two set with `NB_CORE`, normally you can just execute it with `./gridpack_generation.sh <process name> <path to card>` without `pdmv`.

Keep in mind that everything is exactly the same as the standalone tutorial except that `gridpack_generation.sh` is merely replacing every interactive commands that we were giving to MadGraph prompt shell.

You can see that MadGraph is downloaded from the web,

~~~output
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

~~~output
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

~~~output
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
generate p p > z, z > e+ e-
INFO: Checking for minimal orders which gives processes. 
INFO: Please specify coupling orders to bypass this step. 
INFO: Trying process: g g > z WEIGHTED<=2 @1  
~~~
{: .output}

and finally giving out the computed cross sections.

~~~output
  === Results Summary for run: pilotrun tag: tag_1 ===

     Cross-section :   1730 +- 5.893 pb
     Nb of events :  0
~~~
{: .output}

> ## Why did the cross section change?
>
> You would notice that the cross section has changed from 1493pb to 1730pb. What would be the reasons although we ran on exact same physics process?
>
> > ## Solution
> >
> > Most importantly, different PDF set has been used (check by looking at `ztoee-0j/ztoee-0j_gridpack/work/gridpack/process/madevent/Cards/run_card.dat`). This will give you totally different assumptions on parton fractions in a proton. Other minor reasons for the difference could be use of different MadGraph version or different random seed.
> > 
> {: .solution}
{: .challenge}

Now try the same with different gridpack cards.

~~~bash
./gridpack_generation.sh ztoee-0j-5FS gridpack/ztoee-0j-5FS/ pdmv
~~~
{:. shell}

This will give you a new contribution to the process that is `b b~ > z, z > e+ e-` to the calculation as it uses different UFO model that is `sm-no_b_mass`. Strictly speaking, we are using the same UFO model but adding a restriction to the model. Take a look at [restriction_card_tutorial](https://indico.cern.ch/event/239005/attachments/402547/559634/13_02_16_tutomg.pdf) from slide 24 for more information.

~~~output
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
generate p p > z, z > e+ e-

INFO: Trying process: b b~ > z WEIGHTED<=2 @1  
INFO: Process has 1 diagrams 
INFO: Process b~ b > z added to mirror process b b~ > z 
~~~
{:. output}

Now you would get the following, somewhat increased cross section 1795pb compared to the previous example 1740pb.

~~~output
  === Results Summary for run: pilotrun tag: tag_1 ===

     Cross-section :   1795 +- 6.29 pb
     Nb of events :  0
~~~
{:. output}

> ## Why did the cross section change?
>
> Did your `proc_card.dat` add any new processes?
>
> > ## Solution
> >
> > Yes, we now have 5 flavor quarks in the proton which was 4 in the previous example by adding up the bottom quark contributions. So we have new contribution that is `b b~ > z, z > e+ e-`.
> > 
> {: .solution}
>
> ## But why did it not scale up so much?
>
> When we tried out `p p > z, z > e+ e-` and `p p > z, z > l+ l-`, the cross sections was roughly 3 times larger. When we consider 4 (udcs) and 5 (udcsb) flavor schemes of proton should it not be 5/4 times larger?
>
> > ## Solution
> > 
> >  No, keep in mind that proton consists of two up quarks and one down antiquark which are valence quarks. The rest are sea quark contributions, smaller in PDF. Hence, the amount of increase coming from bottom quark contributions are not so large is what we can infer from this result.
> >
> {: .solution}
{: .challenge}

Let's recall why gridpack was useful.
It is a precompiled library to make the LHE files faster for the given process.
CMS uses gridpacks to produce official samples as it is much easier to keep consistency of the sample.
Now it's time to find out how we make LHE files from gridpacks.
To start with, copy and paste the gridpack to a new temporary directory and untar it.

~~~bash
mkdir test
cp ../ztoee-012j_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz ./
tar -xvf ztoee-012j_slc7_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz
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

The first step of tutorial has finished.
Before moving on to the next step, let's run below as it takes some time to finish.

~~~bash
cd -
./gridpack_generation.sh ztoee-012j gridpack/ztoee-012j/ pdmv
~~~
{:. shell}

{% include links.md %}

