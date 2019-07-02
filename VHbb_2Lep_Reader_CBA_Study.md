# This is aimed at the main analysis of my PhD: Associated Production of Higgs with a Vector boson (most likely Z) where the Higgs decays to a pair of bottom quarks. This document is about generating and creating ntuples from VHbb CxAOD's and testing them. #

## "VHbb 2 Lepton Reader CBA Study" ##
===============================================================================
Last Edited: 28-05-2018
-------------------------------------------------------------------------------

# Setup

The first thing that you want to do is ensure that you are up to date with the framework. To do this you should have access to the latest branch/tag of the CxAOD Framework. It's good to check online
(https://gitlab.cern.ch/CxAODFramework/FrameworkSub/tree/master/bootstrap) but as it's said in VHBB_2Lep_Reader_Inputs.md there is a process of continual updates.

For a more detailed version of what to do in terms of the setup, see VHBB_2Lep_Reader_Inputs.md

# Nominal CBA Run (without Systematics)
The CBA analysis is an important cross-check for the baseline MVA analysis, and as such also needs to be optimised for each analysis round. There have been some optimisations performed for the 0L and 1L CBA but it has yet to be seen if these work for the 2L.  The idea for this section is to run the CBA in the CxAOD framework and compare output and yields to that a) of the EPS study and b) to the 0/1L results.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.33
setupATLAS && lsetup git
~~~
To calibrate this though, you need to look over the file submitReader.sh. Be wary this help is only relevant for that version of code.
~~~
vim source/FrameworkExe/scripts/submitReader.sh
~~~
>   CHANGE the driver to run on condor
>   > DRIVER="condor" (L52) lxplus is migrating from LSF to HTCondor

> CHANGE other running options L56 - L65
>   > JOBSIZELIMITMB="-1"
>   > NRFILESPERJOB="10"
>   > NOMINALONLY="true"
> COMMENT IN SAMPLE FILES (~L69-94)
>   > SAMPLES1=""
>   > #SAMPLES1+="cutflow-dat16 qqZllHbbJ_PwPy8MINLO" #R31 cutflow
>   >
>   > SAMPLES1=""
>   > #SAMPLES1+=" data15 data16 data17" # data
>   >   if [[ "${MCTYPEs}" == "a" ]]; then
>   >       SAMPLES1+=" data15 data16" # old data
>   >   fi
>   >   if [[ "${MCTYPEs}" == "d" ]]; then
>   >       SAMPLES1+=" data17 " # new data
>   >   fi
>   >
>   > SAMPLES1+=" qqZllHbbJ_PwPy8MINLO qqWlvHbbJ_PwPy8MINLO ggZllHbb_PwPy8" # VH + extra jets, V->leptons, H->bb: Signal
>   >
>   > SAMPLES1+=" WqqWlv_Sh221 ggWqqWlv_Sh222" # WW
>   > SAMPLES1+=" WqqZvv_Sh221 WqqZll_Sh221 WlvZqq_Sh221" # WZ
>   > SAMPLES1+=" ZqqZvv_Sh221 ZqqZll_Sh221 ggZqqZvv_Sh222 ggZqqZll_Sh222" # ZZ
>   >
>   > SAMPLES1+=" stops_PwPy8 stopt_PwPy8 stopWt_PwPy8" # stop
>   > SAMPLES1+=" ttbar_dil_PwPy8" # ttbar dilepton
>   > #SAMPLES1+=" ttbar_nonallhad_PwPy8" # ttbar nonallhad
>   >
>   > SAMPLES1+=" WenuB_Sh221 WenuC_Sh221 WenuL_Sh221 Wenu_Sh221 WmunuB_Sh221 WmunuC_Sh221 WmunuL_Sh221 Wmunu_Sh221 WtaunuB_Sh221 WtaunuC_Sh221 WtaunuL_Sh221 Wtaunu_Sh221" # W+jets
>   > SAMPLES1+=" ZeeB_Sh221 ZeeC_Sh221 ZeeL_Sh221 Zee_Sh221 ZmumuB_Sh221 ZmumuC_Sh221 ZmumuL_Sh221 Zmumu_Sh221 ZtautauB_Sh221 ZtautauC_Sh221 ZtautauL_Sh221 Ztautau_Sh221 ZnunuB_Sh221 ZnunuC_Sh221 ZnunuL_Sh221 Znunu_Sh221" # Z+jets
~~~
vim source/FrameworkExe/data/framework-read-automatic.cfg
~~~
> CHANGE
>   > bool doMergeJetBins   = true (L24)
>   > bool autoDiscoverVariations = false (L214)

