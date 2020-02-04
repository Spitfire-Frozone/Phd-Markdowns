# This will be aimed at my the second part of my qualification task which is to join the efforts surrounding the study of fake rates. This documentation gives help when trying to add MC based truth studies to the framework.

## "Fake Rate Investigations" ##

## Last Edited: 21-09-2018

### I highly recommend you read Fake_Rate_Qualification_Task.md if you think some steps are skipped without explanation. This will hit the ground running so to speak. ###

# Investigating the Fake Rate


## Cloning ##
Active and up-to-date repository is now.
### https://gitlab.cern.ch/Atlas-Inner-Tracking/atlas-trackingCP-fakestudy ###
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/
setupATLAS && lsetup git
mkdir build
git clone --recurse-submodules https://gitlab.cern.ch/Atlas-Inner-Tracking/atlas-trackingCP-fakestudy/
~~~

Create a topic branch to store further changes.
~~~
git fetch origin
git checkout -b master_[something] upstream/master --no-track
> git checkout -b [branch to merge to]-[name] [upstream/origin]/[parent_branch] --no-track
~~~

##  Additions to General Fake Rate Plots ##
It was discussed earlier on in the year (but there was little time to impliment it at this point) about adding in a mu-based efficiency scaling to the NTracksVsMu plots. First lets check out a new topic branch
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/
setupATLAS && lsetup git
git checkout -b master_EffLossAddition-dspiteri origin/master --no-track
~~~
Essentially what is wanted is an mu-dependent fit modification that follows the trend that allows for the track-efficiency loss for TightPrimary tracks to be present in our recommendations. The first thing then is to obtain functional forms for the mu-dependent efficiency loss
>  https://indico.cern.ch/event/571469/contributions/2746526/attachments/1535755/2405751/TrackingEfficiencyZ_20171005.pdf#search=harish
> https://indico.cern.ch/event/571443/contributions/2546493/attachments/1441502/2219470/ppt.pdf#search=harish
> https://indico.cern.ch/event/703761/contributions/2905563/attachments/1605443/2546967/MuDependenceInMC_LumiMeeting_VLang_22022018.pdf#search=valerie%20lang

and incorporate them into the TF1 fit posted on the plots in NTracksVsMu_datamc_fakerate_v3.C.
~~~
vim /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/NTracksVsMu_datamc_fakerate_v3.C.
~~~
>   CHANGE ~ L282-283, 296, 300
>   >     TF1* fitTightData = new TF1("fitTightData", "x*(1-0.000120*x)*[0]");
>   >     TF1* fitTightMC = new TF1("fitTightMC", "x*(1-0.000122*x)*[0]");
>   >     TF1* fitTightInnerd0Data = new TF1("fitTightInnerd0Data", "x*(1-0.000120*x)*[0]");
>   >     TF1* fitTightInnerd0MC   = new TF1("fitTightInnerd0MC", "x*(1-0.000122*x)*[0]");
Then to check that this is working run the code once provisionally and check that there is a recommendations file still produced.

~~~
rcSetup Base,2.5.0
rc find_packages && rc compile
mkdir ZeroBias_FakeTracks/Plots/_G-I_
mkdir ZeroBias_FakeTracks/Plots/EffLoss

root -b -l -q 'ZeroBias_FakeTracks/NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/I-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_G-I_","2017Data2vMC16c",0)'

