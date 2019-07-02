# This will be aimed at my the second part of my qualification task which is to join the efforts surrounding the study of fake rates. This documentation is mainly focussed on adding d0 and z0 plots to the framework to try and understand fake tracks better.  

## "Fake Rate Investigations" ##
===============================================================================
Last Edited: 24-11-2017
-------------------------------------------------------------------------------
### I highly recommend you read Fake_Rate_Qualification_Task.md if you think some steps are skipped without explanation. This will hit the ground running so to speak. ###

# Investigating the Fake Rate 
### https://gitlab.cern.ch/dspiteri/atlas-trackingCP-fakestudy ###

Now that passable plots have been produced, you want the framework to create more plots upon running to investigate the fake tracks.
The following files are the ones of the most interest but to just add plots, then only FakeRecommendationsPlots.cxx and FakeRecommendationsPlots.h should need to be edited.

- MyVertexAnalysis/Root/MyVertexAnalysis.cxx
- MyVertexAnalysis/Root/FakeRecommendationsPlots.cxx
- MyVertexAnalysis/MyVertexAnalysis/MyVertexAnalysis.h
- MyVertexAnalysis/MyVertexAnalysis/FakeRecommendationsPlots.h


## Adding d0 plots to the framework - I

~~~  
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks  
setupATLAS && lsetup git
git fetch origin  
git checkout -b master_d0Plots origin/master --no-track   
~~~

### (Terminal 1) ###

~~~
vim MyVertexAnalysis/Root/MyVertexAnalysis.cxx  
~~~

This is the main code run for Run.py so it's important to look at this and see what is going on and what needs to be edited. 

>   CHANGE Initialisation Values  ~L133
>   > int nMu = 50 -> 80    
>   > int maxMu = 50 -> 80     
>   > int nVtx = 50 -> 80    
>   > int nMu = 49.5 -> 79.5    

You will want to create new TProfile histograms so search for "new TProfile". These should be being initially created before you load the ntuples, so in EL::StatusCode MyVertexAnalysis::histInitialize (). The easiest way to test the addition of plots is actually to add it to the FakeRecommendationsPlots class seen from L383, for which you will need to understand what is going on in the FillPlots function (~L1250). 

> void FakeRecommendationsPlots::FillPlots(const std::vector<MyTrack> &tracks, float weight, double mu, bool L2V, bool T2V, bool Loose, bool Tight, float pTcut){
> [...]
> }

After edits it will do to remove all extraneous commented code (if not done so already). 
 
### (Terminal 2) ###
~~~
vim MyVertexAnalysis/Root/FakeRecommendationsPlots.cxx  
~~~

> CHANGE Inialisation Values    
>   > int nMu = 60; -> 80; L26  
>   > int maxMu = 60; -> 80; L28    

> ADD new plots ~L35  
>   > p_nLooseTracks_vs_muAvg_innerd0 = new TProfile(m_name+"nLooseTracks_vs_muAvg_innerd0", "Number of loose tracks versus #mu (scaled) mean for d0 less than 0.5mm; <#mu(scaled)>;<nTracksLoose>", nMu, minMu, maxMu);  

>   > p_nTightTracks_vs_muAvg_innerd0 = new TProfile(m_name+"nTightTracks_vs_muAvg_innerd0", "Number of loose tracks versus #mu (scaled) mean for d0 less than 0.5mm; <#mu(scaled)>;<nTracksTight>", nMu, minMu, maxMu);  

> ADD these into the ->Sumw2() ~L77   
>   > p_nLooseTracks_vs_muAvg_innerd0 ->Sumw2();    
>   > p_nTightTracks_vs_muAvg_innerd0 ->Sumw2();   

> ADD these plots to the worker ~L110     
>   > wk->addOutput(p_nLooseTracks_vs_muAvg_innerd0);      
>   > wk->addOutput(p_nTightTracks_vs_muAvg_innerd0);   

> ADD the filling of the plots inside FillPlots ~L242    
>   > if (Loose==true)p_nLooseTracks_vs_muAvg_innerd0->Fill(mu,nPreSelInnerd0Tracks,weight);        
>   > else if(Tight==true)p_nTightTracks_vs_muAvg_innerd0->Fill(mu,nPreSelInnerd0Tracks,weight);      

> ADD details about the currently undefined nPreSelInnderd0Tracks  
>   > int nPreSelTracks            = 0; ~L178  
>   > int nPreSelInnerd0Tracks     = 0; ~L179  
>   > if (pt > 1 && abs(trk.eta) < 2.5)     
>   > {  
>   >   > ++nPreSelTracks;  
>   >   > if (trk.d0 < 0.5)  
>   >   > {  
>   >   >   > ++nPreSelInnerd0Tracks;

>   >   > }

>   > } ~L223  

Now these just have to be defined in the header file.

### (Terminal 3) ###
~~~
> vim MyVertexAnalysis/MyVertexAnalysis/FakeRecommendationsPlots.h  
~~~

> ADD new TProfile Plots ~L52  
>   > TProfile* p_nLooseTracks_vs_muAvg_innerd0; //!  
>   > TProfile* p_nTightTracks_vs_muAvg_innerd0; //!  
>  ~You do need the //! on the end.~ 

~~~
git status  
git commit -a -m "Editing code to allow the addition of d0 cuts on NTracks vs Mu plots"  
git push -u origin master
~~~

Now to test this.

### (Terminal 2) ###
~~~
setupATLAS
~~~
> rcSetup -u if done before just to clean up the area.

~~~
rcSetup Base,2.5.0    
rc find_packages && rc compile   
python MyVertexAnalysis/scripts/Run.py --submitDir Plots/H-_-R21_data-327342-327764-330470-337404-337491 --filelist 2017DataFileList.txt --filelistarea /eos/user/d/dspiteri/FakeStudy/Data2017/  
~~~

If you see Tight1GeVTracks/nTightTracks_vs_muAvg_innerd0 and it is filled then you're set. 

## Adding d0 plots to the framework - II
Now you have sucessfully added one set of plots. We now want to add the rest of the plots. I will start with 4 plots targetting different regions of d0.

- d0 < - 5 & d0 > 5 
- -5 < d0 < -2 & 2 < d0 < 5
- -2 < d0 < -0.5 & 0.5 < d0 < 2
- -0.5 < d0 < 0.5

