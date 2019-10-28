---
layout: page
title: "DISVIS and HADDOCK2.4 modelling of the apo RNA-Polymerase-III complex from MS cross-linking data"
excerpt: "A tutorial demonstrating the use of MS crosslinks data to build a complex molecular machine."
tags: [MS, Cross-links, Interaction, HADDOCK, RNA Polymerase, Pymol, Visualisation]
image:
  feature: pages/banner_education-thin.jpg
---
This tutorial consists of the following sections:

* table of contents
{:toc}



<hr>
## Introduction

This tutorial will demonstrate the use of DISVIS and HADDOCK for predicting the structure of a large biomolecular assembly from MS cross-linking data.
The case we will be investigating is the apo form of the RNA Polymerase-III (Pol-III). Pol III is a 17-subunit enzyme that transcribes tRNA genes. Its architecture can be subdivided into a core, stalk, heterodimer, and heterotrimer of C82, C34, and C31 subunits. D
<ul>
<figure align="center">
  <img src="/education/HADDOCK24/RNA-Pol-III/Pol-III-architecture.png">
</figure>
<b>Figure 1:</b> Pol III subunits are shown as rectangular bars except for C160 and C128, which are shown as ovals for the sake of clarity. Inter-links are shown as lines connecting the protein bars, while intra-links are shown as curves. Inter-links to C31 are colored yellow, to C34 - gold, to C37 – violet, to C53 - cyan. The remaining inter-links are colored gray. Domains of C82 and C34 discussed in this work are indicated. Regions missing in crystal structures or homology models are colored black. The figure was created with xiNET. Figure reproduced from ​https://www.nature.com/articles/nmeth.3838
<br>
<br>
</ul>

During this tutorial, we pretend that the structure of the Pol 3 core (14 subunits) is known and will focus on modeling the positioning of the C82/C34/C31 heterotrimer subunits relatively to the others (which we will treat as the core of Pol-III). The structure of Pol III is quite well characterized, with multiple cryo-EM structures of Pol III published.

