---
title: "3 - Optional (MadSpin and BSM UFO model)"
teaching: 10
exercises: 20
questions:
- "What is MadSpin used for?"
- "How do we customize the BSM parameters in the UFO model?"
objectives:
- "Understand the role of MadSpin."
- "Customize BSM parameters for gridpacks."
keypoints:
- "PDF uncertainties can be estimated from a set of different per-event weights. The method depends on the type of PDF set that is used (hessian, MC replicas)"
- "Scale and alphaS variations are another source of uncertainty in the prediction of a simulated sample and can be used to estimate systematic uncertainties"
- "We differentiate between total uncertainties and acceptance / shape uncertainties"
---

# Decay of resonant particles in MadGraph

Until now, we ran the physics process defined as 
~~~
generate p p > e+ e-
~~~
{: .output}

What we can also do is below.
~~~
generate p p > z, z > e+ e-
~~~
{: .output}

The difference between `p p > e+ e-` and `p p > z, z > e+ e-` is that the former is the full DY physics process whereas the latter forces the two quarks to produce a Z boson initially and then lets it decay into electrons.
Splitting the two processes with `,` is the key which tells where the matrix element calculation should be split.
`p p > e+ e-` calculates `2 -> 2` and `p p > z, z > e+ e-` is `2 -> 1` and `1 -> 2`.

~~~
cd $GENMGPATH
./bin/mg5 standalone/onshellz.config
~~~
{: .output}

Compare events in the new LHE file `standalone-onshellz/Events/run_01/unweighted_events.lhe.gz` with the old `standalone-drellyan-mll4//Events/run_01/unweighted_events.lhe.gz`.
For every event in the new LHE file, a particle with `23` shows up as onshell Z boson is always produced in matrix element level calculations.
In contrast, the old LHE file which allows offshell Z (along with gamma), does not necessarily require the existence of such particles in the LHE file.
One event for example is below, `u = 2` and `ubar = -2` quarks produce `z = 23` and decays into `positron = -11` and `electron = 11`.

~~~
<event>
 5      1 +1.4201000e+03 9.06792500e+01 7.54677100e-03 1.30121500e-01
        2 -1    0    0  501    0 +0.0000000000e+00 +0.0000000000e+00 +1.4078650070e+02 1.4078650070e+02 0.0000000000e+00 0.0000e+00 1.0000e+00
       -2 -1    0    0    0  501 -0.0000000000e+00 -0.0000000000e+00 -1.4601410033e+01 1.4601410033e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
       23  2    1    2    0    0 +0.0000000000e+00 +0.0000000000e+00 +1.2618509067e+02 1.5538791074e+02 9.0679246224e+01 0.0000e+00 0.0000e+00
      -11  1    3    3    0    0 -2.5416006610e+01 +3.5010752474e+01 +8.6334112633e+01 9.6567619754e+01 0.0000000000e+00 0.0000e+00 1.0000e+00
       11  1    3    3    0    0 +2.5416006610e+01 -3.5010752474e+01 +3.9850978037e+01 5.8820290983e+01 0.0000000000e+00 0.0000e+00 -1.0000e+00
</event>
~~~
{ .output}

By making the histograms, situation becomes more clearly understandable.

~~~
cd $GENPLOTPATH
python3 lhe-root-plotter-onshellz.py
~~~
{: .output}

From the histograms, you can clearly see that there is no event generated outside the Z boson mass window.

## Using MadSpin

Now we will learn some advanced use cases of MadGraph which is using the MadSpin plugin.
MadSpin, as we saw earlier, is one of the modules that runs through MadGraph interface which handles the decay of resonant particles.
We just took a look at a physics process `p p > z, z > e+ e-`.
Here, `z` is the resonant particle so when we chooses to use MadSpin, above process can be split into two where we first do
~~~
generate p p > z
~~~
{: .output}
and then decay `z` into the electron pair using MadSpin.

But the question still remains, why is MadSpin in any case useful?
The answer lies in NLO calculations in QCD or loop-induced processes.
Let's launch MadGraph prompt shell again.
~~~bash
cd ${GENTUTPATH}/standalone-tut/MG5_aMC_v3_5_2/
./bin/mg5_aMC
~~~
{: .source}

Now try making another simple example that is top pair production.
~~~
import model sm
generate p p > t t~ [QCD]
~~~
{: .output}
It would be not so difficult to realize `[QCD]` has been added in the process definition.
This is a flag which tells MadGraph that you wish to do the calculations at NLO in QCD.

Before going further, try concatenating top decays into a W boson and a b quark similar to what we did for `Z -> ee` example.
~~~
generate p p > t t~, t > w+ b [QCD]
generate p p > t t~ [QCD], t > w+ b
exit
~~~
{: .output}

You will find neither of these working and instead MadGraph complains with an error log saying `str : Decay processes cannot be perturbed`.
So it means that physics processes with decays of particles are are not possible for NLO calculations.
This is where MadSpin becomes necessary, for such cases where resonant particle cannot be decayed can be decayed using MadSpin.
Now lets get back to the working example to see how it works.

~~~
import model sm
generate p p > t t~ [QCD]
output TopPair
launch
shower = PYTHIA8
4
0
~~~
{: .output}

Two lines are noticably added, `shower = PYTHIA8` and `4` (which can be replaced with `madspin = ON`).
We are again not going to do the parton shower here.
This is because depending on which parton shower generator one chooses later, "counter term" calculation differs which accounts as negatively weighted events.