### (Terminal 1) ###
~~~
vim MyVertexAnalysis/Root/FakeRecommendationsPlots.cxx 
~~~

> ADD new plots ~L35  
>   > p_nLooseTracks_vs_muAvg_outermostd0 = new TProfile(m_name+"nLooseTracks_vs_muAvg_outermostd0", "Number of loose tracks versus #mu (scaled) mean for abs(d0) larger than 5mm; <#mu(scaled)>;<nTracksLoose>", nMu, minMu, maxMu);
>   > p_nLooseTracks_vs_muAvg_larged0 = new TProfile(m_name+"nLooseTracks_vs_muAvg_larged0", "Number of loose tracks versus #mu (scaled) mean for abs(d0) between 2mm and 5mm; <#mu(scaled)>;<nTracksLoose>", nMu, minMu, maxMu);
>   > p_nLooseTracks_vs_muAvg_smalld0 = new TProfile(m_name+"nLooseTracks_vs_muAvg_smalld0", "Number of loose tracks versus #mu (scaled) mean for abs(d0) between 0.5mm and 2mm; <#mu(scaled)>;<nTracksLoose>", nMu, minMu, maxMu);

>   > p_nTightTracks_vs_muAvg_outermostd0 = new TProfile(m_name+"nTightTracks_vs_muAvg_outermostd0", "Number of tight tracks versus #mu (scaled) mean for abs(d0) larger than 5mm; <#mu(scaled)>;<nTracksTight>", nMu, minMu, maxMu);
>   > p_nTightTracks_vs_muAvg_larged0 = new TProfile(m_name+"nTightTracks_vs_muAvg_larged0", "Number of tight tracks versus #mu (scaled) mean for abs(d0) between 2mm and 5mm; <#mu(scaled)>;<nTracksTight>", nMu, minMu, maxMu);
>   > p_nTightTracks_vs_muAvg_smalld0 = new TProfile(m_name+"nTightTracks_vs_muAvg_smalld0", "Number of tight tracks versus #mu (scaled) mean for abs(d0) between 0.5mm and 2mm; <#mu(scaled)>;<nTracksTight>", nMu, minMu, maxMu);

> EDIT plots put in  
>   > p_nLooseTracks_vs_muAvg_innerd0 = new TProfile(m_name+"nTightTracks_vs_muAvg_innerd0", "Number of loose tracks versus #mu (scaled) mean for abs(d0) less than 0.5mm; <#mu(scaled)>;<nTracksLoose>", nMu, minMu, maxMu); 
>   > p_nTightTracks_vs_muAvg_innerd0 = new TProfile(m_name+"nTightTracks_vs_muAvg_innerd0", "Number of tight tracks versus #mu (scaled) mean for abs(d0) less than 0.5mm; <#mu(scaled)>;<nTracksTight>", nMu, minMu, maxMu); 

> ADD these into the ->Sumw2() ~L85  
>   > p_nLooseTracks_vs_muAvg_outermostd0 ->Sumw2();   
>   > p_nLooseTracks_vs_muAvg_larged0 ->Sumw2();    
>   > p_nLooseTracks_vs_muAvg_smalld0 ->Sumw2(); 

>   > p_nTightTracks_vs_muAvg_outermostd0 ->Sumw2();    
>   > p_nTightTracks_vs_muAvg_larged0 ->Sumw2();   
>   > p_nTightTracks_vs_muAvg_smalld0 ->Sumw2();    
    

> ADD these plots to the worker ~L125    
>   > wk->addOutput(p_nLooseTracks_vs_muAvg_outermostd0);      
>   > wk->addOutput(p_nLooseTracks_vs_muAvg_larged0);        
>   > wk->addOutput(p_nLooseTracks_vs_muAvg_smalld0);  
>   > wk->addOutput(p_nTightTracks_vs_muAvg_outermostd0); 
>   > wk->addOutput(p_nTightTracks_vs_muAvg_larged0);        
>   > wk->addOutput(p_nTightTracks_vs_muAvg_smalld0);

> ADD the filling of the plots inside FillPlots ~L273    
>   > if (Loose==true)  
>   > {  
>   > p_nLooseTracks_vs_muAvg_outermostd0->Fill(mu,nPreSelOutermostd0Tracks,weight);   
>   > p_nLooseTracks_vs_muAvg_larged0->Fill(mu,nPreSelLarged0Tracks,weight);   
>   > p_nLooseTracks_vs_muAvg_smalld0->Fill(mu,nPreSelSmalld0Tracks,weight);   
>   > p_nLooseTracks_vs_muAvg_innerd0->Fill(mu,nPreSelInnerd0Tracks,weight);  
>   > } else if(Tight==true) {  
>   > p_nTightTracks_vs_muAvg_outermostd0->Fill(mu,nPreSelOutermostd0Tracks,weight);   
>   > p_nTightTracks_vs_muAvg_larged0->Fill(mu,nPreSelLarged0Tracks,weight);   
>   > p_nTightTracks_vs_muAvg_smalld0->Fill(mu,nPreSelSmalld0Tracks,weight);   
>   > p_nTightTracks_vs_muAvg_innerd0->Fill(mu,nPreSelInnerd0Tracks,weight);  
>   > }  

> ADD details about the currently undefined nPreSel* 
>   > int nPreSelOutermostd0Tracks     = 0; ~L200  
>   > int nPreSelLarged0Tracks         = 0; ~L201  
>   > int nPreSelSmalld0Tracks         = 0; ~L202    
>   > if (pt > 1 && abs(trk.eta) < 2.5)     
>   > {  
>   > ++nPreSelTracks;  
>   > if (abs(trk.d0) > 5)  ++nPreSelOutermostd0Tracks;  
>   > if (abs(trk.d0) > 2 && abs(trk.d0) < 5)  ++nPreSelLarged0Tracks;    
>   > if (abs(trk.d0) > 0.5 && abs(trk.d0) < 2)  ++nPreSelSmalld0Tracks;  
>   > if (abs(trk.d0) < 0.5) ++nPreSelInnerd0Tracks;               
>   > } ~L259    

### (Terminal 2) ###
~~~
vim MyVertexAnalysis/MyVertexAnalysis/FakeRecommendationsPlots.h  
~~~