vim ZeroBias_FakeTracks/Plots/_G-I_/Recommendations_RAW.txt (to check the numbers)
rm ZeroBias_FakeTracks/Plots/_G-I_/Recommendations_RAW.txt
~~~
If this seems to work then try running over the variations and see if anything changes. However the files will be placed in the temporary folder \_G-I\_ so will have to be moved to the EffLoss folder.
~~~
rm ZeroBias_FakeTracks/Plots/_G-I_/*
source ZeroBias_FakeTracks/runRecommendationVariations.sh
mv ZeroBias_FakeTracks/Plots/_G-I_/* ZeroBias_FakeTracks/Plots/EffLoss/
~~~

And then generate recommendations in a new shell (using cmake so a different analysis base is effectively being used.)
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/
setupATLAS && lsetup git
mkdir build
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/build
setupATLAS
asetup AnalysisBase,21.2.30 && lsetup "ROOT 6.10.04"
cmake ../atlas-trackingCP-fakestudy/
make -j6
source x86_64-slc6-gcc62-opt/setup.sh

cd ../atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/
Recom_beatif Plots/EffLoss/Recommendations_RAW.txt > Plots/EffLoss/Recommendations.txt
Recom_beatif Plots/EffLoss/Recommendations_Inner_d0_RAW.txt > Plots/EffLoss/Recommendations_Inner_d0.txt
~~~
##  Studying "true" fakes by probing the MC ##
The aim of this section is to use the full-truth samples to look at the behaviour of the 'true' fakes based on their truth matching probability and then use this knowledge to better understand the composition of the non-linear part of NTracks vs mu trend. Once again we start of with a topic branch.

At the time of writing this the MC16c sample that I am using does not have full pile-up truth and only event level truth (which for tracking is useless as it's a simple neutrino sample). I will have to revert to MC16a to get a MC16 sample with the information required.

If you need to re-run the sample on the grid again look in Fake_Rate_Study.md for the sample name and run.
user.dspiteri.159000.ParticleGenerator_nu_E50._Rel21_MC16a_1_myOutput.root -> MC16a

~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/
setupATLAS && lsetup git
git checkout -b master_MCTruthStudies-dspiteri origin/master --no-track
~~~

For this the macro TrackTruthHelper will be very useful
~~~
vim MyVertexAnalysis/Root/TrackTruthHelper.cxx
~~~
One useful function is TrackTruthHelpers::isPrimary which checks if the track is first a truth track, and then if it passes the truth match cut.

>     if (!passesTruthMatchProbCut(track,m_truthCut))
>       return false;

This is effectively a cut on P_match = 0.5 where P_match is the sum of weighted hits associated to a track divided by the total sum of truth hits associated to that track. This is a measure of the trimmed ntuptes that describes the likelihood that this track is actually a fake. But if one wants to check that actually reconstructed tracks are correctly matched to a truth track and if such links properly exist, one needs to use TrackTruthHelpers::truthParticle. Create one for barcodes.

>     const xAOD::TruthParticle* TrackTruthHelpers::truthParticle(const xAOD::TrackParticle *track)
>     {
>     typedef ElementLink< xAOD::TruthParticleContainer > Link_t;
>     static const char* NAME = "truthParticleLink";
>     if( ! track->isAvailable< Link_t >( NAME ) ) {
>     return 0;
>     }
>     const Link_t& link = track->auxdata< Link_t >( NAME );
>     if( ! link.isValid() ) {
>     return 0;
>     }
>
>     return *link;
>     }

Which checks if a link exists between a newly reconstructed track and a truth track by comparing the reconstructed tracks to the truth tracks and if they pass some criterion, the track is linked. Tracks that are NOT linked to truth tracks are very likely to be fake. Also each track is assigned a barcode to ensure that it is unique, and this should be further investigated.

The code will need to be reproduced again with new information in the ntuples which means that MyVertexAnalysis.cxx will have to be edited, but the first thing to do is add a useful member function to the wrapper struct for the tracks.
~~~
vim MyVertexAnalysis/MyVertexAnalysis/MyTrack.h
~~~
>    ADD barcode integer to MyTruthTrack (~L14)
>   >     int barcode;
>    ADD truthmatching boolean to MyTrack (~L28)
>   >     float truthProb;
>   >     bool hasTruthLink;
~~~
vim MyVertexAnalysis/Root/MyVertexAnalysis.cxx
~~~
> COMMENT OUT pile up reweighting luninosity calculations file which is not needed for truth (~L558)
>   >     lumicalcFiles.push_back("$ROOTCOREBIN/data/MyVertexAnalysis/ilumicalc_histograms_L1_ZB_297447-311481.root");

> ADD Investigation of the number of tracks that fail the pre-selection (L867)
>   >     int NoTruthParticles =0;
>   >     int NoTruthParticlesPassingPreSelCuts =0;
>   >     NoTruthParticles++; (L872)
>   >     NoTruthParticlesPassingPreSelCuts++; (L874)
>  (L885-888)
>   >     double AcceptedFraction = NoTruthParticlesPassingPreSelCuts/(double)NoTruthParticles;
>   >     std::cout<<"-------------------------------------------------"<<std::endl;
>   >     std::cout<<"FRACTION OF ACCEPTED TRACKS = "<<AcceptedFraction<<" for truthtrk #"<<NoTruthParticles<<std::endl;
>   >     std::cout<<"Truthtrk #"<<NoTruthParticles<<"'s barcode is"<<truthtrk->barcode()<<std::endl;

> ADD the filling of the track barcodes (~L880)
>   >     myTruthTrack.barcode = truthtrk->barcode();

>   ADD the filling of the truth links boolean (~L933)
>  >     //Truth Matching
>  >     if (m_tracktruthhelper->truthParticle(trk) == 0){
>  >     myTrack.hasTruthLink = false;
>  >     } else {
>  >     myTrack.hasTruthLink = true;
>  >     }


>   ADD the storage of the track truth probability (L900)
>   >     myTrack.truthProb = track->auxdata< float >( "truthMatchProbability" );


Remember, if you want to make new plots, they are stored in FakeRecommendationsPlots.cxx.
~~~
vim MyVertexAnalysis/Root/FakeRecommendationsPlots.cxx
~~~
> ADD header file containing the truth functions to be used
>   >     #include "MyVertexAnalysis/MyVertexAnalysis.h" (L2)

>   ADD new plots (~L36)
>   >     p_nTruthLinkedTracksLoose_vs_muAvg = new TProfile(m_name+"nTruthLinkedTracksLoose_vs_muAvg", "Number of loose reco tracks with truth links versus #mu mean; <#mu>;<nTruthLinkedTracksLoose>", nMu, minMu, maxMu);
>   >     p_nTruthLinkedTracksTight_vs_muAvg = new TProfile(m_name+"nTruthLinkedTracksTight_vs_muAvg", "Number of tight reco tracks with truth links versus #mu mean; <#mu>;<nTruthLinkedTracksTight>", nMu, minMu, maxMu);
>   >     p_nLowTTPTracksLoose_vs_muAvg = new TProfile(m_name+"nHighTTPTracksLoose_vs_muAvg", "Number of loose reco tracks with truth matching Probability > 0.5 versus #mu mean; <#mu>;<nHighTTPTracksLoose>", nMu, minMu, maxMu);
>   >     p_nLowTTPTracksTight_vs_muAvg = new TProfile(m_name+"nHighTTPTracksTight_vs_muAvg", "Number of tight reco tracks with truth matching Probability > 0.5 versus #mu mean; <#mu>;<nHighTTPTracksTight>", nMu, minMu, maxMu);
>   >     p_nZeroBarcodeTracks_vs_muAvg = new TProfile(m_name+"nZeroBarcodeTracks_vs_muAvg", "Number of reco tracks with a barcode of 0 versus #mu mean; <#mu>;<nZeroBarcodeTracks>", nMu, minMu, maxMu);
>   >     p_nBetweenBarcodeTracks_vs_muAvg = new TProfile(m_name+"nBetweenBarcodeTracks_vs_muAvg", "Number of reco tracks with a barcode between 0 and 200,000 versus #mu mean; <#mu>;<nBetweenBarcodeTracks>", nMu, minMu, maxMu);
>   >     p_n200e3PlusBarcodeTracks_vs_muAvg = new TProfile(m_name+"n200e3PlusBarcodeTracks_vs_muAvg", "Number of reco tracks with a barcode greater that 200,000 versus #mu mean; <#mu>;<n200e3PlusBarcodeTracks>", nMu, minMu, maxMu);
>   >     p_nOopsBarcodeTracks_vs_muAvg = new TProfile(m_name+"nOopsBarcodeTracks_vs_muAvg", "Number of reco tracks with a wierd barcodes versus #mu mean; <#mu>;<nOopsBarcodeTracks>", nMu, minMu, maxMu);
>   >     p_nSharedBarcodeTracks_vs_muAvg = new TProfile(m_name+"nSharedBarcodeTracks_vs_muAvg", "Number of reco tracks with shared barcodes versus #mu mean; <#mu>;<nSharedBarcodeTracks_vs_muAvg>", nMu, minMu, maxMu);
>   >     p_matchProbAvg_vs_muAvg = new TProfile(m_name+"matchProbAvg_vs_muAvg", "The average matching probability for a track versus #mu mean; <#mu>;<matchProbAvg_vs_muAvg>", nMu, minMu, maxMu);
>   >     p_matchProbAvgFake_vs_muAvg = new TProfile(m_name+"matchProbAvgFake_vs_muAvg", "The average matching probability for a fake track versus #mu mean; <#mu>;<matchProbAvgFake_vs_muAvg>", nMu, minMu, maxMu);
>   >     p_percTrackswoTLandwMP_vs_muAvg = new TProfile(m_name+"percTrackswoTLandwMP_vs_muAvg", "Number of truth-linked tracks with a matching probability versus #mu mean; <#mu>;<percTrackswoTLandwMP_vs_muAvg>", nMu, minMu, maxMu);

> ADD this into the ->Sumw2() (~L96)
>   >     p_nTruthLinkedTracksLoose_vs_muAvg  ->Sumw2();
>   >     p_nTruthLinkedTracksTight_vs_muAvg  ->Sumw2();
>   >     p_nLowTTPTracksLoose_vs_muAvg      ->Sumw2();
>   >     p_nLowTTPTracksTight_vs_muAvg      ->Sumw2();
>   >     p_nZeroBarcodeTracks_vs_muAvg       ->Sumw2();
>   >     p_nBetweenBarcodeTracks_vs_muAvg    ->Sumw2();
>   >     p_n200e3PlusBarcodeTracks_vs_muAvg  ->Sumw2();
>   >     p_nOopsBarcodeTracks_vs_muAvg       -> Sumw2();
>   >     p_nSharedBarcodeTracks_vs_muAvg     -> Sumw2();
>   >     p_matchProbAvg_vs_muAvg             -> Sumw2();
>   >     p_matchProbAvgFake_vs_muAvg         ->Sumw2();
>   >     p_percTrackswoTLandwMP_vs_muAvg     ->Sumw2();

> ADD this plots to the worker (~L149)
>   >     wk->addOutput(p_nTruthLinkedTracksLoose_vs_muAvg  );
>   >     wk->addOutput(p_nTruthLinkedTracksTight_vs_muAvg  );
>   >     wk->addOutput(p_nLowTTPTracksLoose_vs_muAvg      );
>   >     wk->addOutput(p_nLowTTPTracksTight_vs_muAvg      );
>   >     wk->addOutput(p_nZeroBarcodeTracks_vs_muAvg       );
>   >     wk->addOutput(p_nBetweenBarcodeTracks_vs_muAvg    );
>   >     wk->addOutput(p_n200e3PlusBarcodeTracks_vs_muAvg  );
>   >     wk->addOutput(p_nOopsBarcodeTracks_vs_muAvg       );
>   >     wk->addOutput(p_nSharedBarcodeTracks_vs_muAvg     );
>   >     wk->addOutput(p_matchProbAvg_vs_muAvg             );
>   >     wk->addOutput(p_matchProbAvgFake_vs_muAvg         );
>   >     wk->addOutput(p_percTrackswoTLandwMP_vs_muAvg     );

>   ADD counters to store number of truth-linked or barcode=0 tracks (~L177)
>   >     int nZeroBarcodeTracks     = 0;
>   >     int nBetweenBarcodeTracks  = 0;
>   >     int n2e5PlusBarcodeTracks  = 0;
>   >     int nOopsBarcodeTracks     = 0;
>   >     int nSharedBarcodeTracks   = 0;

> ADD Filling of relevant barcode counters (L221)
>   >     if (truth.barcode == 0 ){
>   >       ++nZeroBarcodeTracks;
>   >     } else if (truth.barcode > 0 && truth.barcode < 200e3){
>   >       ++nBetweenBarcodeTracks;
>   >     } else if (truth.barcode >= 200e3){
>   >       ++n2e5PlusBarcodeTracks;
>   >     } else {
>   >       ++nOopsBarcodeTracks;
>   >     }

> ADD barcode checking (~L232)
>   >     //Create a barcodeRecord vector and add all of the barcodes to it one at a time. At any time if the non-zero barcode you want to add already exists in the vector then that triggers a counter to flag an instance of a non-unique track.
>   >     if(!truth.barcode == 0){
>   >         if(std::find(barcodeRecord.begin(), barcodeRecord.end(), truth.barcode) != barcodeRecord.end()) {
>   >         if(std::find(sharedBarcodeRecord.begin(), sharedBarcodeRecord.end(), truth.barcode) != sharedBarcodeRecord.end()) {
>   >             nSharedBarcodeTracks++;
>   >             sharedBarcodeRecord.push_back(truth.barcode);
>   >         } else {
>   >             nSharedBarcodeTracks+=2;
>   >             sharedBarcodeRecord.push_back(truth.barcode);
>   >         }
>   >         } else {
>   >         barcodeRecord.push_back(truth.barcode);
>   >         }
>   >     }

> ADD variables to store truth link and match probability data from tracks (~L258)
>   >      int nHighTTPTracks               = 0;
>   >      int nTruthLinkedTracks           = 0;
>   >      int nNoTruthLink                 = 0;
>   >      int nNoTruthLinkHasMatchProb     = 0;
>   >      float MatchProbSum               = 0;
>   >      float matchProbSumFake           = 0;
>   >      float matchProbAverage           = 0;
>   >      float matchProbAverageFake       = 0;
>   >      float percTrackswoTLandwMP       = 0;

> ADD Truth link and selective truth matching probability filling (~L336)
>   >      if (trk.hasTruthLink){
>   >        ++nTruthLinkedTracks;
>   >        htruthMatchedTrackPt->Fill(pt, weight);
>   >        htruthMatchedTrackEta->Fill(trk.eta, weight);
>   >      } else {
>   >        ++nNoTruthLink;
>   >        if (trk.truthProb > 0.5) {++nNoTruthLinkHasMatchProb;}
>   >      }
>   >      if (trk.truthProb < 0.5) {
>   >        matchProbSumFake += trk.truthProb;
>   >      } else {
>   >        ++nHighTTPTracks;
>   >        matchProbSum += trk.truthProb;

> ADD Calculation of matchProbability average and tracks non fakes without truth match (~L354)
>   >     if (!nPreSelTracks==0){
>   >       matchProbAverage = matchProbSum/nPreSelTracks;
>   >       matchProbAverageFake = matchProbSumFake/nPreSelTracks;
>   >       if (!nNoTruthLink==0){
>   >         percTrackswoTLandwMP = nNoTruthLinkHasMatchProb/nNoTruthLink * 100;
>   >       } else {
>   >       percTrackswoTLandwMP = 0;
>   >       }
>   >       ...
>   >     }

>    ADD filling of the new plots (~L232)
>   >     p_nZeroBarcodeTracks_vs_muAvg->Fill(mu,nZeroBarcodeTracks,weight);
>   >     p_nBetweenBarcodeTracks_vs_muAvg->Fill(mu,nBetweenBarcodeTracks,weight);
>   >     p_n200e3PlusBarcodeTracks_vs_muAvg->Fill(mu,n2e5PlusBarcodeTracks,weight);
>   >     p_nOopsBarcodeTracks_vs_muAvg->Fill(mu,nOopsBarcodeTracks,weight);
>   >     p_nSharedBarcodeTracks_vs_muAvg->Fill(mu,nSharedBarcodeTracks,weight);

>   >     (~L362)
>   >     p_matchProbAvg_vs_muAvg->Fill(mu,matchProbAverage,weight);
>   >     p_matchProbAvgFake_vs_muAvg->Fill(mu,matchProbAverageFake,weight);
>   >     p_percTrackswoTLandwMP_vs_muAvg->Fill(mu,percTrackswoTLandwMP,weight);
>   >     }

>   >     if (Loose==true) (~L374)
>   >     {
>   >     p_nTruthLinkedTracksLoose_vs_muAvg->Fill(mu,nTruthLinkedTracks,weight);
>   >     p_nHighTTPTracksLoose_vs_muAvg->Fill(mu,nHighTTPTracks,weight);
>   >     } else if(Tight==true) {
>   >     p_nTruthLinkedTracksTight_vs_muAvg->Fill(mu,nTruthLinkedTracks,weight);
>   >     p_nHighTTPTracksTight_vs_muAvg->Fill(mu,nHighTTPTracks,weight);
>   >     }

> CHANGE pileup re-weighting file (~L555) if you are not just doing plots that run with mu.
>   >     CHECK( m_pileupReweightingTool->setProperty("UsePeriodConfig", "MC15") );

Now define your new plots in FakeRecommendationsPlots.h.
~~~
vim MyVertexAnalysis/Root/FakeRecommendationsPlots.h
~~~
> ADD some Kinematic Feature plots (~L38)
>   >     TH1F* htruthMatchedTrackPt               ; //!
>   >     TH1F* htruthMatchedTrackEta              ; //!

> ADD New TProfile Plots (~L53)
>   >     TProfile* p_nTruthLinkedTracksLoose_vs_muAvg ; //!
>   >     TProfile* p_nTruthLinkedTracksTight_vs_muAvg ; //!
>   >     TProfile* p_nLowTTPTracksLoose_vs_muAvg      ; //!
>   >     TProfile* p_nLowTTPTracksTight_vs_muAvg      ; //!
>   >     TProfile* p_nZeroBarcodeTracks_vs_muAvg      ; //!
>   >     TProfile* p_nBetweenBarcodeTracks_vs_muAvg   ; //!
>   >     TProfile* p_n200e3PlusBarcodeTracks_vs_muAvg ; //!
>   >     TProfile* p_nOopsBarcodeTracks_vs_muAvg      ; //!
>   >     TProfile* p_nSharedBarcodeTracks_vs_muAvg    ; //!
>   >     TProfile* p_matchProbAvg_vs_muAvg            ; //!
>   >     TProfile* p_matchProbAvgFake_vs_muAvg        ; //!
>   >     TProfile* p_percTrackswoTLandwMP_vs_muAvg    ; //!

Now we need to go and ensure the analysis code fills these points at the correct time and that only the plots we want are filled.
~~~
vim MyVertexAnalysis/Root/MyVertexAnalysis.cxx
~~~
> COMMENT OUT all other track folder selections that will not be used. (~L372)
>   >     FRP_Loose1GeV = new FakeRecommendationsPlots("Loose1GeVTracks/", wk());
>   >     FRP_Tight1GeV = new FakeRecommendationsPlots("Tight1GeVTracks/", wk());
> ADD another track selection folder (~L379)
>   >     FRP_AllTracks1GeV = new FakeRecommendationsPlots("All1GeVTracks/", wk());

> COMMENT OUT the filling all other track folder selections that will not be used. (~L1110)
>   >     FRP_Loose1GeV->FillPlots(*m_tracks,m_weight,m_muActualCorrectedScaled,false,false,true,false,1.);
>   >     FRP_Tight1GeV->FillPlots(*m_tracks,m_weight,m_muActualCorrectedScaled,false,false,false,true,1.);
> ADD another track selection folder (~L1109)
>   >     FRP_AllTracks1GeV->FillTruthPlots(*m_TruthTracks,m_weight,m_muActualCorrectedScaled,1.);

~~~
vim MyVertexAnalysis/MyVertexAnalysis/MyVertexAnalysis.h
~~~
> ADD (L193)
>   >     FakeRecommendationsPlots *FRP_AllTracks1GeV; //!

The next thing to do is to fetch the MCRaw file. These are big so will need a specialised place where they can go.
~~~
cd /eos/user/d/dspiteri/FakeStudy/
setupATLAS && lsetup rucio
voms-proxy-init -voms atlas
mkdir xAODMC16a && cd xAODMC16a
rucio download mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315
~~~
You can stop this download after it finished getting one file to have a look at the output. This will have at least 10,000 events you can run over.

The root files that come from the download all have root.1 when they have finished. This needs to be accounted for in the steering scripts that reads in the files.
~~~
vim MyVertexAnalysis/scripts/Run.py
~~~
> FIX the search directory lists
>   >     search_directories = [] (~L127)
>   >     search_directories.append(input_dir) (~138)

> CHANGE the pattern searching suffix
>   >     pattern = "*.root" -> "*.root.1" (~L137)

> COMMENT OUT the GRL (~L224)
>   >     alg.goodRunList = "$ROOTCOREBIN/data/MyVertexAnalysis/GRL/data17_13TeV.periodAllYear_DetStatus-v95-pro21-09_Unknown_PHYS_StandardGRL_All_Good_25ns_Triggerno17e33prim.xml"
> COMMENT IN a blank
>   >     alg.goodRunList = ""

Now you need to test that these new plots appear in the output and are filled correctly. Then they can go on to be prettied up.
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/build
setupATLAS
asetup AnalysisBase,21.2.30 && lsetup "ROOT 6.10.04"
cmake ../atlas-trackingCP-fakestudy/
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
cd ../atlas-trackingCP-fakestudy/
export ROOTCOREBIN=/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/RootCoreBin
rm -rf ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root
python MyVertexAnalysis/scripts/Run.py --submitDir ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root --input /eos/user/d/dspiteri/FakeStudy/xAODMC16a/mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315/AOD.11398262._000001.pool.root.1 | tee running.log

lsetup root
root -l ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root/hist-mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315.root
~~~
If this is working open up a seperate shell and run over the other MC sets to build up some statistics. The following script will run the previous python command for 9 other of the MC16 in the same folder, hadd the outputs and then delete the folders to save space. You can easily not do this step if you think you have enough statistics.
~~~
source runMCstats.sh
~~~

So after this point various plots could be requested to be added, such as checking the number of non-truth linked tracks that have a matching probability, or investigating the kinematic variables of tracks that contain a truth match. To add these histograms in you just repeat the steps written above. The exception to this is that for investigating the kinematic variables, instead of a profile fit, you probably want a TH1F.

> ADD Track Quality Histograms ~ L323
>   >     if (trk.hasTruthLink){ ++nTruthLinkedTracks;
>   >     htruthMatchedTrackPt->Fill(pt, weight);
>   >     htruthMatchedTrackEta->Fill(trk.eta, weight);
>   >     } else {
>   >     ++nNoTruthLink;
>   >     if (trk.truthProb) {++nNoTruthLinkHasMatchProb;}
>   >     }
>   >     matchProbSum += trk.truthProb;
>   >     }
>   >
>   >     }//Loop on tracks
>   >
>   >     if (!nPreSelTracks==0){
>   >       matchProbAverage = matchProbSum/nPreSelTracks;
>   >       if (!nNoTruthLink==0){
>   >         percTrackswoTLandwMP = nNoTruthLinkHasMatchProb/nNoTruthLink * 100;
>   >       } else {
>   >         percTrackswoTLandwMP = 0;
>   >       }
>   >       p_matchProbAvg_vs_muAvg->Fill(mu,matchProbAverage,weight);
>   >       p_percTrackswoTLandwMP_vs_muAvg->Fill(mu,percTrackswoTLandwMP,weight);
>   >     }


## Beautifying MC Plots.
For this we will want to create a new plotting macro along the style of NTracksVsMu_datamc_fakerate_v3.C
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/
vim NTracksVsMu_mc.cpp
~~~
See https://gitlab.cern.ch/Atlas-Inner-Tracking/atlas-trackingCP-fakestudy/tree/master/ZeroBias_FakeTracks

Now instead of running this through root, as we did for the previous iteration fo NTracksVsMu...C file, we can make an addition to the CMakeLists.txt file, and have this run like Recom_beatif.cpp
>   ADD executable functionality (~L231)
>   >     int main(int argc, char *argv[]){
>   >     std::cout<<argc<<std::endl;
>   >     for(int i=0; i<argc; ++i) {
>   >     std::cout << argv[i] << std::endl;
>   >     }
>   >     if(argc<2) {
>   >     std::cout << "Running with the default perameters" << std::endl;
>   >     NTracksVsMu_mc();
>   >     } else if (argc==4) {
>   >     NTracksVsMu_mc(argv[1],argv[2],argv[3]);
>   >     } else if (argc<6) {
>   >     return 1;
>   >     } else {
>   >     NTracksVsMu_mc(argv[1]);
>   >     }
>   >       return 0;
>   >     }
~~~
vim CMakeLists.txt
~~~
>   CHANGE ROOT package requirements (L5)
>   >     find_package( ROOT REQUIRED COMPONENTS Core Gpad Tree Hist RIO MathCore Graf Physics TreePlayer )

> ADD another compiling executable (~L11)
>   >     atlas_add_executable (NTracksVsMu_mc NTracksVsMu_mc.cpp
>   >     INCLUDE_DIRS ${ROOT_INCLUDE_DIRS}
>   >     LINK_LIBRARIES ${ROOT_LIBRARIES})

Now we need to rebuild the directory to get the new executable
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/
setupATLAS
asetup AnalysisBase,21.2.30 && lsetup "ROOT 6.10.04"
cd build
cmake ../atlas-trackingCP-fakestudy/
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
~~~
Now when you want to run the plot pretty-ing, (if you are happy with the defaults, which are also run by) you run the executable from anywhere.
~~~
cd ../atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/
mkdir Plots/_J_/
NTracksVsMu_mc
~~~
If you make further edits to the file you rebuild and rerun
~~~
cd ../../build
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
cd ../atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/
NTracksVsMu_mc
imgcat Plots/_J_/MPAFakeTracksVsMu_mc.png
imgcat Plots/_J_/TMP05TracksVsMu_mc.png
imgcat Plots/_J_/LinDevLooseTracksVsMu_mc.png
imgcat Plots/_J_/LinDevTightTracksVsMu_mc.png

imgcat Plots/_J_/BBTTracksVsMu_mc.png
imgcat Plots/_J_/HBTTracksVsMu_mc.png
imgcat Plots/_J_/0BTTracksVsMu_mc.png
imgcat Plots/_J_/SBTTracksVsMu_mc.png
imgcat Plots/_J_/NTLTracksVsMuLoose_withPad.png
imgcat Plots/_J_/NTLTracksVsMuTight_withPad.png
imgcat Plots/_J_/MPATracksVsMu_mc.png
imgcat Plots/_J_/pwoTLwMPTracksVsMu_mc.png
imgcat Plots/_J_/TMTPtTracksVsMu_mc.png
imgcat Plots/_J_/TMTEtaTracksVsMu_mc.png

~~~
If you don't want to run over the defaults, for the last command you run something like.
~~~
NTracksVsMu_mc "/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root/hist-mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315.root" "/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracksZeroBias_FakeTracks/Plots/_J_" "2017MC16a"

NTracksVsMu_mc "/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root/hist-mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315-2to10.root" "/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_J_/2-10/" "2017MC16a"
~~~

## Last Touches

So the last thing to do is to add more cuts to the barcode plots to see if checking that the truth particles are from stable objects turns the barcode plots into something useful. Also to run a comaprison between tracks that have a P_match < 0.5 and those who in addition to this also have a truth link. There exists in the CxAOD an integer that tells if the particle is stable or not. Check http://acode-browser1.usatlas.bnl.gov/lxr/source/athena/Event/xAOD/xAODTruth/xAODTruth/versions/TruthParticleAuxContainer_v1.h?v=21.0 for thetype of the variables you want to fetch.
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/
vim MyVertexAnalysis/Root/MyTrack.h
~~~
> ADD in integer to mytrack.h (~L15)
>   >     int status;

~~~
vim MyVertexAnalysis/Root/MyVertexAnalysis.cxx
~~~
> ADD barcode filling (~L881)
>   >     myTruthTrack.status = truthtrk->status();

~~~
vim MyVertexAnalysis/Root/FakeRecommendationsPlots.cxx
~~~
> ADD new plots (~L42)
>   >     p_nHighTTPTracksLoose_vs_muAvg = new TProfile(m_name+"nHighTTPTracksLoose_vs_muAvg", "Number of loose reco tracks with truth matching Probability > 0.5 versus #mu mean; <#mu>;<nHighTTPTracksLoose>", nMu, minMu, maxMu);
>   >     p_nHighTTPTracksTight_vs_muAvg = new TProfile(m_name+"nHighTTPTracksTight_vs_muAvg", "Number of tight reco tracks with truth matching Probability > 0.5 versus #mu mean; <#mu>;<nHighTTPTracksTight>", nMu, minMu, maxMu);
>   >     p_nHighTTPTrackswTLLoose_vs_muAvg = new TProfile(m_name+"nHighTTPTrackswTLLoose_vs_muAvg", "Number of loose reco tracks with TMP > 0.5 and a truth link versus #mu mean; <#mu>;<nHighTTPTrackswTLLoose>", nMu, minMu, maxMu);
>   >     p_nHighTTPTrackswTLTight_vs_muAvg = new TProfile(m_name+"nHighTTPTrackswTLTight_vs_muAvg", "Number of tight reco tracks with TMP > 0.5 and a truth link versus #mu mean; <#mu>;<nHighTTPTrackswTLTight>", nMu, minMu, maxMu);

>   ADD these new plots to the Sumw2() (~L105)
>   >     p_nHighTTPTracksLoose_vs_muAvg     ->Sumw2();
>   >     p_nHighTTPTracksTight_vs_muAvg     ->Sumw2();
>   >     p_nHighTTPTrackswTLLoose_vs_muAvg  ->Sumw2();
>   >     p_nHighTTPTrackswTLTight_vs_muAvg  ->Sumw2();

ADD these new plots to the worker (~162)
>   >     wk->addOutput(p_nHighTTPTracksLoose_vs_muAvg      );
>   >     wk->addOutput(p_nHighTTPTracksTight_vs_muAvg      );
>   >     wk->addOutput(p_nHighTTPTrackswTLLoose_vs_muAvg   );
>   >     wk->addOutput(p_nHighTTPTrackswTLTight_vs_muAvg   );


> ADD status check for barcode filling (~L238)
>   >     if (truth.status == 1 ){
>   >     [barcode counter fillling]
>   >     } (~L238)

> ADD counters for new plots. (~L289)
>   >     int nHighTTPTracks               = 0;
>   >     int nHighTTPTrackswTruthLink     = 0;

> ADD filling of counters for new plots (~L370 - inside else for trk.truthProb < 0.5)
>   >     ++nHighTTPTracks;
>   >     if (trk.hasTruthLink){++nHighTTPTrackswTruthLink;}

> ADD filling of histograms (~L404/L413)
>   >     p_nHighTTPTracksLoose_vs_muAvg->Fill(mu,nHighTTPTracks,weight);
>   >     p_nHighTTPTrackswTLLoose_vs_muAvg->Fill(mu,nHighTTPTrackswTruthLink,weight);

>   >     p_nHighTTPTracksTight_vs_muAvg->Fill(mu,nHighTTPTracks,weight);
>   >     p_nHighTTPTrackswTLTight_vs_muAvg->Fill(mu,nHighTTPTrackswTruthLink,weight);

~~~
vim MyVertexAnalysis/MyVertexAnalysis/FakeRecommendationsPlots.h
~~~
> ADD the new plots to the header file (~L57)
>   >     TProfile* p_nHighTTPTracksLoose_vs_muAvg     ; //!
>   >     TProfile* p_nHighTTPTracksTight_vs_muAvg     ; //!
>   >     TProfile* p_nHighTTPTrackswTLLoose_vs_muAvg  ; //!
>   >     TProfile* p_nHighTTPTrackswTLTight_vs_muAvg  ; //!

~~~
vim /ZeroBias_FakeTracks/NTracksVsMu_mc.cpp
~~~
> ADD Pulling in of new histograms (~L54)
>   >     TProfile* pHighTTPLoose     = (TProfile*) fMC->Get("Loose1GeVTracks/nHighTTPTracksLoose_vs_muAvg");
>   >     TProfile* pHighTTPwTLLoose  = (TProfile*) fMC->Get("Loose1GeVTracks/nHighTTPTrackswTLLoose_vs_muAvg");

> ADD plot prettying (~L110)
>   >      neatenTwoNTracksVsMuPlots_mc(pHighTTPLoose, pHighTTPwTLLoose, 250, "#LTN_{Tracks}#GT", kMagenta+2, kSpring+4, outdir, "Loose Tracks with P_{match} > 0.5 (A)", "A & has Truth Link", "LRTaLRTTL", false);


Now the CxAOD needs to be run again to produce the ntuples with the relevant barcode cuts, and then you run the plot neatening macro.

~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/build
setupATLAS
asetup AnalysisBase,21.2.30 && lsetup "root 6.10.04-x86_64-slc6-gcc62-opt"
cmake ../atlas-trackingCP-fakestudy/
make -j6
source x86_64-slc6-gcc62-opt/setup.sh
cd ../atlas-trackingCP-fakestudy/
export ROOTCOREBIN=/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/RootCoreBin
rm -rf ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root
python MyVertexAnalysis/scripts/Run.py --submitDir ZeroBias_FakeTracks/Plots/J-_-R21_mc16a-159000.ParGen_nu_E50.root --input /eos/user/d/dspiteri/FakeStudy/xAODMC16a/mc16_13TeV.159000.ParticleGenerator_nu_E50.merge.AOD.e3711_s3126_r9425_r9315/AOD.11398262._000001.pool.root.1 | tee running.log
cd ZeroBias_FakeTracks
NTracksVsMu_mc
imgcat Plots/_J_/BBTTracksVsMu_mc.png
imgcat Plots/_J_/HBTTracksVsMu_mc.png
imgcat Plots/_J_/SBTTracksVsMu_mc.png
imgcat Plots/_J_/LRTaLRTTLTracksVsMu_mc.png
imgcat Plots/_J_/LinDevTightTracksVsMu_mc.png
imgcat Plots/_J_/NTLTracksVsMuLoose_withPad.png
~~~
