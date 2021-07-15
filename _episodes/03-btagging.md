---
title: "Demo: Heavy flavor tagging"
teaching: 12 min
exercises: 
questions:
- "How are b hadrons identified in CMS"
objectives:
- "Understand the basics of heavy flavor tagging"
- "Learn to access tagging information in AOD files"
keypoints:
- "Tagging algorithms separate heavy flavor jets from jets produced by the hadronization of light quarks and gluons"
- "Tagging algorithms produce a disriminator value for each jet that represents the likelihood that the jet came from a b hadron"
- "Each tagging algorithm has recommended 'working points' (discriminator values) based on a misidentification probability for light-flavor jets"
---


Jet reconstruction and identification is an important part of the analyses at the LHC. A jet may contain
the hadronization products of any quark or gluon, or possibly the decay products of more massive particles such as W or Higgs bosons.
Several "b tagging" algorithms exist to identify jets from the hadronization of b quarks, which have unique
properties that distinguish them from light quark or gluon jets. 


## B Tagging Algorithms

Tagging algorithms first connect the jets with good quality tracks that are either associated with one of the jet's particle flow candidates or within a nearby cone.
Both tracks and "secondary vertices" (track vertices from the decays of b hadrons) can be used in track-based, vertex-based, or "combined" tagging algorithms.
The specific details depend upon the algorithm use. However, they all exploit properties of b hadrons such as:

 * long lifetime,
 * large mass,
 * high track multiplicity,
 * large semileptonic branching fraction,
 * hard fragmentation fuction. 

Tagging algorithms are Algorithms that are used for b-tagging:

 * Track Counting: identifies a b jet if it contains at least N tracks with significantly non-zero impact parameters.
 * Jet Probability: combines information from all selected tracks in the jet and uses probability density functions to assign a probability to each track
 * Soft Muon and Soft Electron: identifies b jets by searching for a lepton from a semi-leptonic b decay.
 * Simple Secondary Vertex: reconstructs the b decay vertex and calculates a discriminator using related kinematic variables.
 * **Combined Secondary Vertex**: exploits all known kinematic variables of the jets, information about track impact parameter significance and the secondary vertices
 to distinguish b jets. This tagger became the default CMS algorithm.

These algorithms produce a single, real number (often the output of an MVA) called a b tagging "discriminator" for each jet. The more positive the discriminator
value, the more likely it is that this jet contained b hadrons. 

## Accessing tagging information

In `PatJetAnalyzer.cc` we access the information from the Combined Secondary Vertex (CSV) b tagging algorithm and associate discriminator values with the jets.
The CSV values are stored in a separate collection in the AOD files called a `JetTagCollection`, which is effectively a vector of associations between jet references and
float values (such as a b-tagging discriminator). During the creation of `pat::Jet` objects, this information is merged into the jets so that the discriminators can be accessed with a dedicated member function (similar merging is done for other useful associations like generated jets and jet flavor in simulation!).

