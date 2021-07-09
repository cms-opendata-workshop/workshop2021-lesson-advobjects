---
title: "Advanced objects hands-on"
teaching: 0
exercises: 40
questions:
- "FIXME"
objectives:
- "FIXME"
keypoints:
- "FIXME"
---

FIXME describe the exercises. Rename "challenge" to "exercise"?

>## Challenge: alternate taggers FIXME TO POET AND PAT
>
>Use `edmDumpEventContent` to investiate other b tagging algorithms available as edm::AssociationVector types.
>Add 1-2 new branches for alternate taggers and compare those discriminants to CSV (compile and run cmsRun as you've done before).
>
>>## Solution
>>Let's add the MVA version of CSV and the high purity track counting tagger, which was the most common tagger in 2011.
>>After adding new array declarations and branches in the top sections of `AOD2NanoAOD.cc`, we can open the collections
>>for these alternate taggers:
>>~~~
>>Handle<JetTagCollection> btagsMVA, btagsTC;
>>iEvent.getByLabel(InputTag("trackCountingHighPurBJetTags"), btagsTC);
>>iEvent.getByLabel(InputTag("combinedSecondaryVertexMVABJetTags"), btagsMVA);
>>
>>// inside the jet loop
>>  value_jet_btagmva[value_jet_n] = btagsMVA->operator[](it - jets->begin()).second;
>>  value_jet_btagtc[value_jet_n] = btagsTC->operator[](it - jets->begin()).second;
>>~~~
>>{: .language-cpp}
>>
>>The distributions in ttbar events (excluding events with values of -9 where the tagger wasn't evaluated) look like this:
>>![](../assets/img/TTbar_btaggers.png)
>{: .solution}
{: .challenge}

>## Challenge: count medium CSV b tags
>
>Calculate the number of jets per event that are b tagged according to the medium working point of the CSV algorithm.
>
>>## Solution
>>We count the number of "Medium CVS" b-tagged jets by summing up the number of jets with discriminant values greater than 0.679.
>>After adding a variable declaration and branch we can sum up the counter:
>>
>>FIXME TO POET AND PAT
>>~~~
>>value_jet_nCSVM = 0;
>>for (auto it = jets->begin(); it != jets->end(); it++){
>>
>>  // skipping bits
>>
>>  value_jet_btag[value_jet_n] = btags->operator[](it - jets->begin()).second
>>  if (value_jet_btag[value_jet_n] > 0.679) value_jet_nCSVM++;
>>  value_jet_n++;
>>}
>>~~~
>>{: .language-cpp}
>{: .solution}
{: .challenge}

>## Challenge: create PAT jets
>
>Run `simulation_patjets_cfg.py`, open the file, and compare the two jet correction and b-tagging methods. Method 1 has `Jet_` and `CorrJet_` branches
>and Method 2 has `PatJet_` and `PatJet_uncorr` branches.
>
>>## Solution
>>An important difference between value_jet_pt and value_uncorr_patjet_pt is how the momentum threshold is applied: in PFJets all uncorrected jets have pT > 15 GeV
>>while in PATJets this is applied to the corrected jets. There are small deviations in the corrected jet momentum between the 2 methods, most likely because
>>of differences between the `rho` collection used for pileup corrections.
>{: .solution}
{: .discussion}

{% include links.md %}

