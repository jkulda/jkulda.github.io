---
layout: docs
title: Pair distribution functions
---


The `MP_PDF` tool accumulates the total and partial pair distribution functions (PDF) by direct real-space sampling and binning of atom distances between pairs of unit cells in the simulation box. The present version employs solely Monte-Carlo pseudo-random sampling using the `RANDOM_NUMBER` intrinsic generator of `GFORTRAN`, which is an implementation of the `xoshiro256**` pseudorandom number generator. This generator has a period of 2<sup>256</sup>−1, and when using multiple threads up to 2<sup>128</sup> threads can each generate 2<sup>128</sup> random numbers before any aliasing occurs (for more details cf. the `GFORTRAN` documentation).  
<br>
On input only binary data imported by the `CELL` method are eligible. The `BULK` input method does not index atom positions in a systematic order by relating them to their unit cells, as a consequence it is not possible to establish pairs of atoms in a deterministic way (without searching the whole box).  
<br>
The number of Monte-Carlo events to be generated is specified in units of 10<sup>6</sup> cell pairs corresponding to subcycles, consisting in cross-correlating each of 1000 primary unit cells with 1000 secondary ones. Ten or a few tens of such cycles may be sufficient depending on the exact purpose. A single medium sized supercell (> 10<sup>4</sup> atoms) usually provides enough data to generate a usable PDF set. In case of fractional site occupancies the supercell size may prove too small, the input dialogue permits then to define partial sampling of a larger number of snapshots (supercells). 
<br>
Pseudo-atoms can be defined in the dialogue to easily display correlations involving subgroups of the basis atoms. Several partial PDFs with individually adjustable scale factors can be plotted (maximum number specified in the `.PAR` file) together with the total PDF. The optional output files in the `.PS` (graphics) and `.TXT` (PDF tables) formats can be requested in the dialogue.
<br>

The behaviour of `MP_PDF` can be adjusted by setting the parameter values in the `.PAR` file. The initial file output behaviour is set by the `&MP_OUT` namelist parameters `J_WEIGHT`,`J_PS` and `J_TXT` and can be modified later on in the dialogue. The `&MP_PDF` namelist, specific to this tool, contains the following parameters:
>
>    &MP_PDF  
>     N_PDF = 1024           ! dimension of the PDF array (2**N best for FFT)  
>     PDF_STEP = .06	        ! PDF step [Å]		useful choice: n_pdf=1024, pdf_step=.02  
>     A_PAR_PDF = 4.02 4.02 4.02	! lattice parameter, needed if input in LATTICE units
>     J_RAND = 2            ! 0,1,N defines the seeding of the RANDOM_NUMBERS generator
>     N_PART_MAX = 4        ! maximum number of partial PDFs that can be displayed
>     N_PSEUDO_MAX_ = 4     ! maximum number of pseudo atoms that can be specified 
>     J_SIZE = 0            ! .TXT ouptput size: S corresponding to plot, XXL complete
>
It may be followed by lists specifying the partial PDFs for display, eg.
>    
>    partial_pdf            ! partial PDFs for display in mp_pdf
>     Pb_Pb  1  1   	      ! atom pair name, atom number1, atom number2
>     Mg_Mg  2  2   
>     Nb_Nb  3  3   
>
and defining pseudo-atoms, eg.
>    
>    pseudo_atoms          ! pseudo_atom definitions for mp_pdf
>     B    0 1 1 0 0 0     ! name & 0/1 masks of each basis atom (n_atom on total)
>     O    0 0 0 1 1 1
>
<br>
The PDF accumulation is performed by Monte-Carlo (MC) sampling using the advanced `RANDOM_NUMBER` generator, its seeding behaviour is defined by the `J_RAND` parameter:

`J_RAND = 0` - the generator uses its intrinsic seeding and guaranties, for all practical purposes, the absence of correlation effects between successive pseudo-random number sequences.  The hic is however, that repeated accumulations of the same PDF will coincide only within statistical errors. 

`J_RAND = 1` - in this case each of the 10<sup>6</sup> cell-pair sub-cycles is seeded separately, with seed values determined by the cycle number; in this way the results become  exactly reproducible and permit comparison between accumulations using different numbers of parallel OMP processes (including the OMP-off case), on the other hand this sampling may prove less ideal in terms of possible correlations between the sub-cycles

`J_RAND = N` (any whole number except 0,1, positive or negative) - the generator is seeded just once at the start of the program execution and, thus, offers the same robust behaviour as for `J_RAND = 0` with the added advantage that the reference to `N` makes the results reproducible whenever the program has been re-run for the same MD input with exactly the same sequence of directives, even on different hardware, provided it uses the same version of the intrinsic `RANDOM_NUMBER` procedure - there is a difference between its earlier (eg. GCC v.7) and more recent (eg. GCC v.11) `GFORTRAN` implementations
    