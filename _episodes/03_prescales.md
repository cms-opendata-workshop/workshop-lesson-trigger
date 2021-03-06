---
title: "Getting trigger prescales"
teaching: 0
exercises: 25
questions:
- "How do I get the prescales for my triggers?"
objectives:
- "Learn how to extract the prescales for you triggers"
keypoints:
- "Prescales can be extracted using the trigger tools in CMSSW and from the *brilcalc* tool."
---

## The TriggerInfoTool

The CERN Open Data portal maintains some records that explain how to use certain tools for analysis.  One of those is the record about the [TriggerInfoTool](http://opendata.cern.ch/record/5004).  You immediately see that it points to the [TriggerInfoTool repository](https://github.com/cms-opendata-analyses/TriggerInfoTool/tree/2011) on Github.  This a place where you can find a few examples and explore the way the code is used for extracting trigger information.

## Get the Prescales and the acceptance bit

Let's work on one of the examples stored in the repository above.  In particular, we are going to look at the [TriggerSimplePrescalesAnalyzer](https://github.com/cms-opendata-analyses/TriggerInfoTool/tree/2011/TriggerSimplePrescalesAnalyzer) example (package).  The following directions can also be found in that repository.

First make sure you are at the top of your `CMSSW_5_3_32/src` area and that you have issued the `cmsenv` command.

Now let's clone the *2011* branch of this repository, which will be good for the 2012 data we are working with:

```bash
git clone -b 2011 git://github.com/cms-legacydata-analyses/TriggerInfoTool.git
```

Compile:

```bash
scram b
```

Edit the config file to choose the triggers we are interested in (you can use `vi` or other editor).  Remember, we will exercise the `TriggerSimplePrescalesAnalyzer` example:

```bash
vi TriggerInfoTool/TriggerSimplePrescalesAnalyzer/python/simpleprescalesinfoanalyzer_cfg.py
```
Replace the `PoolSource` files with the ones we used in our last episode, replace the triggerPatterns parameter with a simpler trigger, like `"HLT_Mu12_v??"` (note the wildcard at the end `??`, so we can get the prescales for all versions of this trigger).

> ## Take a loot at the config file
>
> The config file should look like:
> ~~~
> import FWCore.ParameterSet.Config as cms
>
> process = cms.Process("TriggerInfo")
>
> process.load("FWCore.MessageService.MessageLogger_cfi")
> #if more events are activated, choose to print every 1000:
> #process.MessageLogger.cerr.FwkReport.reportEvery = 1000
>
> process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(100) )
>
> process.source = cms.Source("PoolSource",
>     fileNames = cms.untracked.vstring(
>         'root://eospublic.cern.ch//eos/opendata/cms/Run2012B/TauPlusX/AOD/22Jan2013-v1/20000/0040CF04-8E74-E211-AD0C-00266CFFA344.root',
>         'root://eospublic.cern.ch//eos/opendata/cms/Run2012C/TauPlusX/AOD/22Jan2013-v1/310001/0EF85C5C-A787-E211-AFC9-003048C6942A.root'
>     )
> )
>
> #needed to get the actual prescale values used from the global tag
> process.load('Configuration.StandardSequences.FrontierConditions_GlobalTag_cff')
> process.GlobalTag.connect = cms.string('sqlite_file:/cvmfs/cms-opendata-conddb.cern.ch/FT_53_LV5_AN1_RUNA.db')
> process.GlobalTag.globaltag = 'FT_53_LV5_AN1::All'
>
> #configure the analyzer
> #inspired by https://github.com/cms-sw/cmssw/blob/CMSSW_5_3_X/HLTrigger/HLTfilters/interface/HLTHighLevel.h
> process.gettriggerinfo = cms.EDAnalyzer('TriggerSimplePrescalesAnalyzer',
>                               processName = cms.string("HLT"),
>                              triggerPatterns = cms.vstring("HLT_Mu12_v??"), #if left empty, all triggers will run
>                               triggerResults = cms.InputTag("TriggerResults","","HLT"),
>                               triggerEvent   = cms.InputTag("hltTriggerSummaryAOD","","HLT")
>                               )
>
>
> process.triggerinfo = cms.Path(process.gettriggerinfo)
> process.schedule = cms.Schedule(process.triggerinfo)
> ~~~
> {: .language-python}
{: .solution}

