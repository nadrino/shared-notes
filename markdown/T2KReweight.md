Installing NEUT
---------------

```sh
setup_old_gcc
set_t2k_CERNLIB
set_t2k_psyche # old root -> + compiled without GSL -> needs to remove instances
cd $REPO_DIR

git clone https://github.com/yhayato/neut.git
cd neut/
git branch -a
git checkout remotes/origin/OA2020Development
git tag
git checkout tags/OA2020RC2_p1

cd src/
autoreconf -if

mkdir $INSTALL_DIR/neut;
mkdir $BUILD_DIR/neut; cd $BUILD_DIR/neut;
$REPO_DIR/neut/src/configure --prefix=$INSTALL_DIR/neut

# patch
sed -i -e "s/-lEve //g" /sps/t2k/ablanche/build/neut/neutgeom/Makefile

make install

```

Installing NIWG
---------------

```sh
setup_old_gcc
set_t2k_CERNLIB
set_t2k_psyche # old root ?
# set_t2k_highland2
cd $REPO_DIR

git clone https://github.com/t2k-software/NIWGReWeight.git NIWGReWeight_ROOT6
cd NIWGReWeight
git branch -a
# git checkout --track remotes/origin/bugfix/EbSplineBuild
git checkout --track remotes/origin/develop

make

```

Installing JNuBeam (JReWeight)
------------------------------

```sh
mkdir $REPO_DIR/t2k-cvs
cd $REPO_DIR/t2k-cvs

setup_old_gcc
set_t2k_psyche # set CMT + align with nd280 cvs

export CVSROOT=:ext:anoncvs@repo.t2k.org:/home/trt2kmgr/T2KRepository
export CVS_RSH=ssh
unset CVS_SERVER

cvs co GlobalAnalysisTools/JReWeight
cd GlobalAnalysisTools/JReWeight
make

```

Installing T2KReWeight
----------------------

```sh
cd $REPO_DIR
git clone https://github.com/t2k-software/T2KReWeight.git
cd T2KReWeight
git branch -a
git checkout --track remotes/origin/feature/BANFF2020genWeights # does not work on CCLyon's git version...

cd $REPO_DIR/T2KReWeight

# For using OLD GCC (like gcc-5)
setup_old_gcc
# For analysis tools
set_t2k_highland2
# For enabling psyche
set_t2k_psyche
# For enabling CERNLIB
set_t2k_CERNLIB
# For enabling NEUT
set_t2k_neut
# For OAanalysis
set_t2k_oaAnalysisReader

# NIWG
export NIWG=$REPO_DIR/NIWGReWeight/
export LD_LIBRARY_PATH=${NIWG}:$LD_LIBRARY_PATH;
export NIWGREWEIGHT_INPUTS=${NIWG}/inputs

export T2KREWEIGHT=$REPO_DIR/T2KReWeight
export PATH=$T2KREWEIGHT/bin:$PATH:$T2KREWEIGHT/app:$ROOTSYS/bin:$PATH;
export LD_LIBRARY_PATH=$T2KREWEIGHT/lib:$LD_LIBRARY_PATH;
cleanup_env

# For JReweight
export JNUBEAM=$REPO_DIR/t2k-cvs/GlobalAnalysisTools/JReWeight
export LD_LIBRARY_PATH=$JNUBEAM:$LD_LIBRARY_PATH;


./configure \
  --enable-oaanalysis \
  --enable-neut \
  --enable-psyche \
  --enable-niwg \
  --enable-jnubeam \
  --with-cern=$(echo $CERN_ROOT)
gmake
cd app/

# sed -i -e "s/\/vols\/build\/t2k\/cvw09\/neut.d.card/\/sps\/t2k\/ablanche\/install\/neut\/share\/neut\/Cards\/neut.card/g" genWeightsFromNRooTracker_BANFF_2020.cxx
sed -i -e "s/\/vols\/build\/t2k\/cvw09\/neut.d.card/\/sps\/t2k\/ablanche\/resources\/neut.d.card/g" genWeightsFromNRooTracker_BANFF_2020.cxx

make genWeightsFromNRooTracker_BANFF_2020

```

```sh
genWeightsFromNRooTracker_BANFF_2020.exe -p /sps/t2k/ablanche/work/irods/run2a_FlatTree_v2r41_11_25_000.root -o test.root -r MC
```

```c
TClonesArray *arr = new TClonesArray("TGraph");
sample_sum->SetBranchAddress("MAQEGraph",&arr);
int reac = 0;
flattree->SetBranchAdress("sTrueVertexReacCode", &reac);
int nentries = sample_sum->GetEntries();
TCanvas* c = new TCanvas("c", "c", 800, 600);
for (int ev=0;ev<nentries;ev++) {
  arr->Clear();
  sample_sum->GetEntry(ev);
  flattree->GetEntry(ev);
  int ngraph = arr->GetEntriesFast();
  for (int i=0;i<ngraph;i++) {
    TGraph *gr = (TGraph*)arr->At(i);
    gr->SetMarkerStyle(kFullDotLarge);
    gr->SetMarkerSize(1);
    gr->GetXaxis()->SetRangeUser(-1,1);
    gr->GetYaxis()->SetRangeUser(-1,1);
    cerr << "Event = " << ev << " / Graph = " << i << " / sTrueVertexReacCode=" << reac << endl;
    for(i_pt = 0 ; i_pt < gr->GetN() ; i_pt++){
      cerr << "(x,y) = " <<  gr->GetX()[i_pt] << ", " << gr->GetY()[i_pt] << endl;
    }
    if(ev == 0 && i == 0){
      gr->Draw("AP");
    } else{
      gr->Draw("PSAME");
    }
  }
}
```