> # Negative weighted events
> We won't cover what it is in the tutorial but important things to remember are that
> 1. Some portion of the events are negatively weighted so one needs to be careful with the normalization.
> 2. LHE files at NLO are even more unphysical than LHE files at LO before parton shower.

Press `tab` to turn off timer.
MadGraph again asks if you would like to edit the cards now including `madspin_card.dat`.
~~~
/------------------------------------------------------------\
|  1. param   : param_card.dat                               |
|  2. run     : run_card.dat                                 |
|  3. madspin : madspin_card.dat                             |
\------------------------------------------------------------/
~~~
{: .output}

If you take a look at the `run_card.dat`, you might notice that the template for it is quite different from when we did DY at LO.
Template for NLO is shown in (link)[https://github.com/cms-PdmV/GridpackFiles/blob/master/Campaigns/Run3Summer22/MadGraph5_aMCatNLO/Templates/NLO_run_card.dat] and for LO is shown in (link)[https://github.com/cms-PdmV/GridpackFiles/blob/master/Campaigns/Run3Summer22/MadGraph5_aMCatNLO/Templates/LO_run_card.dat].
Although MadGraph shares the same user interface, LO and NLO calculations run on totally different codes in the backend.
So NLO type `run_card.dat` does not work for LO calculations and vice versa.

Now take a look at `madspin_card.dat` by pressing `3`.

~~~
# specify the decay for the final state particles
decay t > w+ b, w+ > all all
decay t~ > w- b~, w- > all all
decay w+ > all all
decay w- > all all
decay z > all all
~~~
{: .output}

This card lets you define how you want your resonant particles to decay.
For example, if you do :
~~~
decay t > w+ b, w+ > e+ ve
decay t~ > w- b~, w- > mu- vm~
~~~
{: .output}

This forces top to decay into positron and antitop to decay into muon.
Remove unnecessary decay definitions and add these two lines to make a top pair sample that ends up giving you positron and a muon.
Before moving on, do `set run_card nevents 50` to save time, producing only 50 events.

You will see inclusive top pair production cross section being computed which includes all possible decays for the top quark.
~~~
   --------------------------------------------------------------
      Summary:
      Process p p > t t~ [QCD]
      Run at p-p collider (6500.0 + 6500.0 GeV)
      Number of events generated: 50
      Total cross section: 6.847e+02 +- 4.3e+00 pb
   --------------------------------------------------------------
~~~
{: .output}

And then you will see MadSpin doing its job, decaying the top quarks to desired channels.
~~~
************************************************************
*                                                          *
*           W E L C O M E  to  M A D S P I N               *
*                                                          *
************************************************************

...

INFO: decay channels for t : ( width = 1.4915 GeV ) 
INFO:        BR                 d1  d2 
INFO:    1.000000e+00            b  w+  
INFO:    
INFO:    
INFO: decay channels for w+ : ( width = 2.04793 GeV ) 
INFO:        BR                 d1  d2 
INFO:    3.333610e-01            d~  u  
INFO:    3.333610e-01            s~  c  
INFO:    1.111195e-01            e+  ve  
INFO:    1.111195e-01            mu+  vm  
INFO:    1.110390e-01            ta+  vt  
INFO:    
INFO:    
INFO: decay channels for t~ : ( width = 1.4915 GeV ) 
INFO:        BR                 d1  d2 
INFO:    1.000000e+00            b~  w-  
INFO:    
INFO:    
INFO: decay channels for w- : ( width = 2.04793 GeV ) 
INFO:        BR                 d1  d2 
INFO:    3.333610e-01            d  u~  
INFO:    3.333610e-01            s  c~  
INFO:    1.111195e-01            e-  ve~  
INFO:    1.111195e-01            mu-  vm~  
INFO:    1.110390e-01            ta-  vt~

...

INFO:    Estimating the maximum weight     
INFO:    *****************************     
INFO:      Probing the first 75 events 
INFO:      with 400 phase space points 
INFO:    
INFO: Event 1/75 :  0.068s   
INFO: Event 6/75 :  0.63s   
INFO: Event 11/75 :  1.2s   
INFO: Event 16/75 :  1.8s   
INFO: Event 21/75 :  2.1s   
INFO: Event 26/75 :  3s   
INFO: Event 31/75 :  3.8s   
INFO: Event 36/75 :  4.6s   
INFO: Event 41/75 :  5.7s   
INFO: Event 46/75 :  6.5s   
~~~
{: .output}

> ## What is the cross section?
>
> Inclusive cross section was reported to be 684.7pb as we saw above.
> When considering the decay channels (`e+` and `mu-` final states), what is the proper cross section?
> What are the branching ratios for `w+ > e+ ve` and `w- > mu- vm~`?
>
> > ## Solution
> >
> > ~~~
> > 8.5pb (from 684.7 x 11% x 11%)
> > ~~~
> > {: .solution}
> ## How can we make a sample that yields `mu+`, `vm`, and this time, two quark jets (hadronically decaying `w-`)
>
> > ## Solution
> > ~~~
> > decay t > w+ b, w+ > mu+ vm
> > decay t~ > w- b, w- > j j
> > ~~~
> > {: .output}
> {: .solution}
> 
{: .challenge}

