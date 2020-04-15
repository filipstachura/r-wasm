## Building from source

Building is easiest on Linux and harder on Mac.

Make sure the prerequisites for emsdk are installed. 
TODO(?): Ryodide will build a custom, patched version of emsdk, so there is no need to build it yourself prior.

Additional build prerequisites are:

    - A working native compiler toolchain, enough to build standard R.
    - gfortran (GNU Fortran 95 compiler)
    - f2c (included in repo)
    - ATLAS CBLAS/LAPACK. You could compile your own CBLAS/CLAPACK or use package manager:
```
apt-get install libblas-dev liblapack-dev libatlas-base-dev -y
```

Download F2Clib from https://www.netlib.org/f2c/. We need f2clib(the library),
make a folder and unzip this file inside the folder (else you'll clutter your workspace).

R was downloaded from the R website

Directory structure:
```
r-wasm
|
|--> README.md
|--> R-3.6.1
|--> libf2c/
|--> pref [create empty folder]
```

Read README in f2clib. Modify makefile to compile with `CFLAGS=-fPIC`.
In typical setup you need to modify headers according to README:
```
        mv f2c.h f2c.h0
        sed 's/long int /int /' f2c.h0 >f2c.h
```
This is important else all R calls to fortran result in segfaults of random errors. 

Make sure that both files are present before moving forward:
```
libf2c/f2c.h
libf2c/libf2c.a
```

Now, go to the R folder!

You won't need to copy/modify anything since commit [4afa62b81308c72ed4aa72c23af88113401138a2](https://github.com/iodide-project/r-wasm/commit/4afa62b81308c72ed4aa72c23af88113401138a2) onward have the files modified.

Next, inside R-3.6.1 run (change the paths to reflect your installation):

```
F2C="$HOME/r-wasm/libf2c" CPPFLAGS="-I$F2C" CFLAGS="-I$F2C" MAIN_LDFLAGS="-L$F2C" SHLIB_LDFLAGS="-L$F2C" LDFLAGS="-L$F2C" \ 
./configure --prefix=$HOME/r-wasm/pref/  --with-blas="-L/usr/lib64/atlas/ -ltatlas"     --with-lapack  --with-x=no --enable-java=no --with-readline=no --with-recommended-packages=no  --enable-BLAS-shlib=no --enable-R-shlib=yes --with-tcltk=no
make V=1 -j15 | tee -a what_run.txt
make install help

```
Confirm that `what_run.txt` does not contain any calls to `gfortran` (except for -lgfortran which i *could not remove*).

Now you can run some tests! See https://cran.r-project.org/doc/manuals/r-release/R-admin.html#Testing-a-Unix_002dalike-Installation
Unfortunately for some reason I've not been able to install help files fro the stats library. the tests depend on those help files being present.