> ADD new TProfile Plots ~L53   
>   > TProfile* p_nLooseTracks_vs_muAvg_outermostd0; //!  
>   > TProfile* p_nLooseTracks_vs_muAvg_larged0;     //!  
>   > TProfile* p_nLooseTracks_vs_muAvg_smalld0;     //!  
>   > TProfile* p_nTightTracks_vs_muAvg_outermostd0; //!  
>   > TProfile* p_nTightTracks_vs_muAvg_larged0;     //!  
>   > TProfile* p_nTightTracks_vs_muAvg_smalld0;     //!  
>  You do need the //! on the end.  

### (Terminal 3) ###
~~~
setupATLAS
~~~
> (rcSetup -u)  

~~~
rcSetup Base,2.5.0      
rc find_packages && rc compile  
rm -rf Plots/H-_-R21_data-327342-327764-330470-337404-337491     
python MyVertexAnalysis/scripts/Run.py --submitDir   Plots/H-_-R21_data-327342-327764-330470-337404-337491-339849 --filelist 2017DataFileList.txt --filelistarea /eos/user/d/dspiteri/FakeStudy/Data2017/  
~~~

If this works now produce MC16c in the framework as well.

~~~
rc find_packages && rc compile  
rc clean
rm -rf Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root  
python MyVertexAnalysis/scripts/Run.py --submitDir Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root --filelist MC16cFileList.txt --filelistarea /eos/user/d/dspiteri/FakeStudy/MC16/
~~~

If at this step the package isn't loading the libraries, then rc clean has probably deleted them. Delete the RootCoreBin and try again without the rc clean command.
~~~
rm -rf RootCoreBin/   
rcSetup -u     
rcSetup Base,2.5.0  
rc find_packages && rc compile  
~~~

Now save your work. 

~~~
git status  
git commit -a -m "Code edits putting the four d0 slices on NTracks vs Mu plots"    
git push -u origin master_d0Plots  
~~~

## Beautifying d0 plots made in the framework  
Need to edit the NTracksvsMu_data_mc_fakerate_v3.C file
~~~  
> cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/  
> vim NTracksVsMu_datamc_fakerate_v3.C  
~~~

> ADD fetching of newly created TProfile Plots ~L124  
>   > TProfile* pLooseOuterd0Data = (TProfile*) fData->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_outermostd0");  
>   > TProfile* pLooseLarged0Data = (TProfile*) fData->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_larged0");  
>   > TProfile* pLooseSmalld0Data = (TProfile*) fData->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_smalld0");  
>   > TProfile* pLooseInnerd0Data = (TProfile*) fData->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_innerd0");  
>   > TProfile* pLooseOuterd0MC = (TProfile*) fMC->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_outermostd0");  
>   > TProfile* pLooseLarged0MC = (TProfile*) fMC->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_larged0");  
>   > TProfile* pLooseSmalld0MC = (TProfile*) fMC->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_smalld0");  
>   > TProfile* pLooseInnerd0MC = (TProfile*) fMC->Get("Loose1GeVTracks/nLooseTracks_vs_muAvg_innerd0");  
>   >         
>   > TProfile* pTightOuterd0Data = (TProfile*) fData->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_outermostd0");  
>   > TProfile* pTightLarged0Data = (TProfile*) fData->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_larged0");  
>   > TProfile* pTightSmalld0Data = (TProfile*) fData->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_smalld0");  
>   > TProfile* pTightInnerd0Data = (TProfile*) fData->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_innerd0");  
>   > TProfile* pTightOuterd0MC = (TProfile*) fMC->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_outermostd0");  
>   > TProfile* pTightLarged0MC = (TProfile*) fMC->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_larged0");  
>   > TProfile* pTightSmalld0MC = (TProfile*) fMC->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_smalld0");  
>   > TProfile* pTightInnerd0MC = (TProfile*) fMC->Get("Tight1GeVTracks/p_nTightTracks_vs_muAvg_innerd0");  

> ADD title data to the isDataData==0 loop ~L199  
>   > pLooseOuterd0Data->SetTitle("Loose, Data Rel21 |d0|>5mm");    
>   > pLooseLarged0Data->SetTitle("Loose, Data Rel21 2mm<|d0|<5mm");    
>   > pLooseSmalld0Data->SetTitle("Loose, Data Rel21 0.5mm<|d0|<2mm");    
>   > pLooseInnerd0Data->SetTitle("Loose, Data Rel21 |d0|< 0.5mm");    
>   > pLooseOuterd0MC->SetTitle("Loose, MC16 simulations |d0|>5mm");    
>   > pLooseLarged0MC->SetTitle("Loose, MC16 simulations 2mm<|d0|<5mm");    
>   > pLooseSmalld0MC->SetTitle("Loose, MC16 simulations 0.5mm<|d0|<2mm");    
>   > pLooseInnerd0MC->SetTitle("Loose, MC16 simulations |d0|< 0.5mm");    
>   > 
>   > pTightOuterd0Data->SetTitle("Tight, Data Rel21 |d0|>5mm");    
>   > pTightLarged0Data->SetTitle("Tight, Data Rel21 2mm<|d0|<5mm");    
>   > pTightSmalld0Data->SetTitle("Tight, Data Rel21 0.5mm<|d0|<2mm");    
>   > pTightInnerd0Data->SetTitle("Tight, Data Rel21 |d0|< 0.5mm");    
>   > pTightOuterd0MC->SetTitle("Tight, MC16 simulations |d0|>5mm");    
>   > pTightLarged0MC->SetTitle("Tight, MC16 simulations 2mm<|d0|<5mm");    
>   > pTightSmalld0MC->SetTitle("Tight, MC16 simulations 0.5mm<|d0|<2mm");    
>   > pTightInnerd0MC->SetTitle("Tight, MC16 simulations |d0|< 0.5mm");    

