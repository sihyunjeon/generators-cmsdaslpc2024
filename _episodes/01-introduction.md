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


## First part : Z boson to electron pair

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
> >       -11  1    3    3    0    0 -2.3393803385e+01 -7.4187481776e+00 -1.5274153214e+02 1.5470062541e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
> > ~~~
> > {: .output}
> > This line tells you that a positron (`ID`) is an outgoing particle (`status`) with Z as its mother (`mother1` and `mother2` : 3rd particle is Z which is `ID=23`) with no color (`color` and `anticolor`), ...
> {: .solution}
{: .challenge}


## Second part : Z boson to dilepton pair (using particle containers)

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



{% include links.md %}