Let's run:

```bash
cmsRun TriggerInfoTool/TriggerSimplePrescalesAnalyzer/python/simpleprescalesinfoanalyzer_cfg.py  > full_prescales.log 2>&1 &
```

> ## Let's check the output
>
> ~~~
> HLTConfig has changed . . .
> Begin processing the 1st record. Run 194075, Event 14880766, LumiSection 48 at 29-Sep-2020 03:11:45.700 CEST
> Currently analyzing trigger HLT_Mu12_v16
> analyzeSimplePrescales: path HLT_Mu12_v16 [121] prescales L1T,HLT: 50,30
>  Trigger path status: WasRun=1 Accept=0 Error =0
> Begin processing the 2nd record. Run 194075, Event 14844046, LumiSection 48 at 29-Sep-2020 03:11:45.712 CEST
> Currently analyzing trigger HLT_Mu12_v16
> analyzeSimplePrescales: path HLT_Mu12_v16 [121] prescales L1T,HLT: 50,30
>  Trigger path status: WasRun=1 Accept=0 Error =0
> ...
> ~~~
> {: .output}
>
> Notice that the L1 prescale was 50 for that particular run, whereas the HLT prescale was 30.  We would have to run on many more events to see an `Accept=1`, which would mean the event was accepted by this trigger.  
>
{: .solution}

> ## Challenge!
>
> How about now you run with the triggers we were interested in, i.e., the `HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v?` ones.  What do you get?
>
> > ## solution
> >
> > You will see an output like this:
> > ~~~
> > HLTConfig has changed . . .
> > Begin processing the 1st record. Run 194075, Event 14880766, LumiSection 48 at 29-Sep-2020 02:26:23.553 CEST
> > Currently analyzing trigger HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2
> > %MSG-e HLTConfigData:  TriggerSimplePrescalesAnalyzer:gettriggerinfo  29-Sep-2020 02:26:23 CEST Run: 194075 Event: 14880766
> >  Error in determining L1T prescale for HLT path: 'HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2' with L1T seed: 'L1_SingleMu14er OR L1_SingleMu16er' using L1GtUtils: error code: 210001. > > (Note: only a single L1T name, not a bit number, is allowed as seed for a proper determination of the L1T prescale!)
> > %MSG
> > analyzeSimplePrescales: path HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2 [394] prescales L1T,HLT: -1,1
> >  Trigger path status: WasRun=1 Accept=0 Error =0
> > Begin processing the 2nd record. Run 194075, Event 14844046, LumiSection 48 at 29-Sep-2020 02:26:23.584 CEST
> > Currently analyzing trigger HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2
> > %MSG-e HLTConfigData:  TriggerSimplePrescalesAnalyzer:gettriggerinfo  29-Sep-2020 02:26:23 CEST Run: 194075 Event: 14844046
> >  Error in determining L1T prescale for HLT path: 'HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2' with L1T seed: 'L1_SingleMu14er OR L1_SingleMu16er' using L1GtUtils: error code: 210001. > > (Note: only a single L1T name, not a bit number, is allowed as seed for a proper determination of the L1T prescale!)
> > %MSG
> > ~~~
> > {: .output}
> >
> > Life is not so easy sometimes.  While there was no problem getting the HLT prescale (it seems to be 1), i.e., the trigger is unprescaled at the HLT, we can't say
> > anything about the L1 because there is a limitation in our software.  Can you understand why we are getting this error?
> >
> > ~~~
> >  Error in determining L1T prescale for HLT path: 'HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v2' with L1T seed: 'L1_SingleMu14er OR L1_SingleMu16er' using L1GtUtils: error code: 210001. > > (Note: only a single L1T name, not a bit number, is allowed as seed for a proper determination of the L1T prescale!)
> > ~~~
> > {: .error}
> >
> > Fortunately, there is a backup solution for us to check on the prescales.  In a later lesson you will learn about the *Brilcalc* tool.  Once you are setup for using it, you could try this line to check on the L1 prescales for this particular run:
> >
> > ~~~
> > brilcalc trg --prescale -r 194075 --hltpath "HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v?"
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

> ## Explore the TriggerSimplePrescalesAnalyzer!
>
> On overtime, you could explore the implementation of the code we just used.  Play around with it, what else can you learn?
{: .discussion}



{% include links.md %}