The next thing you need to do is to impliment the SM cuts in the 2L reader. This presentation
https://indico.cern.ch/event/730953/contributions/3013210/subcontributions/255870/attachments/1654536/2647999/VHbb_20180523.pdf has the cuts for me but you will want to find them in a similar format. After reading the cuts that made the old (in this case EPS) analysis, the thing to do is to impliment the cuts listed. For example, in the EPS analysis

OLD
- pTV 150-250, dRBB 1.8   | mTW < 120GeV
- pTV 200+, dRBB 1.2        | mTW < 120GeV

NEW
- pTV 150-200, dRBB 1.7   | mTW < 120GeV
- pTV 200-250, dRBB 1.4   | mTW < 120GeV
- pTV 250+, dRBB 1.2        | mTW < 120GeV
~~~
vim source/CxAODReader_VHbb/Root/AnalysisReader_VHbb2Lep.cxx
~~~
From L430 there is the structure for implimenting cuts to the CxAODs depending on the model region you are interersted in. The if statement ~L473 is the key here. It contains 4 vectors of unsigned ling integers to denote whether certain cuts are in effect.

For example the cuts_common_resolved, there is an integer flag that corresponds to DiLeptonCuts::dRCut. If there is a line that contains this cut of the syntax.

> if ( m_model == Model::SM && blah blah ) {
updateFlag(eventFlag, DiLeptonCuts::dRCut);
} (~L340)

Then the flag eventFlag knows about the cut, and the next step is to check that the actual cut options are correct which happens nearby in the code. So if you want to add your own cut, then you need to add a lines similar to this.

>   // **Selection** : ETMissOverRootST for CUT CBA
>   //-----------------------------------------
>
>   double ETMissOverRootST = 3.5; //GeV^{-1}
>
>   if ( m_model == Model::SM && METHTsmallR < ETMissOverRootST){
>   //METHTsmallR defined on L168
>   updateFlag(eventFlag, DiLeptonCuts::ETMissOverRootST);
>   }

This is not necessary because the DiLeptonCuts::METHTMerged/Resolved does exactly this.
So you can add DiLeptonCuts::METHTResolved to do the same job. Other cuts may need to be edited.
> if ( m_model == Model::CUT ) {
> cuts_common_resolved = {...}
> }; ~L477

Once all the cuts you want are edited (or inserted) and saved. You can run twice. Once with the old cuts under SM and one with the new cuts.
~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_branch_21.2.33/build
setupATLAS
make -j6
cd ..
source source/FrameworkExe/scripts/setupLocal.sh
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 R31-10_SM_Nominal_OLD 2L a CUT D1 31-10 1
--AND--
../source/FrameworkExe/scripts/submitReader.sh /eos/atlas/atlascerngroupdisk/phys-higgs/HSG5/Run2/VH/CxAOD_r31-10 R31-10_SM_Nominal_NEW 2L a,d CUT D1 31-10 1
~~~
Now if these have run sucessfully it's time to add these inputs together

~~~
cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/R31-10_SM_Nominal_NEW/Reader_2L_SM_31-10_a
vim lofinputs.txt

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/R31-10_SM_Nominal_NEW/Reader_2L_SM_31-10_d
vim lofinputs.txt

cd /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/R31-10_SM_Nominal_OLD/Reader_2L_SM_31-10_a
vim lofinputs.txt


cd /afs/cern.ch/work/d/dspiteri/VHbb/
setupATLAS && lsetup git
cd DynPlottingTool
lsetup root
make clean
make -j6

./PlotMaker -x false -l /afs/cern.ch/work/d/dspiteri/VHbb/CxAODFramework_tag_r31-10/run/R31-10_SM_Nominal_NEW/Reader_2L_SM_31-10_a/lofinputs.txt -B SM -S SM -d data -s ggZHll125 -w 1 -f 1.0 -v mBB/4-400 -o VHbb2Lep_CBAStudy_a -D 1 -e ggZvvH125 2>&1 -b true | tee VHbb2Lep_CBAStudy_a.log
~~~



