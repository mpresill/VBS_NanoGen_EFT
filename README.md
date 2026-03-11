# VBShadNanoGen

References for NanoGEN: https://twiki.cern.ch/twiki/bin/viewauth/CMS/NanoGen, https://indico.cern.ch/event/962610/contributions/4325186/attachments/2235981/3789869/MG5%20tutorial(1).pdf, https://github.com/FNALLPC/cmseft 
NanoGEN is a very convenient format for exploratory studies. The event content of the flat trees is similar to the generator infomration in NanoAOD, but much faster generation time because the detector simulation and reconstruction is being skipped.

References to setup the validation chain needed for EFT samples: https://indico.cern.ch/event/1602054/contributions/6877388/attachments/3210205/5718109/TopPAG_EFTValidation.pdf 

Goals of the validation:
* Produce a sample that isn’t dominated by very large or very small weights 
* Validate the EFT sample reweighted to the SM point against approved central samples - following our publication https://arxiv.org/pdf/2406.14620 "LHC EFT WG Note: SMEFT predictions, event reweighting, and
simulation"
* Validate the EFT parameterization of the sample (i.e. check that the $\sigma_{EFT}$ scales appropriately to the polynomial formula)


# Run 3 test setup:


## Generating EDM GEN files 
Gridpack to be tested as example from VBS semileptonic final state can be found here: `/afs/cern.ch/work/m/mpresill/public/Run3_dim6_semileptonic_gridpacks/wmzjjdim6_ewk_el8_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz`.
Producing GEN files from the above gridpack is usually straight forward and similar to other CMS samples. We will use a fragment file that defines the settings that will be used for decays, parton shower and hadronization in pythia.

Run 3 fragments are taken from these [chains](https://gitlab.cern.ch/cms-gen/mccm/-/issues?sort=created_date&state=opened&search=%5BSMP%5D+VBS&first_page_size=20&show=eyJpaWQiOiIxMTk1IiwiZnVsbF9wYXRoIjoiY21zLWdlbi9tY2NtIiwiaWQiOjM1OTIxN30%3D) 
e.g. for Summer23: https://cms-pdmv-prod.web.cern.ch/mcm/requests?prepid=SMP-Run3Summer23wmLHEGS-00186&page=0&shown=127 , processed with CMSSW release `CMSSW_13_0_24`.





# Various examples from Run 2:
## Setup from Davide to keep LHE Reweighting Weights

```sh
# Passo 0 per alma9:
cmssw-el7 # or use the singularity image from piergiulio: https://gitlab.cern.ch/cms-cat/cmssw-lxplus 
export SCRAM_ARCH=slc7_amd64_gcc700

# Passo 1: Creazione dell'ambiente CMSSW
cmsrel CMSSW_10_6_26
cd CMSSW_10_6_26/src
# Passo 2: Configurazione dell'ambiente
cmsenv
# Passo 3: Inizializzazione del repository Git
git cms-init
# Passo 4: Aggiunta del repository remoto
git remote add valsdav git@github.com:valsdav/cmssw.git
# Passo 5: Recupero della branch desiderata
git fetch valsdav nanogen_EFT_weightspatch_10_6_26
# Passo 6: Unione della branch nel tuo ambiente di lavoro
git checkout -b nanogen_EFT_weightspatch_10_6_26 valsdav/nanogen_EFT_weightspatch_10_6_26
# Passo 7: Compilazione del codice
scram b -j 5


# Passo 8: configura questa cartella per la sottomissione dei files 
mkdir Configuration
cd Configuration
git clone git@github.com:mpresill/VBShadNanoGen.git #or just create your fragment in a <name>/python subfolder
scram b
```

## Setup to run NanoGen with default weights

```sh
cmsrel CMSSW_11_2_0_pre7
cd CMSSW_11_0_0_pre7/src
cmsenv
# The following merge is not strictly necessary, but it enables a bit of functionality
git cms-init
git cms-merge-topic kdlong:NanoGen_dqm
scram b -j 5

mkdir Configuration
cd Configuration
git clone git@github.com:kdlong/WMassNanoGen.git #or just create your fragment in a <name>/python subfolder
scram b
```

## Setup to store all genWeights

```sh 
cmsrel CMSSW_10_6_18
cd CMSSW_10_6_18/src
cmsenv
git cms-init
git cms-merge-topic kdlong:NanoGenWeights_CMSSW_10_6_18
scram b -j 5

mkdir Configuration
cd Configuration
git clone git@github.com:kdlong/WMassNanoGen.git
scram b
```



# Making configs and running

First create a config fragment in python/. Follow the other examples there.
Generate the config for NanoGen with cmsDriver. If you want gridpack --> NanoGen, you can use the script runCmsDriverNanoGen.sh. To generate a full config from your fragment and run:
```sh
scram b
runCmsDriverNanoGen.sh <fragmentName_cff.py> <outputFile.root>
cmsRun fragmentName_cfg.py
```
An example to generate NanoGen from a MiniAOD file (useful to keep more gen particles or more LHE weights) is in runCmsDriverGenericMiniToNanoGen.sh.
```sh
scram b
cmsDriverZJMiNNLONanoGen.sh
cmsRun fragmentName_cfg.py
```
Example crab submit files are in the crab_submit_files directory. Note that you need to copy the gridpacks to a location readable from crab for this to work.