> ADD histogram decorations  
>   > pLooseOuterd0Data->GetXaxis()->SetTitle("mu");  
>   > pLooseOuterd0Data->GetYaxis()->SetTitle("#LTN_{Tracks}#GT");  
>   > pLooseOuterd0Data->GetYaxis()->SetNdivisions(508);  
>   > pLooseLarged0Data->GetXaxis()->SetTitle("mu");  
>   > pLooseLarged0Data->GetYaxis()->SetTitle("#LTN_{Tracks}#GT");  
>   > pLooseLarged0Data->GetYaxis()->SetNdivisions(508);  
>   > pLooseSmalld0Data->GetXaxis()->SetTitle("mu");  
>   > pLooseSmalld0Data->GetYaxis()->SetTitle("#LTN_{Tracks}#GT");  
>   > pLooseSmalld0Data->GetYaxis()->SetNdivisions(508);  
>   > pLooseInnerd0Data->GetXaxis()->SetTitle("mu");  
>   > pLooseInnerd0Data->GetYaxis()->SetTitle("#LTN_{Tracks}#GT");  
>   > pLooseInnerd0Data->GetYaxis()->SetNdivisions(508);  

>   > pLooseOuterd0Data->SetMarkerStyle(kFullCircle);  
>   > pLooseLarged0Data->SetMarkerStyle(kFullCircle);  
>   > pLooseSmalld0Data->SetMarkerStyle(kFullCircle);  
>   > pLooseInnerd0Data->SetMarkerStyle(kFullCircle);  
>   > pTightOuterd0Data->SetMarkerStyle(kFullSquare);  
>   > pTightLarged0Data->SetMarkerStyle(kFullSquare);  
>   > pTightSmalld0Data->SetMarkerStyle(kFullSquare);  
>   > pTightInnerd0Data->SetMarkerStyle(kFullSquare);  
>   > pLooseOuterd0MC->SetMarkerStyle(kOpenCircle);  
>   > pLooseLarged0MC->SetMarkerStyle(kOpenCircle);  
>   > pLooseSmalld0MC->SetMarkerStyle(kOpenCircle);  
>   > pLooseInnerd0MC->SetMarkerStyle(kOpenCircle);  
>   > pTightOuterd0MC->SetMarkerStyle(kOpenSquare);  
>   > pTightLarged0MC->SetMarkerStyle(kOpenSquare);  
>   > pTightSmalld0MC->SetMarkerStyle(kOpenSquare);  
>   > pTightInnerd0MC->SetMarkerStyle(kOpenSquare);  

>   > pLooseOuterd0Data->SetMarkerColor(kBlack);  
>   > pLooseLarged0Data->SetMarkerColor(kBlack);  
>   > pLooseSmalld0Data->SetMarkerColor(kBlack);  
>   > pLooseInnerd0Data->SetMarkerColor(kBlack);  
>   > pTightOuterd0Data->SetMarkerColor(kRed);  
>   > pTightLarged0Data->SetMarkerColor(kRed);  
>   > pTightSmalld0Data->SetMarkerColor(kRed);  
>   > pTightInnerd0Data->SetMarkerColor(kRed);  
>   > pLooseOuterd0MC->SetMarkerColor(kBlack);  
>   > pLooseLarged0MC->SetMarkerColor(kBlack);  
>   > pLooseSmalld0MC->SetMarkerColor(kBlack);  
>   > pLooseInnerd0MC->SetMarkerColor(kBlack);  
>   > pTightOuterd0MC->SetMarkerColor(kRed);  
>   > pTightLarged0MC->SetMarkerColor(kRed);  
>   > pTightSmalld0MC->SetMarkerColor(kRed);  
>   > pTightInnerd0MC->SetMarkerColor(kRed);  

> ADD Fits to plots ~L329
>   > TF1* fitLooseOuterd0Data = new TF1("fitLooseData", "x*[0]");  
>   > TF1* fitLooseLarged0Data = new TF1("fitLooseData", "x*[0]");  
>   > TF1* fitLooseSmalld0Data = new TF1("fitLooseData", "x*[0]");  
>   > TF1* fitLooseInnerd0Data = new TF1("fitLooseData", "x*[0]");  
>   > TF1* fitLooseOuterd0MC   = new TF1("fitLooseMC", "x*[0]");  
>   > TF1* fitLooseLarged0MC   = new TF1("fitLooseMC", "x*[0]");  
>   > TF1* fitLooseSmalld0MC   = new TF1("fitLooseMC", "x*[0]");  
>   > TF1* fitLooseInnerd0MC   = new TF1("fitLooseMC", "x*[0]");  
        
>   > TF1* fitTightOuterd0Data = new TF1("fitTightData", "x*[0]");  
>   > TF1* fitTightLarged0Data = new TF1("fitTightData", "x*[0]");  
>   > TF1* fitTightSmalld0Data = new TF1("fitTightData", "x*[0]");  
>   > TF1* fitTightInnerd0Data = new TF1("fitTightData", "x*[0]");  
>   > TF1* fitTightOuterd0MC   = new TF1("fitTightMC", "x*[0]");  
>   > TF1* fitTightLarged0MC   = new TF1("fitTightMC", "x*[0]");  
>   > TF1* fitTightSmalld0MC   = new TF1("fitTightMC", "x*[0]");  
>   > TF1* fitTightInnerd0MC   = new TF1("fitTightMC", "x*[0]");  

