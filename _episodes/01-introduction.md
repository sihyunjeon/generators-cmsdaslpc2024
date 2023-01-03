---
title: "Introduction"
teaching: 0
exercises: 0
questions:
- "What are Monte Carlo Generators?"
- "Why are we using simulated samples in CMS?"
- "How are simulated samples created in CMS?"
objectives:
- "Use the MadGraph generator in standalone mode"
- "Analyse LHE files"
keypoints:
- "MadGraph is a widely used tool to generate matrix-element predictions for the hard scatter for SM and BSM processes."
---

# What's the fuzz about Monte Carlo?

Samples of simulated events are essential in high energy physics
Processes at vastly different energy regimes are involved, from hard scattering to parton showering
Luckily, this factorizes!
Although the hard and soft process are distinct, they are connected by an evolutionary Markov process that leads to parton showering.
The partons produced in this process eventually participate in the hadron formation (hadronization) where color singlet states are formed.
Monte Carlo techniques can be used for simulating the Markov process, efficient integration of the high dimensional hard scatter problem, and the hadronization models.


## Using Madgraph to simulate the hard scatter process

In this short exercise we will use the interactive prompt of Madgraph to generate proton proton collision events that produce W bosons.
Start the interactive prompt of Madgraph:
~~~bash
cd $CDGPATH/MG5_aMC_v2_6_5/
./bin/mg5_aMC
~~~
{: .source}

Madgraph is configured and steered through text-based cards.
The process definitions can be stored in a card called `proc_card.dat`.
You can look at an example using the following command:
~~~bash
!cat $CDGPATH/gen-cmsdas-2022/cards/wplustest_4f_LO/wplustest_4f_LO_proc_card.dat
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

FIXME: briefly explain commands.

The practically most relevant part is that MG figures out all relevant Feynman diagrams contributing to a process.
If you are trying to set up a new MC sample, looking at these Feynman diagrams is a great way to check that you actually get the physics you want.
To check them out you can open the individual plots in e.g. `wplustest_4f_LO/SubProcesses/P0_qq_wp_wp_lvl/matrix*.ps` with `gv`, `display` or `evince`.
You can also use the `ps2pdf` program to convert the post script files into PDFs.

Now that MG has figured out the feynman diagrams you can start the actual computation with
~~~bash
launch
~~~
{: .source}

Hint: if you closed the interactive MG session for some reason you can still launch without rerunning the previous commands with
~~~bash
launch wplustest_4f_LO
~~~
{: .source}
MG will ask you a few more questions.
Once asked about the `run_card`, one can either use a default run card by just pressing ENTER and edit the default `run_card` by hand or provide a path to a run card of one's choice.
Please provide the path to the pre-made run_card: `${CDGPATH}/gen-cmsdas-2022/cards/wplustest_4f_LO/wplustest_4f_LO_run_card.dat`

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

LHE output

> ## Looking at the output
>
> > ## Solution
> > 
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
> > <!--
> > #*********************************************************************
> > #                                                                    *
> > #                        MadGraph5_aMC@NLO                           *
> > #                                                                    *
> > #                           Going Beyond                             *
> > #                                                                    *
> > #                   http://madgraph.hep.uiuc.edu                     *
> > #                   http://madgraph.phys.ucl.ac.be                   *
> > #                   http://amcatnlo.cern.ch                          *
> > #                                                                    *
> > #                     The MadGraph5_aMC@NLO team                     *
> > #                                                                    *
> > #....................................................................*
> > #                                                                    *
> > # This file contains all the information necessary to reproduce      *
> > # the events generated:                                              *
> > #                                                                    *
> > # 1. software version                                                *
> > # 2. proc_card          : code generation info including model       *
> > # 3. param_card         : model primary parameters in the LH format  *
> > # 4. run_card           : running parameters (collider and cuts)     *
> > # 5. pythia_card        : present only if pythia has been run        *
> > # 6. pgs_card           : present only if pgs has been run           *
> > # 7. delphes_cards      : present only if delphes has been run       *
> > #                                                                    *
> > #                                                                    *
> > #*********************************************************************
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Using the gridpack workflow

As mentioned previously, interactive running of MG5 is useful for exercises and smaller tests, but ultimately not practical for large-scale production.
To avoid having to use the interactive mode, one can use gridpacks instead. A gridpack is simply an archive file that contains all the executable MG5 code needed to produce LHE events for a given process.
It has the advantage that once it is created, it is a one-button program to generate events, no thinking required.
In this part of the exercise, we will use the same input cards as before to create a gridpack, run it, and compare the results to before.



{% include links.md %}

