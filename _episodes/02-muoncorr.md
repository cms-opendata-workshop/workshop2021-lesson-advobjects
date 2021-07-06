---
title: "Muon corrections"
teaching: 12
exercises: 0
questions:
- "How are muons corrected with the Rochester corrections?"
objectives:
- "Learn how to apply the corrections to data and MC"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---
There are misalignments in the CMS detector that make the reconstruction of muon momentum biased. The CMS reconstruction software does not fully correct these misalignments and additional corrections are needed to remove the bias. Correcting the misalignments is important in precision measurements which are sensitive to bias in the measurement of muon momenta.

## The Rochester Corrections

The Rochester Corrections are extracted in a two step method. In the first step, initial corrections are obtained in bins of charge, η and ϕ, which are the variables the reconstruction bias in muon momentum depends on. In the second step, the corrections are fine tuned using the mass of the Z boson.

The misalignments for data and Monte Carlo (MC) are different since the MC events start with no biases but they can be induced during reconstruction. Corrections have been extracted for both data and MC events.

In this example, the Run1 Rochester Corrections are used with a 2012 dataset and two MC datasets. The official code for the Rochester Corrections can be found in the directory `RochesterCorrections`. The example code for using the corrections is in the `Test` directory.

## Applying the corrections to data and MC

`Analysis.C` reads the dataset, applies the corrections to a muon pair, computes invariant mass and produces an output-file with corrected data.

Add how to compile and run the code.

~~~
.L Analysis.C+
~~~
{: .language-bash}

~~~
string code;
~~~
{: .language-cpp}

## Plotting the mean invariant mass

`Plot.C` creates a plot of the mean of M(µ+µ-) as a function of η of µ.

Add how to compile and run the code.

~~~
.L Plot.C+
~~~
{: .language-bash}


~~~
string code;
~~~
{: .language-cpp}

Section to talk about the muon corrections and that tool. Add to the setup page some cloning instructions?

{% include links.md %}