> ADD a set of lines that create each plot. For the love of all that is good
ak this a function ~L1029 (end of document)
>   >  void d0NTracksVsMuPlot_datamc(TProfile* LooseDatad0, TProfile* LooseMCd0, TProfile* TightDatad0, TProfile* TightMCd0, TF1* fitTightDatad0, TF1* fitLooseDatad0, int isDataData, float bottomFraction, TString outdir, TString d0RangeType){
>   >  
>   >  float bottomScale = 1./bottomFraction;  
>   >  float topScale = 1./(1.-bottomFraction);  
>   >  
>   >  gStyle->SetMarkerSize(1.1);  
>   >  TProfile::Approximate(true);  
>   >
>   >  TCanvas* can = new TCanvas("can", "can");  
>   >  TPad* top = new TPad("top", "top", 0, bottomFraction, 1, 1);  
>   >  TPad* bot = new TPad("bot", "bot", 0, 0, 1, bottomFraction);  
>   >  top->Draw();  
>   >  top->SetBottomMargin(0);  
>   >  top->SetTopMargin(gStyle->GetPadTopMargin()*topScale);  
>   >  bot->Draw();  
>   >  bot->SetTopMargin(0);  
>   >  bot->SetBottomMargin(bottomFraction);  
>   >  top->cd();  
>   >   
>   >  scaleHistogram(LooseDatad0, 1.0, topScale);  
>   >  LooseDatad0->GetXaxis()->SetRangeUser(5.5, 80);  
>   >  LooseDatad0->GetYaxis()->SetRangeUser(0.001, 380);  
>   >  LooseDatad0->SetBinContent(1,0);  
>   >  TightDatad0->SetBinContent(1,0);  
>   >  LooseDatad0->SetStats(false); //Removes the stat box on the top plot  
>   >
>   >  LooseDatad0->Draw("same");  
>   >  LooseMCd0->Draw("same");  
>   >  TightDatad0->Draw("same");  
>   >  TightMCd0->Draw("same");  
>   >
>   >
>   >  double markHeight = gStyle->GetLabelSize("x")*0.8;  
>   >  std::cout<<"LEGEND NTRACKS"<<std::endl;  
>   >  buildLegend(top, 1.0, topScale, true, topScale*(0.06+markHeight));  
>   >
>   >  fitLooseDatad0->Draw("same");  
>   >  fitTightDatad0->SetLineColor(kMagenta);  
>   >  fitTightDatad0->Draw("same");  
>   >
>   >
>   >  top->RedrawAxis();  
>   >  bot->cd();  
>   >  bot->SetTopMargin(0.025);  
>   >
>   >  TH1* ratioLoose = (TH1*) LooseDatad0->Clone("ratioLoose");  
>   >  ratioLoose->SetTitle(" ");  
>   >  if(isDataData==2 || isDataData==3)   ratioLoose->GetYaxis()->SetTitle("(Data15+Data16)/Data17");  
>   >  else ratioLoose->GetYaxis()->SetTitle("Data/MC");  
>   >  ratioLoose->GetYaxis()->SetRangeUser(0.9, 1.1);  
>   >  ratioLoose->GetYaxis()->SetLabelSize(0.4);  
>   >  ratioLoose->Divide(LooseMCd0);  
>   >  ratioLoose->GetYaxis()->SetNdivisions(504);  
>   >  ratioLoose->GetYaxis()->SetDecimals(true);  
>   >  // ratioLoose->Draw("axis");  
>   >  scaleHistogram(ratioLoose, 1.0, bottomScale);  
>   >  ratioLoose->Draw("hist PX0");  
>   >  ratioLoose->SetStats(false); // To remove the Stat box on the bottom panel.  
>   >
>   >  TH1* ratioTight = (TH1*) TightDatad0->Clone("ratioTight");  
>   >  ratioTight->SetTitle("Tight");  
>   >  ratioTight->Divide(TightMCd0);  
>   >  ratioTight->DrawCopy("hist PX0 same");  
>   >  ratioTight->SetStats(false); // To remove the second instance of the Stat box on the bottom panel.  
>   >  //buildLegend(bot, 1.0, bottomScale, false);  
>   >
>   >  TLine* line = new TLine();  
>   >  line->SetLineStyle(kDashed);  
>   >  line->DrawLine(5.5, 1, 60, 1);  
>   >
>   >  bot->RedrawAxis();  
>   >  can->cd();  
>   >  TLatex l; //l.SetTextAlign(12); l.SetTextSize(tsize);  
>   >  // l.SetTextSize(gStyle->GetTitleFontSize());  
>   >  l.SetTextSize(markHeight);  
>   >  l.SetNDC();  
>   >  // l.SetTextFont(72);  
>   >  l.SetTextAlign(11); // left top  
>   >  l.DrawLatexNDC(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1, "#bf{#it{ATLAS}} Internal, #sqrt{s} = 13 TeV");
>   >  // l.DrawLatexNDC(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-(0.05*1.1+0.04),"p_{T}>500 MeV, |#eta|<2.5");  
>   >
>   >  can->SaveAs(outdir+"/NTracksVsMu_datamc_"+d0RangeType+".pdf");  
>   >  can->SaveAs(outdir+"/NTracksVsMu_datamc_"+d0RangeType+".png");  
>   >  can->SaveAs(outdir+"/NTracksVsMu_datamc_"+d0RangeType+".eps");    
>   >  can->SaveAs(outdir+"/NTracksVsMu_datamc_"+d0RangeType+".C");   
>   >  }   

> ADD a declaration of created function ~L16    
void d0NTracksVsMuPlot_datamc(TProfile* LooseDatad0, TProfile* LooseMCd0, TProfile* TightDatad0, TProfile* TightMCd0, TF1* fitTightDatad0, TF1* fitLooseDatad0, int isDataData = 0, float bottomFraction = 0.4, TString outdir = "test", TString d0RangeType = "");   

> ADD lines to create the plots ~L1003  
>   > TString d0range1 = "Outermost";  
>   > TString d0range2 = "Large";  
>   > TString d0range3 = "Small";  
>   > TString d0range4 = "Inner";  
>   > d0NTracksVsMuPlot_datamc(pLooseOuterd0Data, pLooseOuterd0MC, pTightOuterd0Data, pTightOuterd0MC, fitTightOuterd0Data, fitLooseOuterd0Data, 0.4, outdir, d0range1);  
>   > d0NTracksVsMuPlot_datamc(pLooseLarged0Data, pLooseLarged0MC, pTightLarged0Data, pTightLarged0MC, fitTightLarged0Data, fitLooseLarged0Data, 0.4, outdir, d0range2);  
>   > d0NTracksVsMuPlot_datamc(pLooseSmalld0Data, pLooseSmalld0MC, pTightSmalld0Data, pTightSmalld0MC, fitTightSmalld0Data, fitLooseSmalld0Data, 0.4, outdir, d0range3);  
>   > d0NTracksVsMuPlot_datamc(pLooseInnerd0Data, pLooseInnerd0MC, pTightInnerd0Data, pTightInnerd0MC, fitTightInnerd0Data, fitLooseInnerd0Data, 0.4, outdir, d0range4);  

Save the file, push to git and re-run until the plots you obtain look sensible.
~~~
> setupATLAS && lsetup git     
> git status  
> git add [untracked files]   
> git commit -a -m "Code edits making the pretty end product four d0 slice plots"    
> git push -u origin master_d0Plots     

