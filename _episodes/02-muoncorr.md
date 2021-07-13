---
title: "Demo: Muon corrections"
teaching: 12
exercises: 0
questions:
- "How to correct biased muon momentum?"
objectives:
- "Learn how to apply muon momentum scale corrections to data and MC"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---
There are misalignments in the CMS detector that make the reconstruction of muon momentum biased. The CMS reconstruction software does not fully correct these misalignments and additional corrections are needed to remove the bias. Correcting the misalignments is important when further analysis or computing is done using the muon momentum, because then the bias in muon momentum will affect the results. For example, if only the information about the number of muons in an event is used, the corrections are not necessary.

## The Muon Momentum Scale Corrections

The Muon Momentum Scale Corrections, also known as the Rochester Corrections, are extracted in a two step method. In the first step, initial corrections are obtained in bins of charge, η and ϕ. The reconstruction bias in muon momentum depends on these variables. In the second step, the corrections are fine tuned using the mass of the Z boson.

The misalignments for data and Monte Carlo (MC) are different since the MC events start with no biases but they can be induced during the reconstruction. Corrections have been extracted for both data and MC events.

In this example, the Run1 Rochester Corrections are used with a 2012 dataset and a MC dataset. The official code for the Rochester Corrections can be found in the directory `RochesterCorrections`. The example code for using the corrections is in the `Test` directory.

## Applying the corrections to data and MC

Let's start by opening ROOT in a terminal and compiling the official corrections code.

~~~
root
.L RochesterCorrections/muresolution.cc++
.L RochesterCorrections/rochcor2012wasym.cc++
~~~
{: .language-bash}

Then let's take a look at `Analysis.C` which is where we apply the corrections. 

The main function of `Analysis.C` is simply used to call the `applyCorrections` function which takes as a parameter the name of the ROOT-file (without the `.root`-part), path to the ROOT-file and a boolean value of whether the file contains data (`true`) or MC (`false`).

~~~
void Analysis::main()
{
  // Data
  applyCorrections("Run2012BC_DoubleMuParked_Muons", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/Run2012BC_DoubleMuParked_Muons.root", true);

  // MC
  applyCorrections("ZZTo2e2mu", "root://eospublic.cern.ch//eos/opendata/cms/upload/stefan/HiggsToFourLeptonsNanoAODOutreachAnalysis/ZZTo2e2mu.root", false);
}
~~~
{: .language-cpp}

The first thing `applyCorrections` does is create an RDataFrame from the ROOT-file. The RDataFrame can be thought of as an array where the variables from the ROOT-file make up columns. We use the RDataFrame function `Define` to add new columns for variables needed to apply the corrections and for the corrected muon properties. `Define` takes as a parameter the name of the new column, a function and a list of RDataFrame columns. `Define` automatically loops over the given columns and performs the given function on each event.

~~~
int applyCorrections(string filename, string pathToFile, bool isData) {
  // Create dataframe from NanoAOD files
  ROOT::RDataFrame df("Events", pathToFile);
~~~
{: .language-cpp}

The official functions for applying the corrections take as a parameter a TLorentzVector which is a four-vector that describes the muons momentum and energy. We add a new column called *TLVectors* for the TLorentzVectors by using `Define` and `createVector`.

~~~
// Create TLorentzVectors
RVec<TLorentzVector> createVector(RVec<float>& pt, RVec<float>& eta, RVec<float>& phi, RVec<float>& mass) {
  TLorentzVector mu1;
  TLorentzVector mu2;
  mu1.SetPtEtaPhiM(pt[0], eta[0], phi[0], mass[0]);
  mu2.SetPtEtaPhiM(pt[1], eta[1], phi[1], mass[1]);
  RVec<TLorentzVector> vectors {mu1, mu2};

  return vectors;
}
~~~
{: .language-cpp}

As mentioned earlier, the muon momentum scale corrections are different for data and MC and therefore there are separate functions for both. For MC the rochor class function `momcor_mc` is called and for data `momcor_data`. Both of these functions are found in `rochcor2012wasym.cc`. Below are the functions in `Analysis.C` which are used to call the rochor class functions and to return the corrected TLorentzVectors to a new column.

~~~
// Add corrections to MC muons
RVec<TLorentzVector> correctMCMuon(RVec<TLorentzVector> muons, RVec<int>& charge) {
  rochcor2012 rmcor; // make the pointer of rochcor class
  float ntrk = 0; //ntrk (number of track layer) is one of input and it can slightly improved the extra smearing
  float qter = 1.0; // added it by Higgs group’s request to propagate the uncertainty

  rmcor.momcor_mc(muons[0], charge[0], ntrk, qter);
  rmcor.momcor_mc(muons[1], charge[1], ntrk, qter);
  RVec<TLorentzVector> vectors {muons[0], muons[1]};

  return vectors;
}

// Add corrections to data muons
RVec<TLorentzVector> correctDataMuon(RVec<TLorentzVector> muons, RVec<int>& charge) {
  rochcor2012 rmcor; // make the notpointer of rochcor class
  float runopt = 0; //No run dependence for 2012 data, so default of “runopt=0”
  float qter = 1.0; // added it by Higgs group’s request to propagate the uncertainty

  rmcor.momcor_data(muons[0], charge[0], runopt, qter);
  rmcor.momcor_data(muons[1], charge[1], runopt, qter);
  RVec<TLorentzVector> vectors {muons[0], muons[1]};

  return vectors;
}
~~~
{: .language-cpp}

Compile and run `Analysis.C` by running the following lines in your ROOT-terminal. As a result you get a new ROOT-file with both the old and the new values.

~~~
.L RochesterCorrections/Test/Analysis.C+
Analysis pf
pf.main()
~~~
{: .language-bash}

## Plotting the mean invariant mass

`Plot.C` creates a plot of the mean of M(µ+µ-) as a function of η of µ.

Compile and run the code.

~~~
.L Plot.C+
main()
~~~
{: .language-bash}

Explain what the code does.

~~~
string code;
~~~
{: .language-cpp}

Section to talk about the muon corrections and that tool. Add to the setup page some cloning instructions?

{% include links.md %}