We will thus be making use our [DISVIS server](https://bianca.science.uu.nl/disvis/) to analyse the cross-links and detect possible false positive and of the new [HADDOCK2.4 webserver](https://bianca.science.uu.nl/haddock24) to setup docking runs, using the new coarse-graining option to speed up the calculations (especially needed here considering the size of the system).

A description of our the previous major version of our web server [HADDOCK2.2](https://haddock.science.uu.nl/services/HADDOCK2.2/) can be found in the following publications:

* G.C.P van Zundert, J.P.G.L.M. Rodrigues, M. Trellet, C. Schmitz, P.L. Kastritis, E. Karaca, A.S.J. Melquiond, M. van Dijk, S.J. de Vries and  A.M.J.J. Bonvin.
[The HADDOCK2.2 webserver: User-friendly integrative modeling of biomolecular complexes](https://doi.org/doi:10.1016/j.jmb.2015.09.014).
_J. Mol. Biol._, *428*, 720-725 (2015).

* S.J. de Vries, M. van Dijk and A.M.J.J. Bonvin.
[The HADDOCK web server for data-driven biomolecular docking.](http://www.nature.com/nprot/journal/v5/n5/abs/nprot.2010.32.html)
_Nature Protocols_, *5*, 883-897 (2010).  Download the final author version <a href="http://igitur-archive.library.uu.nl/chem/2011-0314-200252/UUindex.html">here</a>.

Throughout the tutorial, colored text will be used to refer to questions or
instructions, and/or PyMOL commands.

<a class="prompt prompt-question">This is a question prompt: try answering it!</a>
<a class="prompt prompt-info">This an instruction prompt: follow it!</a>
<a class="prompt prompt-pymol">This is a PyMOL prompt: write this in the PyMOL command line prompt!</a>
<a class="prompt prompt-cmd">This is a Linux prompt: insert the commands in the terminal!</a>


<hr>
## Setup/Requirements


In order to follow this tutorial you only need a **web browser**, a **text editor**, and [**PyMOL**][link-pymol]{:target="_blank"}
(freely available for most operating systems) on your computer in order to visualize the input and output data. 
We will however also be describing the use of our PBD-tools to pre-process PDB files for HADDOCK. This part will require access to a Linux command shell.
It is however to required to complete the tutorial as pre-processed, ready to dock models are provided as part of the material for this tutorial. 
The required data to run this tutorial should be downloaded from [**here**][link-data]{:target="_blank"}.
Once downloaded, make sure to unpack/unzip the archive.

Also, if not provided with special workshop credentials to use the HADDOCK portal, make sure to register in order to be able to submit jobs. Use for this the following registration page: [https://nestor.science.uu.nl/auth/register/haddock](https://nestor.science.uu.nl/auth/register/haddock).

<hr><hr>
## HADDOCK general concepts

HADDOCK (see [http://www.bonvinlab.org/software/haddock2.2](http://www.bonvinlab.org/software/haddock2.2)) is a collection of python scripts derived from ARIA ([http://aria.pasteur.fr](http://aria.pasteur.fr)) that harness the
power of CNS (Crystallography and NMR System, [http://cns-online.org](http://cns-online.org)) for structure
calculation of molecular complexes. What distinguishes HADDOCK from other docking software is its ability, inherited
from CNS, to incorporate experimental data as restraints and use these to guide the docking process alongside
traditional energetics and shape complementarity. Moreover, the intimate coupling with CNS endows HADDOCK with the
ability to actually produce models of sufficient quality to be archived in the Protein Data Bank.

A central aspect to HADDOCK is the definition of Ambiguous Interaction Restraints or AIRs. These allow the translation
of raw data such as NMR chemical shift perturbation or mutagenesis experiments into distance restraints that are
incorporated in the energy function used in the calculations. AIRs are defined through a list of residues that fall
under two categories: active and passive. Generally, active residues are those of central importance for the
interaction, such as residues whose knockouts abolish the interaction or those where the chemical shift perturbation is
higher. Throughout the simulation, these active residues are restrained to be part of the interface, if possible,
otherwise incurring in a scoring penalty. Passive residues are those that contribute for the interaction, but are
deemed of less importance. If such a residue does not belong in the interface there is no scoring penalty. Hence, a
careful selection of which residues are active and which are passive is critical for the success of the docking.

The docking protocol of HADDOCK was designed so that the molecules experience varying degrees of flexibility and
different chemical environments, and it can be divided in three different stages, each with a defined goal and
characteristics:

* **1. Randomization of orientations and rigid-body minimization (it0)** <BR>
In this initial stage, the interacting partners are treated as rigid bodies, meaning that all geometrical parameters
such as bonds lengths, bond angles, and dihedral angles are frozen. The partners are separated in space and rotated
randomly about their centers of mass. This is followed by a rigid body energy minimization step, where the partners are
allowed to rotate and translate to optimize the interaction.
The role of AIRs in this stage is of particular importance. Since they are included in the energy function being
minimized, the resulting complexes will be biased towards them. For example, defining a very strict set of AIRs leads
to a very narrow sampling of the conformational space, meaning that the generated poses will be very similar.
Conversely, very sparse restraints (e.g. the entire surface of a partner) will result in very different solutions,
displaying greater variability in the region of binding.

* **2. Semi-flexible simulated annealing in torsion angle space (it1)** <BR>
The second stage of the docking protocol introduces flexibility to the interacting partners through a three-step
molecular dynamics-based refinement in order to optimize interface packing. It is worth noting that flexibility in
torsion angle space means that bond lengths and angles are still frozen. The interacting partners are first kept rigid
and only their orientations are optimized. Flexibility is then introduced in the interface, which is automatically
defined based on an analysis of intermolecular contacts within a 5A cut-off. This allows different binding poses coming
from it0 to have different flexible regions defined. Residues belonging to this interface region are then allowed to
move their side-chains in a second refinement step. Finally, both backbone and side-chains of the flexible interface
are granted freedom.
The AIRs again play an important role at this stage since they might drive conformational changes.

* **3. Refinement in Cartesian space with explicit solvent (water)** <BR>
The final stage of the docking protocol allows to immerse the complex in a solvent shell so as to improve the energetics of the
interaction. HADDOCK currently supports water (TIP3P model) and DMSO environments. The latter can be used as a membrane
mimic. In this short explicit solvent refinement the models are subjected to a short molecular dynamics simulation at
300K, with position restraints on the non-interface heavy atoms. These restraints are later relaxed to allow all side
chains to be optimized. In the 2.4 version of HADDOCK, the explicit solvent refinement is replaced by default by a simple 
energy minimisation as benchmarking has shown it does not add much to the quality of the models. This allows to save time.

The performance of this protocol of course depends on the number of models generated at each step. Few models are less
probable to capture the correct binding pose, while an exaggerated number will become computationally unreasonable. The
standard HADDOCK protocol generates 1000 models in the rigid body minimization stage, and then refines the best 200 (
regarding the energy function) in both it1 and water. Note, however, that while 1000 models are generated by default
in it0, they are the result of five minimization trials and for each of these the 180 degrees symmetrical solution is also
sampled. Effectively, the 1000 models written to disk are thus the results of the sampling of 10.000 docking solutions.

The final models are automatically clustered based on a specific similarity measure - either the *positional interface
ligand RMSD* (iL-RMSD) that captures conformational changes about the interface by fitting on the interface of the
receptor (the first molecule) and calculating the RMSDs on the interface of the smaller partner, or the *fraction of
common contacts* (current default) that measures the similarity of the intermolecular contacts. For RMSD clustering,
the interface used in the calculation is automatically defined based on an analysis of all contacts made in all models.

The new 2.4 version of HADDOCK also allows now to coarse grain the system, which effectively reduces the number of 
particles and speeds up the computations. We are using for this the [MARTINI2.2 force field](https://doi.org/10.1021/ct300646g), 
which is based on a four-to-one mapping of atoms on coarse-grained beads.

<hr><hr>
## The information at hand

Let us first inspect the available data, namely the various structures (or homology models) as well as
the information from MS we have at hand to guide the docking. After unpacking the archive provided for this tutorial (see [Setup](#setuprequirements) above),
you should see a directory called RNA-Pol-III with the following subdirectories in it:

  * __disvis__: This directory contains text files called `xlinks-all-X-Y.txt` describing the cross-links between the various domains (X and Y).
These files are in the format required to run DISVIS.

  * __docking__: This directory contains json files containing all the parameters and input data for HADDOCK. These are reference files of the docking setup and allow to repeat the modelling using the `file-submit` option of the HADDOCK2.4 web server.
  
  * __input-pdbs__: This directory contains the HADDOCK-ready input PDB files for the various domains
    * `A_5fja-core.pdb`: The core region of Pol-III with non-overlapping residue numbering (named as chain A)
    * `B_C82-2XUBA.pdb`: The C82 structure, homology modelled on PDB entry 2XUB excluding the disordered long loops (named as chain B)
    * `C_C34_wHTH1-2DK8A.pdb`: The first helix-turn-helix domain of C34, homology modelled on PDB entry 2DK8-chainA (named as chain C)
    * `D_C34_wHTH2-2DK8A.pdb`: The second helix-turn-helix domain of C34, homology modelled on PDB entry 2DK8-chainA (named as chain D)
    * `E_C34_wHTH3-1KDDA.pdb`: The third helix-turn-helix domain of C34, homology modelled on PDB entry 1KDD-chainA (named as chain E)
    * `D_C31_model.pdb`: A de novo model from I-TASSER, conformation very uncertain (named as chain F)
 	
  * __restraints__:
    * `xlink-all-inter.tbl`: This file contains all cross-links between the various domains (using the full C31 model)
    * `xlink-all-inter-disvis-filtered.tbl`: This file contains the disvis-filtered cross-links between the various domains (using the full C31 model)

From MS, we have experimentally determined cross-links between the various domains. We have only kept  here  the inter-domain cross-links relevant for  this tutorial.
The cross-links are taken from ([Kerber et al. 2016](https://www.nature.com/articles/nmeth.3838). These are the files present in the `disvis` directory. As an example here
are the cross-links identified between the Pol-III core (chain A here) and C31 (chain F):

<pre style="background-color:#DAE4E7">
A  143 CB F 179 CB 0.0 30.0
A 1458 CB F 111 CB 0.0 30.0
A 1458 CB F  91 CB 0.0 30.0
A  166 CB F 196 CB 0.0 30.0
A  189 CB F 199 CB 0.0 30.0
A 3402 CB F  91 CB 0.0 30.0
A 3514 CB F 111 CB 0.0 30.0
A 4206 CB F  91 CB 0.0 30.0
A 4359 CB F  91 CB 0.0 30.0
A 4361 CB F  91 CB 0.0 30.0
</pre>

This is the format used by DisVis to represent the cross-links. Each cross-link definition consists of eight fields:

* chainID of the 1st molecule
* residue number
* atom name
* chainID of the 2nd molecule
* residue number
* atom name
* lower distance limit
* upper distance limit


<hr><hr>
## Using DISVIS to visualize the interaction space and filter false positive restrsaints


### Introduction to DISVIS

DisVis is a software developed in our lab to visualise and quantify the information content of distance restraints
between macromolecular complexes. It is open-source and available for download from our [Github repository][link-disvis]{:target="_blank"}.
To facilitate its use, we have developed a [web portal][link-disvis-web]{:target="_blank"} for it.

DisVis performs a full and systematic 6 dimensional search of the three translational and rotational degrees of freedom to
determine the number of complexes consistent with the restraints. It outputs information about the inconsistent/violated
restraints and a density map that represents the center-of-mass position of the scanning chain consistent with a given
number of restraints at every position in space.

DisVis requires three input files: atomic structures of the biomolecules to be analysed and a text file containing the list of distance restraints between the two molecules .
This is also the minimal required input for the web server in order to setup a run.  

DisVis and  its webserver are described in:

* G.C.P. van Zundert, M. Trellet, J. Schaarschmidt, Z. Kurkcuoglu, M. David, M. Verlato, A. Rosato and A.M.J.J. Bonvin.
[The DisVis and PowerFit web servers: Explorative and Integrative Modeling of Biomolecular Complexes.](https://doi.org/10.1016/j.jmb.2016.11.032){:target="_blank"}.
_J. Mol. Biol._. *429(3)*, 399-407 (2016).

* G.C.P van Zundert and A.M.J.J. Bonvin.
[DisVis: Quantifying and visualizing accessible interaction space of distance-restrained biomolecular complexes](https://doi.org/doi:10.1093/bioinformatics/btv333){:target="_blank"}.
  _Bioinformatics_ *31*, 3222-3224 (2015).

<hr>
### Analysis the Pol-III domain-domain interactions with DISVIS

Before modelling Pol-III based on the available cross-links we will first run DisVis on those for the various pairs of domains in order to both 
assess the information content of the cross-links and detect possible false positive. For the latter, please note that DisVis does not account for
any conformational changes. As such a cross-link flagged as possible false positive might also simply reflect a conformational change in the structures.

We have at hand cross-links available for 8 pairs of domains (see the `disvis` directory from the downloaded data). As an illustration of running DisVis we will here
setup the analysis for the Pol-III core (chain A) - C31 (chain F) pair.

To run DisVis, go to

<a class="prompt prompt-info" href="https://bianca.science.uu.nl/disvis" target="_blank">https://bianca.science.uu.nl/disvis</a>

On this page, you will find the most relevant information about the server as well as the links to the local and grid versions of the portal's submission page.

#### Step1: Register to the server

[Register][link-disvis-register]{:target="_blank"} for getting access to the web server (or use the credentials provided in case of a workshop).

You can click on the "**Register**" menu from any DisVis page and fill the required information.
Registration is not automatic but is usually processed within 12h, so be patient.


#### Step2: Define the input files and parameters and submit

Click on the "**Submit**" menu to access the [input form][link-disvis-submit]{:target="_blank"}.

<figure align="center">
<img src="/education/HADDOCK24/RNA-Pol-III/disvis-submission.png">
</figure>

From the `input-pdbs` directory select:
<a class="prompt prompt-info">Fixed chain → A_5fja-core.pdb</a>
<a class="prompt prompt-info">Scanning chain → F_C31_model.pdb</a>
From the `disvis` directory select:
<a class="prompt prompt-info">Restraints file → xlinks-all-A-F.txt</a>

Once the fields have been filled in, you can submit your job to our server
by clicking on "**Submit**" at the bottom of the page.

If the input fields have been correctly filled you should be redirected to a status page displaying a message
indicating that your run has been successfully submitted.
While performing the search, the DisVis web server will update you on the progress of the
job by reloading the status page every 30 seconds.
The runtime of this example case is below  5 minutes on our local CPU and grid GPU servers. However the load of the server as well as
pre- and post-processing steps might substantially increase the time until the results are available.

If you want to learn more about the meaning of the various parameters go to:

<a class="prompt prompt-info" href="https://bianca.science.uu.nl/disvis" target="_blank">https://bianca.science.uu.nl/disvis</a>

Then click on the "**Help/Manual**" menu.

The rotational sampling interval option is given in
degrees and defines how tightly the three rotational degrees of freedom will be
sampled. Voxel spacing is the size of the grid's voxels that will be cross-correlated during the 3D translational search.
Lower values of both parameters will cause DisVis to perform a finer search, at the
expense of increased computational time. The default values are `15°` and `2.0Å` for a quick scanning and `9.72°` and `1.0Å`
for a more thorough scanning.
For the sake of time in this tutorial, we will keep the sampling interval to the quick scanning settings (`15.00°` and `2.0Å`).
The number of processors used for the calculation is fixed to 8 processors on the web server side.
This number can of course be changed when using the local version of DisVis.

<hr>
#### Analysing the results

Once your job has completed, and provided you did not close the status page, you will be automatically redirected to the results
page (you will also receive an email notification).

If you don't' want to wait for your run to complete, you can access the precalculated results of a previously run submitted
run [here](https://bianca.science.uu.nl/disvis/run/few_3AOlFUSu){:target="_blank"}.

The results page presents a summary split into several sections:

* `Status`: In this section you will find a link from which you can download the output data as well as some information
about how to cite the use of the portal.
* `Accessible Interaction Space`: Here, images of the fixed chain together with the accessible interaction space, in
a density map representation, are displayed. Different views of the molecular scene can be chosen by clicking
 on the right or left part of the image frame. Each set of images matches a specific level of N restraints which corresponds
 to the accessible interaction space by complexes consistent with at least N restraints. A slider below the image container
 allows you to change the the number of restraints N and load the corresponding set of images.
* `Accessible Complexes`: Summary of the statistics for number of complexes consistent with at least N number of restraints.
 The statistics are displayed for the N levels, N being the total number of restraints provided in the restraints file (here `restraints.txt`)
* `z-Score`: For each restraint provided as input, a z-Score is provided, giving an indication of how likely it is that the restraint is a false positive.
The higher the score, the more likely it is that a restraint might be a false positive. Putative false positive restraints
are only highlighted if no single solution was found to be consistent with the total number of restraints provided. If DisVis
finds complexes consistent with all restraints, the z-Scores are still displayed, but should be ignored.
* `Violations`: The table in this sections shows how often a specific restraint is violated for all models consistent with
a given number of restraints. The higher the violation fraction of a specific restraint, the more likely it is to be a false positive.
Column 1 shows the number of consistent restraints N, while each following column indicates the violation fractions of
a specific restraint for complexes consistent with at least N restraints. Each row thus represents the fraction of all
complexes consistent with at least N restraints that violated a particular restraint. As for the z-Scores, if solutions have been found
that are consistent with all restraints provided, this table should be ignored.

It is possible to extract significant results from the results page of this initial run.

<a class="prompt prompt-question"> Using the different descriptions of the sections we provided above together with the information
on the results page of your run, what are likely false positive restraints according to DisVis?</a>

As mentioned above, the two last sections feature a table that highlights putative false positive restraints based on
their z-Score and their violation frequency for a specific number of restraints. We will naturally look for the
crosslinks with the highest number of violations. The DisVis web server preformats the results in a way that false positive restraints
are highlighted and can be spotted at a glance.

In our case, you should observe that DisVis found solutions consistent with up to 8 restraints indicating that there might be two false positive restraints.
Taking a closer look at the violations table might already be enough to determine which residues are most likely True false positives.
In this example two restraints are violated in all complexes consistent with 8 restrains and are thus the most likely candidates:

<details style="background-color:#DAE4E7"><summary><b>See solution:</b>
</summary>
<center><b>A166(CB)-F196(CB)</b></center>
<center><b>A189(CB)-F199(CB)</b></center>
</details>
<br>
<br>
When DisVis fails to identify complexes consistent with all provided restraints during quick scanning it is advisable to rerun with the complete scanning parameters before remove all restraints (or remove only the most violated ones and rerun with complete scanning). It is possible that a more thourough sampling of the interaction space will yield complexes consistent with all restraints or at least reduce the list of putative false positive restraints.  

<hr>
#### DisVis output files

It is difficult to really appreciate the accessible interaction space between the two partners with only images.
Therefore download the results archive to your computer which is available at the top of your results page.
You will find in it the following files:

* `accessible_complexes.out`: A text file containing the number of complexes consistent with a number of restraints.
* `accessible_interaction_space.mrc`: A density file in MRC format. The density represents the center of mass of the
scanning chain conforming to the maximum number of consistent restraints at every position in space
* `disvis.log`: A log file showing all the parameters used, together with date and time indications.
* `violations.out`: A text file showing how often a specific restraint is violated for each number of consistent restraints.
* `z-score.out`: A text file giving the Z-score for each restraint. The higher the score, the more likely the restraint
is a false positive.
* `run_parameters.json`: A text file containing the parameters of your run.
* `result.html`: A reduced version of the results page for viewing the results offline or after the data have been deleted from our servers.

_Note_: Results for the different pair combinations are available from the tutorial data directory in the `disvis` directory as `disvis-results-X-Y`.

Let us now inspect the solutions and visualise of the interaction space in Chimera:

<a class="prompt prompt-info">
  Open the *fixed_chain.pdb* file and the *accessible_interaction_space.mrc* density map in Chimera.
</a>

<a class="prompt prompt-info">
  UCSF Chimera Menu → File → Open... → Select the file
</a>

Or from the Linux command line:

<a class="prompt prompt-linux">
chimera fixed_chain.pdb accessible_interaction_space.mrc
</a>

The values of the `accessible_interaction_space.mrc` slider bar correspond to the number of satisfied restraints (N).
In this way, you can selectively visualise regions where complexes have been found to be consistent with a given number of
restraints. Try to change the level in the "**Volume Viewer**" to see how the addition of restraints reduces
the accessible interaction space. 

_Note_: The interaction space displayed corresponds to the region of space in which you can place the center of mass 
        of the scanning molecule while satisfying a given number of restraints


<hr>
### Converting DISVIS restraints into HADDOCK restraints

In principle you should repeat the DisVis analysis for all pairs to detect possible false positives. 
We are however providing already files that have been filtered for possible false positive cross-links.
Try to figure out how many cross-links were removed by comparing the following two files in the `disvis` directory:

 * `xlinks-all-inter.txt`
 * `xlinks-all-inter-disvis-filtered.txt`

This can be easily done at the Linux level with:

<a class="prompt prompt-linux">
diff xlinks-all-inter.disvis xlinks-all-inter-disvis-filtered.disvis
</a>

<a class="prompt prompt-question">
  How many restraints have been removed as false positive for the entire system? And between which domains?.
</a>

<details style="background-color:#DAE4E7"><summary><b>See solution:</b>
</summary>
Next to the two cross-links between domains A and F that we already identified from our DisVis analysis, one additional one was removed:
B322(CB)-F179(CB)
</details>
<br>

Before setting up the docking we need first to generate the distance restraint file for the cross-links in a format suitable for HADDOCK.
HADDOCK uses [CNS][link-cns] as computational engine. A description of the format for the various restraint types supported by HADDOCK can
be found in our [Nature Protocol](http://www.nature.com/nprot/journal/v5/n5/abs/nprot.2010.32.html) paper, Box 4.

Distance restraints are defined as:

<pre>
assi (selection1) (selection2) distance, lower-bound correction, upper-bound correction
</pre>

The lower limit for the distance is calculated as: distance minus lower-bound correction
and the upper limit as: distance plus upper-bound correction

The syntax for the selections can combine information about chainID - `segid` keyword -, residue number - `resid`
keyword -, atom name - `name` keyword.
Other keywords can be used in various combinations of OR and AND statements. Please refer for that to the [online CNS manual][link-cns].

Here would be an example of a distance restraint between the CB carbons of residues 10 and 200 in chains A and B with an
allowed distance range between 10 and 20Å:

<pre>
assi (segid A and resid 10 and name CB) (segid B and resid 200 and name CB) 20.0 10.0 0.0
</pre>

<a class="prompt prompt-question">
Can you think of a different way of defining the distance and lower and upper corrections while maintaining the same
allowed range?
</a>


Under Linux (or OSX), this file can be generated automatically from the `xlinks-all-inter-disvis-filtered.txt`
file provided with the data for this tutorial by giving the following command (one line) in a terminal window:

<a class="prompt prompt-linux">
cat xlinks-all-inter-disvis-filtered.txt | awk '{if (NF == 8) {print "assi (segid ",$1," and resid ",$2," and name ",$3,") (segid ",$4," and resid ",$5," and name ",$6,") ",$8,$8,$7}}' > xlinks-all-inter-disvis-filtered.tbl
</a>

The correspondong pre-generated CNS/HADDOCK formatted restraints files are provided in the `haddock` directory as:

  * `xlinks-all-inter.tbl`
  * `xlinks-all-inter-disvis-filtered.tbl`

Inspect the `xlinks-all-inter.tbl` file (open it as a text file)

<a class="prompt prompt-question">
We used CB atoms to define the restraints in the disvis restraint file. Can you find those is this file?
Are there other atoms defined? What could those be? (Hint... MARTINI)
</a>

<details style="background-color:#DAE4E7"><summary><b>See solution:</b>
</summary>
Additional atoms are included in the distance restraints definitions: SC* . These correspond to the side-chain atom names in the MARTINI representation.
</details>
<br>

In the restraints directory provided, there is one additional restraint file provided: `C34-connectivity.tbl`.
Inspect its content.

<a class="prompt prompt-question">
What are those restraints for?
</a>

<details style="background-color:#DAE4E7"><summary><b>See solution:</b>
</summary>
C34 consists of three helix-turn-helix domains which have been modelled separately. They are connected by flexible linkers for which no structure is available.
The defined restraints impose upper limits to the distance between the C- and N-terminal domains of the the domains. The upper limit was estimated as the number of missing segments/residues * 4.5Å (a typical distance observed in diffraction data for amyloid fibrils, representing a CA-CA distance in an extended conformation).
</details>
<br>



<hr><hr>
## Setting up the docking with cross-links

We only have cross-links for two of the three domains of C34. Will will therefore not include C34_wHTH3 into the modelling.
We will therefore be setting up a five-body docking using the PolIII-core, C82, C34_wHTH1, C34_wHTH2 and C31 structures/models.


<hr>
### Setting up the docking with cross-links using the full C31

#### Registration / Login

In order to start the submission, either click on "*here*" next to the submission section, or click [here](https://nestor.science.uu.nl/auth/register/). To start the submission process, we are prompted for our login credentials. After successful validation of our credentials we can proceed to the structure upload.
If running this tutorial in the context of a course/workshop, you will be provided with course credentials.

**Note:** The blue bars on the server can be folded/unfolded by clicking on the arrow on the left

#### Submission and validation of structures

We will make us of the [HADDOCK2.4 interface](https://bianca.science.uu.nl/haddock2.4/submit/1) of the HADDOCK web server.

* **Step 1:** Define a name for your docking run, e.g. *RNA-Pol-III-xlinks*.

* **Step 2:** Define the number of components, i.e. *5*.

* **Step 3:** Input the first protein PDB file. For this unfold the **Molecule 1 input menu**.

<a class="prompt prompt-info">
First molecule: where is the structure provided? -> "I am submitting it"
</a>
<a class="prompt prompt-info">
Which chain to be used? -> All (for this particular case)
</a>
<a class="prompt prompt-info">
PDB structure to submit -> Browse and select *A_PolIII-5fja-core.pdb*
</a>
<a class="prompt prompt-info">
Do you want to coarse-grain your molecule? -> turn on
</a>
<a class="prompt prompt-info">
Segment ID to use during docking -> A
</a>

* **Step 4:** Input the second protein PDB files.

<a class="prompt prompt-info">
PDB structure to submit -> Browse and select *B_C82-2XUBA.pdb*
</a>
Since we do not allow to mix all-atom and coarse grained models, the option to coarse grain this molecule is already turned on.
<a class="prompt prompt-info">
Segment ID to use during docking -> B
</a>

* **Step 5:** Input the third protein PDB files.

<a class="prompt prompt-info">
PDB structure to submit -> Browse and select *C_C34_wHTH1-2DK8A.pdb*
</a>
<a class="prompt prompt-info">
Segment ID to use during docking -> C
</a>

* **Step 6:** Input the fourth protein PDB files.

<a class="prompt prompt-info">
PDB structure to submit -> Browse and select *D_C34_wHTH2-2DK8A.pdb*
</a>
<a class="prompt prompt-info">
Segment ID to use during docking -> D
</a>

* **Step 7:** Input the fifth protein PDB files.

<a class="prompt prompt-info">
PDB structure to submit -> Browse and select *B_C82-2XUBA.pdb*
</a>
<a class="prompt prompt-info">
Segment ID to use during docking -> F
</a>

===> _Make sure to change the Segmend ID to F otherwise the restraints for C31 won't be used!_ <===

* **Step 8:** Click on the "Next" button at the bottom left of the interface. This will upload the structures to the HADDOCK webserver where they will be processed and validated (checked for formatting errors). The server makes use of [Molprobity](http://molprobity.biochem.duke.edu/) to check side-chain conformations, eventually swap them (e.g. for asparagines) and define the protonation state of histidine residues.


#### Definition of restraints

If everything went well, the interface window should have updated itself and it should now show the list of residues for molecules 1 and 2.

* **Step 9:** Instead of specifying active and passive residues, we will supply restraint files to HADDOCK. 
No further action is required in this page, so click on the "Next" button at the bottom of the **Input parameters** window, 
which proceeds to the  **Distance Restraint menu**  menu of the **Docking Parameters** window.

* **Step 10:** Upload the cross-link restraints file to the ambiguous restraints category

<a class="prompt prompt-info">
Instead of specifying active and passive residues, you can supply a HADDOCK restraints TBL file (ambiguous restraints) -> Browse and select *xlinks-all-inter-disvis-filtered.tbl*
</a>

**Note:** Although the name points to ambiguous restraints, any type of distance restraints can be uploaded here. The only thing to remember
          is that by default 50% of the ambiguous restraints will be randomly discarded for each docking model generated. This option can be
          turned off -something we will do below since we have a limited amount of cross-links and already checked them with DisVis.

* **Step 11:** Upload the C34 connectivity restraints file to the unambiguous restraints category

<a class="prompt prompt-info">
You can supply a HADDOCK restraints TBL file with restraints that will always be enforced (unambiguous restraints) -> Browse and select *C34-connectivity.tbl*
</a>

* **Step 12:** Turn off random removal of restraints

<a class="prompt prompt-info">
Randomly exclude a fraction of the ambiguous restraints (AIRs) -> Turn off
</a>

#### Other docking settings and job submission

In  the same page as where restraints are provided you can modify a large number of docking settings.

* **Step 11:** Unfold the **sampling parameters menu.

Here you can change the number of models that will be calculated, the default being 1000/200/200 for the three stages of HADDOCK (see [HADDOCK General Concepts](#haddock-general-concepts). When docking multiple subunits it is recommended to increase the sampling, e.g. to 10000/400/400 (at the cost of longer computations). For this tutorial we will keep the default settings. 
When docking only with  interface information (i.e. no specific distances), we systematically sampling the 180 degrees rotated solutions for each interface, mimimizing the rotated solution and keeping the best of the two in terms of HADDOCK score. Since here we are using rather specific distance restraints, we can turn off this option to save time.

<a class="prompt prompt-info">
Sample 180 degrees rotated solutions during rigid body EM -> turn off
</a>


* **Step 12:** Unfold the **clustering parameters** menu. 

Cluster of multibody assemblies is non trivial. We are using here the default clustering based on the fraction of common contacts between subunits.
To make the clustering more sensitive to the many interfaces we will increase the cutoff to 75%.

<a class="prompt prompt-info">
RMSD Cutoff for clustering (recommended: 7.5A for RMSD, 0.60 for FCC)   (AIRs) -> 0.75
</a>

And we will also reduce the minimum cluster size to 20

<a class="prompt prompt-info">
Minimum cluster size -> 2
</a>

We are now ready to submit the docking run.

The interface also allows us to download the input structures of the docking run (in the form of a tgz archive) and a haddockparameter file which contains all the settings and input structures for our run (in json format). We strongly recommend to download this file as it will allow you to repeat the run after uploading into the [file upload inteface](https://bianca.science.uu.nl/haddock2.4/submit_file) of the HADDOCK webserver. It can serve as input reference for the run. This file can also be edited to change a few parameters for example. An excerpt of this file is shown here:

<pre>
{
    "runname": "RNA-Pol-III-xlinks",
    "amb_cool1": 10.0,
    "amb_cool2": 50.0,
    "amb_cool3": 50.0,
    "amb_firstit": 0,
    "amb_hot": 10.0,
    "amb_lastit": 2,
    "anastruc_1": 200,
...
</pre>

This file contains all parameters and input data of your run, including the uploaded PDB files and the restraints.

<a class="prompt prompt-question">
Can you locate the distance restraints in this file?
</a>

* **Step 13:** Click on the "Submit" button at the bottom left of the interface.

Upon submission you will be presented with a web page which also contains a link to the previously mentioned haddockparameter file as well as some information about the status of the run.

<figure align="center">
<img src="/education/HADDOCK24/RNA-Pol-III/submission.png">
</figure>

Currently your run should be queued but eventually its status will change to "Running" with the page showing the progress of the calculations.
The page will automatically refresh and the results will appear upon completion (which can take between 1/2 hour to
several hours depending on the size of your system and the load of the server). Since we are dealing here with a large complex, 
the docking will take quite some time (probably 1/2 day). So be patient. You will be notified by email once your job has successfully completed.



<hr>
## First analysis of the results

Once your run has completed you will be presented with a result page showing the cluster statistics and some graphical
representation of the data (and if registered, you will also be notified by email). If using course credentials
provided to you, the number of models generated will have been decreased to allow the runs to complete within a
reasonable amount of time. Because of that, the results might not be very good.

We already pre-calculated full docking runs (meaning that the default number of models has been generated: 1000 for
rigid-body docking and 200 for semi-flexible and water refinement). The full run can be accessed at:

[https://wenmr.science.uu.nl/haddock2.4/run/4242424242/PRE5-PUP2-MS-crosslinks](https://wenmr.science.uu.nl/haddock2.4/run/4242424242/PRE5-PUP2-MS-crosslinks)


<figure align="center">
<img src="/education/HADDOCK24/RNA-Pol-III/HADDOCK-result-page.png">
<p> Example result page</p>
</figure>

<a class="prompt prompt-question">Inspect the result page</a>
<a class="prompt prompt-question">How many clusters are generated?</a>


__Note:__ The bottom of the page gives you some graphical representations of the results, showing the distribution of
the solutions for various measures (HADDOCK score, van der Waals energy, ...) as a function of the RMSD from the best
generated model (the best scoring model). The plots are interactive and you can zoom into selected areas. You also turn on and off
specific clusters.

You can also quickly visualize a specific structure by clicking on the "eye" icon next to a structure.


<figure align="center">
<img src="/education/HADDOCK24/RNA-Pol-III/HADDOCK-result-graph.png">
</figure>



The ranking of the clusters is based on the average score of the top 4 members of each cluster. The score is calculated
as:
<pre>
      HADDOCKscore = 1.0 * Evdw + 0.2 * Eelec + 1.0 * Edesol + 0.1 * Eair
</pre>
where Evdw is the intermolecular van der Waals energy, Eelec the intermolecular electrostatic energy, Edesol represents
an empirical desolvation energy term adapted from Fernandez-Recio *et al.* J. Mol. Biol. 2004, and Eair the AIR energy.
The cluster numbering reflects the size of the cluster, with cluster 1 being the most populated cluster. The various
components of the HADDOCK score are also reported for each cluster on the results web page.

<a class="prompt prompt-question">Consider the cluster scores and their standard deviations.</a>
<a class="prompt prompt-question">Is the top ranked significantly better than the second one? (This is also
reflected in the z-score).</a>

In case the scores of various clusters are within the standard deviation from each other, all should be considered as a
valid solution for the docking. Ideally, some additional independent experimental information should be available to
decide on the best solution.

<hr>
## Visualisation of docked models


Let's now visualize the various clusters

<a class="prompt prompt-info">Download and save to disk the first model of each cluster (create a subdirectory for each
scenario to avoid mixing models</a>

We illustrate the procedure using the results from scenario 1. You can repeat it for the other scenarios.
Start PyMOL and load each cluster representative:

<a class="prompt prompt-pymol">File menu -> Open -> select cluster1_1.pdb</a>

Repeat this for each cluster. Once all files have been loaded, type in the PyMOL command window:

<a class="prompt prompt-pymol">
show cartoon<br>
util.cbc<br>
hide lines<br>
</a>

Let's then superimpose all models on chain A of the first cluster:

<a class="prompt prompt-pymol">
select cluster1_1 and chain A<br>
align cluster2_1, sele
</a>

<a class="prompt prompt-info">
Repeat the align command for each cluster representative.
</a>

This will align all clusters on chain A (PRE5), maximizing the differences in the orientation of chain B (PUP2).

<a class="prompt prompt-question">
Examine the various clusters. How does the orientation of PUP2 differ between them?
</a>

__Note:__ You can turn on and off a cluster by clicking on its name in the right panel of the PyMOL window.

Let's now check if the solutions actually fit the cross-links we defined.
Start a new PyMOL session and load as described above the model you want to analyze, e.g. the best model of the top
ranking cluster `cluster1_1.pdb` for scenario 1.
In the PyMOL command window type:

<a class="prompt prompt-pymol">
show cartoon<br>
util.cbc<br>
hide lines<br>
distance d1-23A, chain A and resid 27 and name CA, chain B and resid 18 and name CA<br>
distance d2-23A, chain A and resid 122 and name CA, chain B and resid 125 and name CA<br>
distance d3-23A, chain A and resid 122 and name CA, chain B and resid 128 and name CA<br>
distance d4-23A, chain A and resid 122 and name CA, chain B and resid 127 and name CA<br>
distance d5-26A, chain A and resid 55 and name CA, chain B and resid 169 and name CA<br>
distance d6-26A, chain A and resid 55 and name CA, chain B and resid 179 and name CA<br>
distance d7-26A, chain A and resid 54 and name CA, chain B and resid 179 and name CA<br>
</a>

This will draw lines between the connected atoms and display the corresponding Euclidian distance.
Objects are created in the left panel with their name corresponding to the cross-link and its associated maximum distance.

<details style="background-color:#DAE4E7">
<summary>See solution:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/PRE5-PUP2-crosslinks.png">
</figure>
<br>
</details>

<br>
<a class="prompt prompt-info">
Inspect the various cross-link distances.
</a>

<a class="prompt prompt-question">
Is the model satisfying the cross-link restraints?
</a>

__Note__ that the reported distances are Euclidian distances. In reality, the cross-linker will have to follow the
surface of the molecule which might results in a longer effective distance. A proper comparison would required
calculating the surface distance instead. Such an analysis can be done with the [XWalk sofware][link-xwalk].


We can also visualize the interface residues identified from the DisVis interaction analysis.
For this type in the PyMOL command window:

<a class="prompt prompt-pymol">
color orange, chain A and resid 7+10+13+15+55+58+60+82+83+125+126+127+128+129+131+133<br>
color pink, chain B and resid 1+2+3+5+8+11+13+15+16+17+114+121+122+123+124+140+152+154+177<br>
</a>

For better visualization turn on the surface representation:

<a class="prompt prompt-pymol">
show surface<br>
</a>

<a class="prompt prompt-question">
Are all predicted interface residues from DisVis forming contacts?
</a>

<details style="background-color:#DAE4E7">
<summary>See solution:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/cluster1-crosslinks-surface.png">
</figure>
<br>
While the model is mostly satisfying the defined distance restraints, many residues identified as potential interface
by DisVis are not forming contacts.
</details>

<br>
<a class="prompt prompt-info">
Now repeat this analysis for the top ranking model of scenario 2 and 3.
</a>

<a class="prompt prompt-question">
Is the model satisfying the cross-link restraints?
</a>
<a class="prompt prompt-question">
Are all predicted interface residues from DisVis forming contacts?
</a>

<details style="background-color:#DAE4E7">
<summary>See solution for scenario 2:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/cluster10-interface-surface.png">
</figure>
<br>
In this model some cross-links are severely violated with distances > 40Å. But the putative interface residues are
mostly forming contacts.
</details>
<br>
<details style="background-color:#DAE4E7">
<summary>See solution for scenario 3:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/cluster1-crosslinks-interface-surface.png">
</figure>
<br>
This model nicely satisfies both the cross-links and the predicted interface residues from the DisVis interaction
analysis.
</details>


<hr>
## Comparison with the crystal structure of an homologous complex

In order to check if any of the docking models we obtained using the various scenarios makes sense,
we will compare them to a recently published [article](https://doi.org/10.15252/embj.201695222){:target="_blank"} (November 2016) structure of a homologue
of the 26S proteasome (from *S. cerevisiae*). This structure has been solved by X-ray crystallography at 2.4Å resolution (PDBid
[5L5A](http://www.rcsb.org/pdb/explore/explore.do?structureId=5L5A){:target="_blank"}).
We will only use the two chains that are of interest, namely chains **D** and **C** corresponding to **PRE5** and **PUP2** respectively.
The corresponding PDB file is available in the downloaded tutorial data as `5l5a_CD.pdb`.

For each of the three docking scenarios we will visualize the top ranking model of each cluster
and compare those to the crystal structure of the homologous complex. We will only illustrate this process for scenario 3,
for which 6 clusters were generated.

<a class="prompt prompt-info">
Open PyMOL and load the top ranking model of each cluster
</a>

<a class="prompt prompt-info">
Also load the structure of the homologous complex -> *5l5a_CD.pdb*
</a>

In PyMOL type the following commands (the last two lines are for removing water from the crystal structure):

<a class="prompt prompt-pymol">
show cartoon<br>
util.cbc<br>
hide lines<br>
select 5l5a_CD and resn HOH<br>
remove sele<br>
</a>

Let's now superimpose all models onto the crystal structure, using PRE5 to emphasize the difference in the orientation of PUP2.
For this we need to superimpose chain A of our models onto chain D of the homologous crystal structure.

<a class="prompt prompt-pymol">
select refe, 5l5a_CD and chain D<br>
align cluster1_1 and chain A, refe <br>
align cluster2_1 and chain A, refe <br>
align cluster3_1 and chain A, refe <br>
align cluster4_1 and chain A, refe <br>
align cluster5_1 and chain A, refe <br>
align cluster6_1 and chain A, refe <br>
align cluster7_1 and chain A, refe <br>
align cluster8_1 and chain A, refe <br>
align cluster9_1 and chain A, refe <br>
align cluster11_1 and chain A, refe <br>
zoom vis<br>
</a>

You now see all models superimposed on the the crystal structure of the homologous complex (shown in yellow and magenta).
Turn on and off models (by clicking on the name in the right panel).

<a class="prompt prompt-question">
Can you identify any cluster that resembles the crystal structure?<br>
</a>
<a class="prompt prompt-question">
If this is the case, what is the rank of that cluster in the HADDOCK ranking?<br>
</a>

__Note__ that the cluster number does not indicate its rank.
The clusters are listed in the HADDOCK result page in the order of their ranking.
The cluster number only reflects the size of the cluster, with cluster1 being the most populated.


<details style="background-color:#DAE4E7">
<summary>See solution for scenario 3:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/crosslinks-interface-vs-xray.png">
</figure>
<br>
Cluster1 of the scenario 3 docking run using both cross-links and the interfaces predicted by DisVis nicely matches the crystal structure.
This is also the top-ranking cluster according to HADDOCK.

</details>

<br>
<a class="prompt prompt-info">
Now repeat this analysis, but for the other two scenarios<br>
</a>

<a class="prompt prompt-question">
Does the docking using only the cross-links as distance restraints generate any reasonable models?<br>
</a>
<details style="background-color:#DAE4E7">
<summary>See solution for scenario 1:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/crosslinks-vs-xray.png">
</figure>
<br>
No single cluster matches the crystal structure in this case.
</details>

<br>
<a class="prompt prompt-question">
Does the docking using only the predicted interfaces from the DisVis interaction analysis generate any reasonable models?<br>
</a>
<details style="background-color:#DAE4E7">
<summary>See solution for scenario 2:
</summary>
<figure align="center">
  <img src="/education/HADDOCK24/HADDOCK24-Xlinks/interface-vs-xray.png">
</figure>
<br>
Cluster10 of the scenario 2 docking run using the interfaces predicted by DisVis nicely matches the crystal structure.
Cluster10 in this case is the third-ranking cluster. Its score (-73.0 +/- 6.5) is higher (the lower the score the better) than
the top-ranking cluster (-112.8 +/- 7.0). This is a nice example that scoring remains a challenge. We therefore always recommend
to look at multiple solutions and possibly cross-validate with additional data.

</details>

<br>


<hr>
## Conclusions

We have demonstrated the use of cross-linking data from mass spectrometry for guiding the docking process in HADDOCK.
Because of the rather low precision of the detected cross-links (meaning here their rather large distance range),
using them alone to guide docking in HADDOCK might not be the best scenario.

Additional information on the putative interfaces can be extracted by performing an interaction analysis in our [DisVis web portal][link-disvis].
By performing an exhaustive search of all geometrically feasible complexes consistent with the cross-linking data, DisVis can
extract key interacting residues. This is additional information that might be beneficial to guide the docking as demonstrated here.

We should also note that the modeling was done from models of PRE5 and PUP2, with their limitations.
If better, higher accuracy structures are available the results might well be different. Still, from this analysis, the recommended scenario
for docking using our HADDOCK software would be the combination of distance restraints representing the cross-links with interface information
extracted from the DisVis interaction analysis.


<hr>
## Congratulations!

Thank you for following this tutorial. If you have any questions or suggestions, feel free to contact us via email, or post your question to
our [HADDOCK forum](http://ask.bioexcel.eu/c/haddock){:target="_blank"} hosted by the
[<img width="70" src="/images/Bioexcel_logo.png">](http://bioexcel.eu){:target="_blank"} Center of Excellence for Computational Biomolecular Research.

[link-cns]: http://cns-online.org "CNS online"
[link-data]: http://milou.science.uu.nl/cgi/services/DISVIS/disvis/disvis-tutorial.tgz "DisVis tutorial data"
[link-disvis]: https://github.com/haddocking/disvis "DisVis GitHub repository"
[link-disvis-web]: http://haddock.science.uu.nl/services/DISVIS/ "DisVis web server"
[link-disvis-submit]: https://milou.science.uu.nl/cgi/enmr/services/DISVIS/disvis/submit "DisVis submission"
[link-disvis-register]: https://milou.science.uu.nl/cgi/enmr/services/DISVIS/disvis/register "DisVis registration"
[link-pymol]: http://www.pymol.org/ "PyMOL"
[link-haddock]: http://bonvinlab.org/software/haddock2.2 "HADDOCK 2.2"
[link-haddock-web]: https://wenmr.science.uu.nl/haddock2.4/ "HADDOCK 2.4 webserver"
[link-haddock-easy]: http://haddock.science.uu.nl/services/HADDOCK2.2/haddockserver-easy.html "HADDOCK2.2 webserver easy interface"
[link-haddock-expert]: http://haddock.science.uu.nl/services/HADDOCK2.2/haddockserver-expert.html "HADDOCK2.2 webserver expert interface"
[link-haddock-register]: http://haddock.science.uu.nl/services/HADDOCK2.2/register.html "HADDOCK web server registration"
[link-molprobity]: http://molprobity.biochem.duke.edu "MolProbity"
[link-xwalk]: http://www.xwalk.org