> cd ZeroBias_FakeTracks/   
> setupATLAS && lsetup root    
> mkdir Plots/_F-H_   
> root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/H-_-R21_data-327342-327764-330470-337404-337491-339849/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/F-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracksZeroBias_FakeTracks/Plots/_F-H_","2017Data2vMC16c",0)'   
~~~

Or if you were trying to run from the test 
~~~ 
> mkdir Plots/_H-I_   
> 
root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/H-_-R21_data-327342-327764-330470-337404-337491-339849/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/I-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_H-I_","2017Data2vMC16c",0)'    
~~~
Create a topic branch to store further changes.
~~~
> git pull
> git checkout -b master_d0Plots master --no-track
~~~

## Public Plots  
From now on, to simply get the old plots without the edits quickly. You want NTracksVsMu_datamc_fakerate_v2.C. Remember it is unedited, so the axis ranges will all have to be extended.  

I have also used v2 to produce plots for circulation. Hence there are further edits to this plotting macro. 
~~~  
vim ../NTracksVsMu_datamc_fakerate_v2.C  
~~~

> ADD Style information  L1  
>   > #include "MyVertexAnalysis/macros/AtlasLabels.C"  
>   > #include "MyVertexAnalysis/macros/AtlasStyle.C"  
>   > #include "MyVertexAnalysis/macros/AtlasUtils.C"  
>   > SetAtlasStyle(); L86   

> CHANGE run range of X axis    
>   > 5.5 -> 8.5 ~L432, L510, L580, L618, L691, L807  

> CHANGE  
>   > leg -> auto leg ~L591  

> FOR NTracksVsMu_data plots       
> CHANGE run range of Y axis     
>   > 250 -> 380 ~L560    

> CHANGE the size of the axis labels (and the labels themselves) ~L564     
>   > pLooseData->GetXaxis()->SetTitle("#LT#mu#GT_{bunch}");           

> ADD in changes to fit styles. ~L571 
>   > fitTightData->SetMarkerColor(kRed);
>   > fitTightData->SetLineColor(kRed); 

> ADD mu smearing ~L569  
>   > TGraphAsymmErrors* muLowerBoundLoose = new TGraphAsymmErrors();     
>   > TGraphAsymmErrors* muUpperBoundLoose = new TGraphAsymmErrors();   
>   > TGraphAsymmErrors* muLowerBoundTight = new TGraphAsymmErrors();   
>   > TGraphAsymmErrors* muUpperBoundTight = new TGraphAsymmErrors();   
>    
>   > muLowerBoundLoose->SetLineStyle(8);    
>   > muLowerBoundLoose->SetFillColor(kWhite);      
>   > muLowerBoundLoose->SetMarkerStyle(6);      
>   > muUpperBoundLoose->SetLineStyle(8);  
>   > muLowerBoundTight->SetLineStyle(8);  
>   > muLowerBoundTight->SetFillColor(kWhite);     
>   > muLowerBoundTight->SetLineColor(kRed);   
>   > muLowerBoundTight->SetMarkerColor(kRed);
>   > muLowerBoundTight->SetMarkerStyle(6);    
>   > muUpperBoundTight->SetLineStyle(8);   
>   > muUpperBoundTight->SetLineColor(kRed);     

>   > int counter=0;    
>   > for(int i=0; i<pLooseData->GetNbinsX()+1; i++ )   
>   > {    
>   >   > if (pLooseData->GetBinContent(i)==0) continue;    
>   >   > double x1 = pLooseData->GetBinCenter(i);  
>   >   > double x2 = pTightData->GetBinCenter(i);        
>   >   > double xerr = x1*0.04;   
>   >   > double y1_i = pLooseData->GetBinContent(i);  
>   >   > double y2_i = pTightData->GetBinContent(i);    
>   >   > muLowerBoundLoose->SetPoint(counter,x1-xerr,y1_i);    
>   >   > muUpperBoundLoose->SetPoint(counter,x1+xerr,y1_i);   
>   >   > muLowerBoundTight->SetPoint(counter,x2-xerr,y2_i);    
>   >   > muUpperBoundTight->SetPoint(counter,x2+xerr,y2_i);     
>   >   > counter++;

>   > }   

>   > muLowerBoundLoose->Draw("c same");  
>   > muUpperBoundLoose->Draw("c same");  
>   > muLowerBoundTight->Draw("c same");  
>   > muUpperBoundTight->Draw("c same");  


> CHANGE the way the legend is built ~L600  
>   > auto leg = buildLegend(can, 1.0, 1.0 true, 0.06+markheight); -> auto legend = TLegend(0.17, 0.80, 0.53, 0.58);     
>   > legend.SetFillStyle(1001);     
>   > legend.SetBorderSize(0);     
>   > legend.SetTextSize(0.033);   
>   > legend.SetTextFont(42);  
>   > legend.AddEntry(pLooseData,"Loose Selection, with #pm 4% syst on #LT#mu#GT_{bunch}", "p");  
>   > legend.AddEntry(fitLooseData, "Linear fit on low #LT#mu#GT_{bunch}", "l");  
>   > legend.AddEntry(pTightData,"TightPrimary Selection, with #pm 4% syst on #LT#mu#GT_{bunch}", "p");  
>   > legend.AddEntry(fitTightData, "Linear fit on low #LT#mu#GT_{bunch}", "l");  
>   > legend.Draw();  

> ADD Function to generate Dashed lines ~L69  
>   > void addDashedLines(Double_t x,Double_t y,Int_t colour, double lineStyle, double width){  
>   > TLine mline;  
>   > mline.SetLineWidth(2);  
>   > mline.SetLineColor(colour);  
>   > mline.SetLineStyle(lineStyle);  
>   > mline.DrawLineNDC(x-0.04,y,x-0.01,y);  
>   > }  

> ADD Dashed lines to represent uncertainties ~L629 
>   > addDashedLines(0.24, 0.790, 1, 8, 1);  
>   > addDashedLines(0.24, 0.759, 1, 8, 1);  
>   > addDashedLines(0.24, 0.682, 2, 8, 1);  
>   > addDashedLines(0.24, 0.650, 2, 8, 1);  

> ADD Labels ~L618  
>   > gStyle->SetTextSize(0.06);  
>   > ATLASLabel(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.01, "   Preliminary");  
>   > gStyle->SetTextSize(0.04);  
>   > myText(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.06,1, "#it{#sqrt{s}} = 13 TeV, Data 2017");  
>   > gStyle->SetTextSize(0.03);  
>   > myText(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.10,1, "p_{T} >1GeV & |#eta| < 2.5 Tracks");

