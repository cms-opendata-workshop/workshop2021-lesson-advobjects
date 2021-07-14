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
There are misalignments in the CMS detector that make the reconstruction of muon momentum biased. The CMS reconstruction software does not fully correct these misalignments and additional corrections are needed to remove the bias. Correcting the misalignments is important when further analysis or computing is done using the muon momentum, because the bias in muon momentum will affect the results.

## The Muon Momentum Scale Corrections

The Muon Momentum Scale Corrections, also known as the Rochester Corrections, are extracted in a two step method. In the first step, initial corrections are obtained in bins of the charge of the muon and the η and ϕ coordinates of the muon track. The reconstruction bias in muon momentum depends on these variables. In the second step, the corrections are fine tuned using the mass of the Z boson.

The corrections for data and Monte Carlo (MC) are different since the MC events start with no biases but they can be induced during the reconstruction. Corrections have been extracted for both data and MC events.

In this example, the Run1 Rochester Corrections are used with a 2012 dataset and a MC dataset. The official code for the Rochester Corrections can be found in the directory `RochesterCorrections`. The example code for applying the corrections is in the `Test` directory.

## Applying the corrections to data and MC

Let's start by opening ROOT in a terminal and compiling the official corrections code by running the following lines.

~~~
cd MuonCorrectionsTool
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

The first thing `applyCorrections` does is create an RDataFrame from the ROOT-file. The RDataFrame can be thought of as an array where the variables from the ROOT-file make up columns. We use the RDataFrame function `Define` to add new columns for variables needed in applying the corrections and for the corrected values. `Define` takes as a parameter the name of the new column, a function and a list of RDataFrame columns. `Define` automatically loops over the given columns, performs the given function on each event and saves the results to the new column.

This tutorial produces a plot which shows that the corrections have been applied correctly. In the y-axis of the plot we have the invariant mass of μ<sup>+</sup>μ<sup>-</sup>, which is why the events are filtered to muon pairs with opposite charges and the invariant mass is computed. After this a few more columns needed for the plot are added.

~~~
int applyCorrections(string filename, string pathToFile, bool isData) {
  // Create dataframe from NanoAOD files
  ROOT::RDataFrame df("Events", pathToFile);
  
  // Select events with exactly two muons
  auto df_2mu = df.Filter("nMuon == 2", "Events with exactly two muons");

  // Select events with two muons of opposite charge
  auto df_os = df_2mu.Filter("Muon_charge[0] != Muon_charge[1]", "Muons with opposite charge");

  // Compute invariant mass of the dimuon system
  auto df_mass = df_os.Define("Dimuon_mass", computeInvariantMass, {"Muon_pt", "Muon_eta", "Muon_phi", "Muon_mass"});
~~~
{: .language-cpp}

Next step in `applyCorrections` is adding TLorentzVectors. The functions for applying the Rochester Corrections take as a parameter a TLorentzVector which is a four-vector that describes the muons momentum and energy. We add a new column called *TLVectors* by using `Define` and the `createVector` function which uses muon pt, eta, phi and mass to create the vectors.

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

As mentioned earlier, the muon momentum scale corrections are different for data and MC and therefore there are separate functions for both. In `applyCorrections`, we call either `correctDataMuon` or `correctMCMuon` to create a new column for the corrected muons.

~~~
// Run the correctios and add corrected muons as a new column
  auto df_cor = std::make_unique<RNode>(df_tlv);

  if(isData){
    df_cor = std::make_unique<RNode>(df_cor->Define("CorrectedMuons", correctDataMuon, {"TLVectors","Muon_charge"}));
  } else {
    df_cor = std::make_unique<RNode>(df_cor->Define("CorrectedMuons", correctMCMuon, {"TLVectors","Muon_charge"}));
  }
~~~
{: .language-cpp}

These functions further call the rochor class functions `momcor_mc` and `momcor_data` to apply the corrections. In `applyCorrections`, the corrected values of muon variables are then extracted to their own columns and the corrected invariant mass is computed. The dataframe is saved to a new ROOT-file. The rochor class functions can be found in `rochcor2012wasym.cc` if you want to take a look at them.

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

Compile and run `Analysis.C` by running the following lines in your ROOT-terminal.

~~~
.L RochesterCorrections/Test/Analysis.C+
Analysis pf
pf.main()
~~~
{: .language-bash}

## Plotting the mean invariant mass

To make sure the corrections have been applied correctly, we create a plot of the mean of M(μ<sup>+</sup>μ<sup>-</sup>) as a function of η of μ<sup>+</sup> and μ<sup>-</sup>. The data is divided into bins by muon η and a fit is made for each bin. The mean values of the fits are saved to a histogram. This process is done for both η of μ<sup>+</sup> and η of μ<sup>-</sup> and for uncorrected and corrected data and MC. The histograms are plotted resulting in the picture below.

ADD PLOT

To create the plot, make sure you have the corrected data and MC files, `Run2012BC_DoubleMuParked_Muons_Cor.root` and `ZZTo2e2mu_Cor.root`, and compile and run `Plot.C` by running the lines below.

~~~
.L Plot.C+
main()
~~~
{: .language-bash}


ADD CLONING INSTRUCTIONS TO SETUP PAGE.

{% include links.md %}