> For examples of working with b tags and RECO jets, see [last year's lesson](https://cms-opendata-workshop.github.io/workshop-lesson-jetmet/02-btagging/index.html).
{: .callout}

~~~
#include "DataFormats/PatCandidates/interface/Jet.h"

Handle<std::vector<pat::Jet>> myjets;
iEvent.getByLabel(jetInput, myjets); // jetInput opens "selectedPatJetsAK5PFCorr"

for (std::vector<pat::Jet>::const_iterator itjet=myjets->begin(); itjet!=myjets->end(); ++itjet){

  jet_btag.push_back(itjet->bDiscriminator("combinedSecondaryVertexBJetTags"));

}
~~~
{: .language-cpp}

We can investigate at the b tag information in a dataset using ROOT. 

~~~
$ root -l myoutput.root // you produced this earlier in hands-on exercise #2
[0] _file0->cd("myjets");
[1] Events->Draw("jet_btag");
~~~
{: .language-bash}

The discriminator values for jets with kinematics in the correct range for this algorithm lie between 0 and 1. Other jets that do not have a valid discriminator value pick up values of -1, -9, or -999.


## Working points

A jet is considered "b tagged" if the discriminator value exceeds some threshold. Different thresholds will have different
efficiencies for identifying true b quark jets and for mis-tagging light quark jets. As we saw for muons and other objects,
a "loose" working point will allow the highest mis-tagging rate, while a "tight" working point will sacrifice some correct-tag
efficiency to reduce mis-tagging. The [CSV algorithm has working points](https://twiki.cern.ch/twiki/bin/view/CMSPublic/BtagRecommendation2011OpenData)
defined based on mis-tagging rate: 

 * Loose = ~10% mis-tagging = discriminator > 0.244
 * Medium = ~1% mis-tagging = discriminator > 0.679 
 * Tight = ~0.1% mis-tagging = discriminator > 0.898 


## Correcting efficiency differences

When training a tagging algorithm, it's highly probable that the efficiencies for tagging different quark flavors as b jets will vary between simulation
and data. These differences must be measured and corrected for using "scale factors" constructed from ratios of the efficiencies from different sources. The figures below
show examples of the b and light quark efficiencies and scale factors as a function of jet momentum ([read more](https://twiki.cern.ch/twiki/bin/view/CMSPublic/PhysicsResultsBTV13001)).

![](../assets/img/bEff.PNG) ![](../assets/img/lightEff.PNG)

In simulation, the relevant efficiencies are defined as:
 * b efficiency = [number of "real b jets" (jets spatially matched to generator-level b hadrons) tagged as b jets] / [number of real b jets]
 * c efficiency = [number of "real c jets" (jets spatially matched to generator-level c hadrons) tagged as c jets] / [number of real c jets]
 * light/gluon efficiency (often called "mistag rate") = [number of "real light/gluon jets" (jets spatially matched to generator-level light hadrons) tagged as light/gluon jets] / [number of real light/gluon jets]

These values are typically computed as functions of the momentum or pseudorapidity of the jet and are computed by analyzers for simulated samples with kinematics relevant to their analysis. In POET, a [BTagging module](https://github.com/cms-legacydata-analyses/PhysObjectExtractorTool/tree/master/BTagging) has been provided for calculating efficiencies. 

As an example, efficiencies for the CSV Medium working point in the top quark pair sample have been computed and stored in lookup function in `PatJetAnalyzer.cc`. The CSV algorithm has a b-quark efficiency of ~60% and a light quark mistag rate of ~1%, as advertised. 
~~~
double PatJetAnalyzer::getBtagEfficiency(double pt){
  if(pt < 25) return 0.263407;
  else if(pt < 50) return 0.548796;
  else if(pt < 75) return 0.656801;
  ...etc...
  else if(pt < 400) return 0.625296;
  else return 0.394916;
}

double PatJetAnalyzer::getLFtagEfficiency(double pt){
  if(pt < 25) return 0.002394;
  else if(pt < 50) return 0.012683;
  else if(pt < 75) return 0.011459;
  ...etc...
  else if(pt < 400) return 0.014760;
  else return 0.011628;
}
~~~
{: .language-cpp}
![](../assets/img/beffs.PNG)

## Applying scale factors

Scale factors to increase or decrease the number of b-tagged jets in simulation can be applied in a number of ways, but typically involve weighting simulation
events based on the efficiencies and scale factors relevant to each jet in the event. Scale factors for the CSV algorithm are available for Open Data and involve
extracting functions from a comma-separated-values file. Details and usage reference can be found at these references:

 * [Explanation](https://twiki.cern.ch/twiki/bin/view/CMSPublic/BtagRecommendation2011OpenData#Data_MC_Scale_Factors)
 * [Data file for the CSV algoritm](https://twiki.cern.ch/twiki/pub/CMSPublic/BtagRecommendation2011OpenData/CSV.csv)
 * [Examples of application methods](https://twiki.cern.ch/twiki/bin/view/CMSPublic/BtagRecommendation2011OpenData#Methods_to_Apply_b_Tagging_Effic)

In POET, the scale factor functions in the comma-separated-values file for the Medium working point have been implemented in `PatJetAnalyzer.cc`. The scale factor is a single numerical value calculated from a function of the corrected jet momentum:
~~~
double PatJetAnalyzer::getBorCtagSF(double pt, double eta){
  if (pt > 670.) pt = 670;
  if(fabs(eta) > 2.4 or pt<20.) return 1.0;

  return 0.92955*((1.+(0.0589629*pt))/(1.+(0.0568063*pt)));
}
~~~
{: .language-cpp}

The most commonly used method to apply scale factors is [Method 1a](https://twiki.cern.ch/twiki/bin/view/CMSPublic/BtagRecommendation2011OpenData#1a_Event_reweighting_using_scale), and it has been implemented in POET. This is an **event weight** method of applying a correction, so the final result is an event weight to be stored for later in the analysis, along with extra event weights to describe the uncertainty.

The method relies on 4 pieces of information for each jet in simulation:
 * Tagging status: does this jet pass the discriminator threshold for a given working point?
 * Flavor (b, c, light): accessed using a `pat::Jet` member function called `partonFlavour()`.
 * Efficiency: accessed from the lookup functions based on the jet's momentum.
 * Scale factor: accessed from the helper functions based on the jet's momentum.

Each jet then contributes to the event weight based on these four features. For example, jets that pass the discriminator threshold and are b quark jets contribute the following factor to variables called `MC`, which appears as P(MC) in the reference TWiki, and `btagWeight`, which appears as P(DATA) in the reference TWiki.
~~~
hadronFlavour = itjet->partonFlavour();
corrpt = corr_jet_pt.at(value_jet_n);

if (jet_btag.at(value_jet_n) > 0.679){
  if(abs(hadronFlavour) == 5){
    eff = getBtagEfficiency(corrpt);
    SF = getBorCtagSF(corrpt, jet_eta.at(value_jet_n));
    SFu = SF + uncertaintyForBTagSF(corrpt, jet_eta.at(value_jet_n));
    SFd = SF - uncertaintyForBTagSF(corrpt, jet_eta.at(value_jet_n));
  }
  // ... repeat for c and light quark jets...
  MC *= eff;
  btagWeight *= SF * eff;
  btagWeightUp *= SFu * eff;
  btagWeightDn *= SFd * eff;
}
//...similar for non-tagged jets...
~~~
{: .language-cpp}

After the jet loop, the reference TWiki defines the weight as P(DATA)/P(MC):
~~~
btagWeight = (btagWeight/MC);
btagWeightUp = (btagWeightUp/MC);
btagWeightDn = (btagWeightDn/MC);
~~~
{: .language-cpp}

The weight and its uncertainties are all stored in the tree for use later in filling histograms. 

{% include links.md %}