For the public plot, there is discussion as to whether the run 339849 is strictly comparable to the other runs in the dataset due to special conditions so I shall revert to G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root for the data ntuple input. 
~~~ 
> root -b -l -q  '../NTracksVsMu_datamc_fakerate_v2.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/I-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_G-I_","2017Data2vMC16c",0)'
~~~
Look at the plots and if they are up to standard push them to a public webpage. If not, edit the plotting macro repeat the command below until you are satisfied with the plots.
~~~
imgcat Plots/_G-I_/NTracksVsMu_data.pdf  
cp -r Plots/_G-I_/ ~/www/FakeratePlots/     
cd ~/www/FakeratePlots/   
mv _G-I_/* FakeStudyPublicPlots/ && rm -rf _G-I_    
git add [all the new files]    
git commit -a -m "Package State upon completion of public plot NtrackVsMu_data"     
git push -u origin master_d0Plots    
~~~

Once you are happy with the public plots and the d0plots, you can merge the master_d0Plots branch with the master branch (best to do this in the online interface on gitLab), and then delete the branch locally when you have resolved all the differences.
~~~    
git branch -d master_d0Plots 
~~~

## Beautifying d0 plots Part II 
When the public plots are done port over the changes made in here to the plotting function NTracksVsMu_datamc_fakerate_v3.C

Actually in doing so I added a bool ismuune (initialised as false) as a flag to see if the user wants uncertainties on their plots, a float muunc (initialised as 0.04) to store the level of uncertainty requires and a int Yaxis (initialised as 380) to store the maximum Y-axis wanted.
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks
setupATLAS && lsetup root
rcSetup Base,2.5.0  
rc find_packages && rc compile 
rm -rf Plots/_G-I_/*
root -b -l -q '../NTracksVsMu_datamc_fakerate_v3.C(1,"/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/G-_-R21_data-327342-327764-330470-337404-337491/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/I-_-R21_mc16c-159000.ParGen_nu_E50.root/hist-sample.root","/afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks/Plots/_G-I_","2017Data2vMC16c",0)'
~~~

## Additional d0 plots
The recommendations for the fake rate MC mis-modelling are generated from 'fakerate' plots which are based on the NTracks vs Mu plots. These should also need to be created for the d0 plots.
~~~
cd /afs/cern.ch/work/d/dspiteri/PhysVal/FakeStudy/atlas-trackingCP-fakestudy/ZeroBias_FakeTracks
vim ../NTracksVsMu_datamc_fakerate_v3.C
~~~
> ADD fakerate extension to d0NTracksVsMu_datamc function
>   >     //-------------------------------------------------------------------FAKERATEPLOTS
>   >     //------------------------------------------------------------------
>   >     //------------------------------------------------------------------
>   >     TCanvas* can2 = new TCanvas("can2");
>   >     top = new TPad("top", "top", 0, bottomFraction, 1, 1);
>   >     bot = new TPad("bot", "bot", 0, 0, 1, bottomFraction);
>   >   
>   >     auto fkrlegend = TLegend(0.17, 0.65, 0.56, 0.40); //with Pad
>   >     auto fkrlegend = TLegend(0.17, 0.77, 0.52, 0.58); //without Pad
>   >     fkrlegend.SetFillStyle(1001);
>   >     fkrlegend.SetBorderSize(0);
>   >     fkrlegend.SetTextFont(42);
>   >    
>   >     if(withPad){
>   >     top->Draw();
>   >     top->SetBottomMargin(0);
>   >     top->SetTopMargin(gStyle->GetPadTopMargin()*topScale);
>   >     bot->Draw();
>   >     bot->SetTopMargin(0);
>   >     bot->SetBottomMargin(0.4);
>   >     top->cd();
>   >     top->SetGridy(true);
>   >   
>   >     scaleHistogram(hLooseDataFake, 1, topScale);
>   >     hLooseDataFake->GetXaxis()->SetRangeUser(8, 60);
>   >     hLooseDataFake->SetMinimum(-0.06);
>   >     hLooseDataFake->SetMaximum(0.30);
>   >     hLooseDataFake->GetYaxis()->SetLabelSize(0.047);
>   >     hLooseDataFake->GetYaxis()->SetTitleSize(0.047);
>   >     hLooseDataFake->GetXaxis()->SetTitleSize(0.047);
>   >     hLooseDataFake->GetYaxis()->SetTitle("Linear Deviation");
>   >     hLooseDataFake->Draw();
>   >   
>   >     fkrlegend.SetTextSize(0.048);
>   >     }else{
>   >     scaleHistogram(hLooseDataFake, 1.0, 1.0);
>   >     can2->SetGridy(true);
>   >     hLooseDataFake->GetXaxis()->SetRangeUser(8, 60);
>   >     hLooseDataFake->SetMinimum(-0.06);
>   >     hLooseDataFake->SetMaximum(0.30);
>   >     hLooseDataFake->GetYaxis()->SetLabelSize(0.047);
>   >     hLooseDataFake->GetYaxis()->SetTitleSize(0.047);
>   >     hLooseDataFake->GetXaxis()->SetTitleSize(0.047);
>   >     hLooseDataFake->GetYaxis()->SetTitle("Linear Deviation");
>   >     hLooseDataFake->SetStats(false); //Removes the stat box on the top plot
>   >     hLooseDataFake->Draw();
>   >   
>   >     fkrlegend.SetTextSize(0.033);  
>   >     }
>   >   
>   >     // Rel 20.7 Uncertainties
>   >     TH1* hLooseMCSysUp   = (TH1*) hLooseMCFake->Clone();
>   >     TH1* hLooseMCSysDown = (TH1*) hLooseMCFake->Clone();
>   > 
>   >     if(isDataData==1)
>   >     for (int i = 1; i < hLooseMCFake->GetNbinsX()+1; i++)
>   >     {
>   >         float value = hLooseMCFake->GetBinContent(i);
>   >         hLooseMCSysUp->SetBinContent(i,value*1.27);
>   >         hLooseMCSysUp->SetBinContent(i,value*0.73);
>   > 
>   >     hLooseMCFake->SetBinError(i,value * 0.27);
>   >     }
>   >   
>   >     if(isDataData==1)
>   >     for (int i = 1; i < hTightMCFake->GetNbinsX()+1; i++)
>   >     {
>   >         float value = hTightMCFake->GetBinContent(i);
>   >         hTightMCFake->SetBinError(i,value * 1.24);
>   >     }
>   > 
>   >     for (int i = 1; i < hTightMCFake->GetNbinsX()+1; i++)
>   >     {
>   >         float value = hTightMCFake->GetBinContent(i);
>   >         //std::cout << "value = " << value << std::endl;
>   >     }
>   >     // -----
>   > 
>   >     hLooseMCFake->SetFillColor(kBlack);
>   >     hLooseMCFake->SetFillStyle(3354);
>   >     hLooseMCFake->DrawCopy("E2 SAME");
>   > 
>   >     hTightMCFake->SetFillColor(kRed);
>   >     hTightMCFake->SetFillStyle(3354);
>   >     hTightMCFake->DrawCopy("E2 SAME");
>   >   
>   >     if (fitType!=1)
>   >       hLooseDataFake_2->Draw("same");
>   >       hTightDataFake_2->Draw("same");
>   > 
>   >     fkrlegend.AddEntry(pLooseData,"Loose Selection", "p");
>   >     fkrlegend.AddEntry(pLooseMC, "Loose Selection, MC16 Simulations", "p");
>   >     fkrlegend.AddEntry(pTightData,"TightPrimary Selection", "p");
>   >     fkrlegend.AddEntry(pTightMC, "TightPrimary Selection, MC16 Simulations", "p");
>   >     fkrlegend.Draw();
>   > 
>   >     //hLooseMCFake->SetFillStyle(1001);
>   >     //hLooseMCFake->SetFillColor(kWhite);
>   >     //hLooseMCFake->SetLineWidth(0);
>   >     //hLooseMCFake->DrawCopy("HIST SAME");
>   >     //hTightMCFake->SetFillStyle(1001);
>   >     //hTightMCFake->SetFillColor(kWhite);
>   >     //hTightMCFake->SetLineWidth(0);
>   >     //hTightMCFake->DrawCopy("HIST SAME");
>   > 
>   >     if (fitType!=1)
>   >     hLooseMCFake_2->Draw("same");
>   >     hTightMCFake->Draw("same");
>   > 
>   >     if(withPad){
>   >     gStyle->SetTextSize(0.08);
>   >     ATLASLabel(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.01, "   Internal");
>   >     gStyle->SetTextSize(0.06);
>   >     myText( can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.06,1, "#it{#sqrt{s}} = 13 TeV, Data 2017");
>   >     gStyle->SetTextSize(0.04);
>   >     myText(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.10,1, d0PlotHeader);
>   >   
>   >     top->RedrawAxis();
>   >     bot->cd();
>   >     bot->SetTopMargin(0.04);
>   > 
>   >     TH1* ratioLoose2 = (TH1*) hLooseMCFake->Clone("ratioLoose2"); // hLooseDataFake->Clone("ratioLoose2");
>   >     TH1* ratioTight2 = (TH1*) hTightMCFake->Clone("ratioTight2"); ///hTightDataFake->Clone("ratioTight2");
>   >     for (int i = 1; i < ratioLoose2->GetNbinsX()+1; i++)
>   >     {
>   >         float valueL = ratioLoose2->GetBinContent(i);
>   >         float valueT = ratioTight2->GetBinContent(i);
>   >         if(valueL==0) ratioLoose2->SetBinContent(i,-100);
>   >         if(valueT==0) ratioTight2->SetBinContent(i,-100);
>   >     }
>   > 
>   > 
>   >     ratioLoose2->SetTitle(" ");
>   >     ratioLoose2->SetStats(false);
>   >     ratioLoose2->GetYaxis()->SetNdivisions(504);
>   >     ratioLoose2->GetYaxis()->SetDecimals(true);
>   >     ratioLoose2->GetYaxis()->SetLabelSize(0.02);
>   >     ratioLoose2->SetFillColor(kBlack);
>   >     ratioLoose2->Divide(hLooseDataFake); ///(hLooseMCFake);
>   >     scaleHistogram(ratioLoose2, 1.0, bottomScale);
>   >     ratioLoose2->Draw("hist PX0");
>   > 
>   >     if(isDataData==2 || isDataData==3)
>   >     {
>   >         ratioLoose2->GetYaxis()->SetTitle("(Data15+Data16)/Data17");
>   >         ratioLoose2->GetYaxis()->SetRangeUser(0.4, 1.6);
>   >     }
>   >     else
>   >     {
>   >         ratioLoose2->GetYaxis()->SetTitle("MC/Data");
>   >         ratioLoose2->GetYaxis()->SetRangeUser(0, 3);
>   >     }
>   > 
>   > 
>   >     ratioTight2->SetTitle(" ");
>   >     ratioTight2->Divide(hTightDataFake); //(hTightMCFake);
>   >     ratioTight2->SetFillColor(kRed);
>   >     ratioTight2->SetStats(false);
>   >     ratioTight2->DrawCopy("hist PX0 same");
>   > 
>   >     line = new TLine();
>   >     line->SetLineStyle(kDashed);
>   >     line->DrawLine(8, 1, 60, 1);
>   > 
>   >     bot->RedrawAxis();
>   > 
>   >     can2->cd();
>   >     can2->Update();
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate_withPad.pdf");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate_withPad.png");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate_withPad.eps");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate_withPad.C");
>   >     } else {
>   >     gStyle->SetTextSize(0.06);
>   >     ATLASLabel(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.01, "   Internal");
>   >     gStyle->SetTextSize(0.04);
>   >     myText( can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.06,1, "#it{#sqrt{s}} = 13 TeV, Data 2017");
>   >     gStyle->SetTextSize(0.03);
>   >     myText(can->GetLeftMargin()+0.03, 1-can->GetTopMargin()-0.05*1.1-0.10,1, d0PlotHeader);
>   > 
>   >     can2->Update();
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate.pdf");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate.png");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate.eps");
>   >     can2->SaveAs(outdir+"/NTracksVsMu_datamc_d0_"+d0RangeType+"fakerate.C");
>   >     }
