---
title: "Demo: Muon corrections"
teaching: 12
exercises: 0
questions:
- "How to correct biased muon momentum?"
objectives:
- "Learn how to apply muon momentum scale corrections to data and MC."
keypoints:
- "Rochester corrections are used to scale the muon momentum so that simulation better matches data."
---
There are misalignments in the CMS detector that make the reconstruction of muon momentum biased. The CMS reconstruction software does not fully correct these misalignments and additional corrections are needed to remove the bias. Correcting the misalignments is important when precision measurements are done using the muon momentum, because the bias in muon momentum will affect the results.

## The Muon Momentum Scale Corrections

The Muon Momentum Scale Corrections, also known as the Rochester Corrections, are available in the [MuonCorrectionsTool](https://github.com/cms-legacydata-analyses/MuonCorrectionsTool). The correction parameters have been extracted in a two step method. In the first step, initial corrections are obtained in bins of the charge of the muon and the η and ϕ coordinates of the muon track. The reconstruction bias in muon momentum depends on these variables. In the second step, the corrections are fine tuned using the mass of the Z boson.

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

The main function of `Analysis.C` is simply used for calling the `applyCorrections` function which takes as a parameter the name of the ROOT-file (without the `.root`-part), path to the ROOT-file and a boolean value of whether the file contains data (`true`) or MC (`false`).

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

The first thing `applyCorrections` does is create a TTree from the ROOT-file. Then variables for holding the values read from the tree are created and branch addresses are set so that the variables are populated when looping over events. New branches for the corrected values, an output file and a few variables needed for the corrections are also created.

~~~
int applyCorrections(string filename, string pathToFile, bool isData) {
  // Create TTree from ROOT file
  TFile *f1 = TFile::Open((pathToFile).c_str());
  TTree *DataTree = (TTree*)f1->Get("Events");
~~~
{: .language-cpp}

Next, we loop over the events and apply the corrections. We are later going to do an exercise to make sure the corrections have been applied correctly. We are going to need the invariant mass of μ<sup>+</sup>μ<sup>-</sup>, which is why the events are filtered to muon pairs with opposite charges and the invariant mass is computed.

~~~
// Loop over events
  Int_t nEntries = (Int_t)DataTree->GetEntries();

  for (Int_t k=0; k<nEntries; k++) {
    DataTree->GetEntry(k);

    // Select events with exactly two muons
    if (nMuon == 2 ) {
      // Select events with two muons of opposite charge
      if (Muon_charge[0] != Muon_charge[1]) {

        // Compute invariant mass of the dimuon system
        Dimuon_mass = computeInvariantMass(Muon_pt[0], Muon_pt[1], Muon_eta[0], Muon_eta[1], Muon_phi[0], Muon_phi[1], Muon_mass[0], Muon_mass[1]);
        bDimuon_mass->Fill();
~~~
{: .language-cpp}

We then loop over the muons in an event and create a TLorentzVector for each muon. The functions for applying the Rochester Corrections take as a parameter a TLorentzVector which is a four-vector that describes the muons momentum and energy. As mentioned earlier, the muon momentum scale corrections are different for data and MC and therefore there are separate functions for both: `momcor_data` and `momcor_mc`. These functions can be found in `rochcor2012wasym.cc` if you want to take a closer look at them.

~~~
        // Loop over muons in event
        for (UInt_t i=0; i<nMuon; i++) {

          // Fill positive and negative muons eta branches
          if (Muon_charge[i] > 0) {
            Muon_eta_pos[i] = Muon_eta[i];
            bMuon_eta_pos->Fill();
          } else {
            Muon_eta_neg[i] = Muon_eta[i];
            bMuon_eta_neg->Fill();
          }

          // Create TLorentzVector
          TLorentzVector mu;
          mu.SetPtEtaPhiM(Muon_pt[i], Muon_eta[i], Muon_phi[i], Muon_mass[i]);

          // Apply the corrections
          if (isData) {
            rmcor.momcor_data(mu, Muon_charge[i], runopt, qter);
          } else {
            rmcor.momcor_mc(mu, Muon_charge[i], ntrk, qter);
          }
~~~
{: .language-cpp}

The corrected values are then saved to the new branches and the corrected invariant mass is computed. The new tree is filled and written to the output file.

~~~
          // Save corrected values
          Muon_pt_cor[i] = mu.Pt();
          bMuon_pt_cor->Fill();
          Muon_eta_cor[i] = mu.Eta();
          bMuon_eta_cor->Fill();
          Muon_phi_cor[i] = mu.Phi();
          bMuon_phi_cor->Fill();
          Muon_mass_cor[i] = mu.M();
          bMuon_mass_cor->Fill();
        }

        // Compute invariant mass of the corrected dimuon system
        Dimuon_mass_cor = computeInvariantMass(Muon_pt_cor[0], Muon_pt_cor[1], Muon_eta_cor[0], Muon_eta_cor[1], Muon_phi_cor[0], Muon_phi_cor[1], Muon_mass_cor[0], Muon_mass_cor[1]);
        bDimuon_mass_cor->Fill();

      }
    }
    //Fill the corrected values to the new tree
    DataTreeCor->Fill();
  }
  
  //Save the new tree
  DataTreeCor->Write();

~~~
{: .language-cpp}

Compile and run `Analysis.C` by running the following lines in your ROOT-terminal. You will get a bunch of warnings but you can safely ignore them.

~~~
.L RochesterCorrections/Test/Analysis.C+
Analysis pf
pf.main()
~~~
{: .language-bash}

And that's it! As a result you can find `Run2012BC_DoubleMuParked_Muons_Cor.root` and `ZZTo2e2mu_Cor.root` in the `Test` directory. These files contain both corrected and uncorrected values.

{% include links.md %}
