# This is aimed at jotting down some useful commands for installing and checking the builds of GAMBIT and FLAVIO which are very useful for doing certain particle physics analyses. # 

## "GAMBIT and FLAVIO installation" ##
===============================================================================
## Last Edited: 07-01-2019

# FLAVIO installation
~~~
pip3 install flavio --user
pip3 install flavio[plotting,sampling,testing] --user
~~~

# GAMBIT installation

In Pip
    yaml -> pyyaml                                                                                                       
    itertools -> more-itertools                                                                                       

## 1) Download GAMBIT for mac
~~~
curl -OL http://www.hepforge.org/archive/gambit/gambit-1.1.2.tar.gz
tar -xvf gambit-1.1.2.tar.gz
cd gambit_1.1-1.1.2
~~~
  
## 2) Building  
~~~
mkdir build
cd build
cmake -D CMAKE_CXX_COMPILER=g++-6 -D CMAKE_C_COMPILER=gcc-6 -D CMAKE_Fortran_COMPILER=gfortran-6 -Ditch="Delphes;great" ../ 
make multinest
make diver
cmake ..          (Don't forget this step!)
make -j2 gambit
~~~  

## 3) To build all backend codes
~~~
make -j2 backends
~~~

## 4) Install pippi and ctioga2
~~~
sudo gem install ctioga2 
make get-pippi
~~~

## 5) Tests
~~~
./gambit backends
./gambit scanners
./gambit -f yaml_files/spartan.yaml
~~~

If the gambit line doesn't work citing printer errors on the last test, open up spartan.yaml in your editor and in the Printer section comment out the right printer
~~~
vim yaml_files/spartan.yaml
~~~
  -> Comment in the ascii block
  -> Comment out the hdf5 one.   
~~~  
imgcat runs/WC_lite/plots/WC_7_like1D.pdf 
imgcat runs/WC_lite/plots/WC_10_like1D.pdf
imgcat runs/WC_lite/plots/WC_7_10_like2D.pdf

open runs/WC_lite/plots/WC_7_like1D.pdf 
open runs/WC_lite/plots/WC_10_like1D.pdf
open runs/WC_lite/plots/WC_7_10_like2D.pdf
~~~
