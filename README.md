# Flatiron Institute Nonuniform Fast Fourier Transform libraries: FINUFFT

### Barnett, Magland, Greengard

Includes code by:

P. Swarztrauber - FFTPACK  
Tony Ottosson - evaluate modified Bessel function I_0  
June-Yub Lee - some test codes co-written with Greengard  
Nick Hale and John Burkardt - Gauss-Legendre nodes and weights  

### Purpose

to do

### Dependencies

The basic libraries need a C++ compiler, GNU make, FFTW, and optionally OpenMP (the makefile can be adjusted for single-threaded).
The fortran wrappers need a fortran compiler.
To run optional speed comparisons which link against the CMCL NUFFT library, this must be installed.
See settings in the `makefile`.

### Installation

1. Download using `git`, `svn`, or as a zip (see green button above).
1. `cp makefile.dist makefile`
1. edit `makefile` for your system
1. `make`


### Contents of this package

  `src` : main library source and headers.  
  `examples` : test codes (drivers) which verify libaries are working correctly, perform speed tests, and show how to call them.  
  `examples/results` : accuracy and timing outputs.  
  `contrib` : 3rd-party code.  
  `matlab` : wrappers and examples for MATLAB. (Not yet working)  
  `fortran` : wrappers and drivers for Fortran. (Not yet working)  
  `doc` : the manual (not yet there)  
  `README.md`  
  `LICENSE`  
  `makefile.dist` : GNU makefile (user should first copy to `makefile`)  

### Notes

C\++ is used for all main libraries, although without much object-oriented code. C\++ complex type arithmetic is not used in the main library, rather FFTW complex types are used, since it is a glorified driver for FFTW. The test codes use C\++ complex types ("dcomplex"). FFTW was considered universal and essential enough to be a dependency for the whole package.

We use an unpublished simplification of the Kaiser--Bessel spreading kernel rather than truncated Gaussians, since they allow roughly half the kernel width for high requested precisions. Our kernel is of the form exp(-beta.sqrt(1-(2x/W)^2)). This, and Kaiser--Bessel, are good approximations to the prolate spheroidal wavefunction of order zero (PSWF), being the functions of given support [-W/2,W/2] whose Fourier transform has minimal L2 norm outside a symmetric interval. The PSWF frequency parameter (see [ORZ]) is c = pi.(1-1/2R).W where R is the upsampling parameter (currently R=2.0), and W the full kernel width.

References for this include:

[ORZ] Prolate Spheroidal Wave Functions of Order Zero: Mathematical Tools for Bandlimited Approximation.  A. Osipov, V. Rokhlin, and H. Xiao. Springer (2013).

[KK] Chapter 7. System Analysis By Digital Computer. F. Kuo and J. F. Kaiser. Wiley (1967).

[FS] Nonuniform fast Fourier transforms using min-max interpolation.
J. A. Fessler and B. P. Sutton. IEEE Trans. Sig. Proc., 51(2):560-74, (Feb. 2003)

This code builds upon the CMCL NUFFT, and the Fortran wrappers duplicate its interfaces. For this the following are references:

[GL] Accelerating the Nonuniform Fast Fourier Transform. L. Greengard and J.-Y. Lee. SIAM Review 46, 443 (2004).

[LG] The type 3 nonuniform FFT and its applications. J.-Y. Lee and L. Greengard. J. Comput. Phys. 206, 1 (2005).

The original NUFFT analysis using truncated Gaussians is:

[DR] Fast Fourier Transforms for Nonequispaced data. A. Dutt and V. Rokhlin. SIAM J. Sci. Comput. 14, 1368 (1993). 

### To do

* include nf1 etc size check before alloc, exit gracefully if exceeds RAM
* test non-openmp compile
* theory work on exp(sqrt) being close to PSWF
* figure out why bottom out ~ 1e-10 err for big arrays in 1d. unavoidable roundoff? small arrays get to 1e-14.
* Checkerboard per-thread grid cuboids, compare speed in 2d and 3d against current 1d slicing.
* decide to cut down intermediate copies of input data eg xj -> xp -> xjscal -> xk2 to save RAM in large problems?
* single-prec compile option for RAM-intensive problems?
* test BIGINT -> long long slows any array access down, or spreading? allows I/O sizes (M, N1*N2*N3) > 2^31. Note June-Yub int*8 in nufft-1.3.x slowed things by factor 2-3.
* rename examples as test?
* fortran wrappers (rmdir greengard_work, merge needed into fortran)
* matlab wrappers, mcwrap issue w/ openmp, mex, and subdirs.
* spread_f and matlab wrappers need ier output
* license file
* outreach, alert Dan Foreman-Mackey re https://github.com/dfm/python-nufft
* doc/manual
* boilerplate stuff as in CMCL page

### Done

* efficient modulo in spreader, done by conditionals
* removed data-zeroing bug in t-II spreader, slowness of large arrays in t-I.
* clean dir tree
* spreader dir=1,2 math tests in 3d, then nd.
* Jeremy's request re only computing kernel vals needed (actually was vital for efficiency in dir=1 openmp version), Ie fix KB ker eval in spreader so doesn't wdo 3d fill when 1 or 2 will do.
* spreader removed modulo altogether in favor of ifs
* OpenMP spreader, all dims
* multidim spreader test, command line args and bash driver
* cnufft->finufft names, except spreader still called cnufft
* make ier report accuracy out of range, malloc size errors, etc
* moved wrappers to own directories so the basic lib is clean
* fortran wrapper added ier argument
* types 1,2 in all dims, using 1d kernel for all dims.
* fix twopispread so doesn't create dummy ky,kz, and fix sort so doesn't ever access unused ky,kz dims.
* cleaner spread and nufft test scripts
* build universal ndim Fourier coeff copiers in C and use for finufft
* makefile opts and compiler directives to link against FFTW.
* t-I, t-II convergence params test: R=M/N and KB params
* overall scale factor understand in KB
* check J's bessel10 approx is ok.
* meas speed of I_0 for KB kernel eval
* understand origin of dfftpack (netlib fftpack is real*4)
* [spreader: make compute_sort_indices sensible for 1d and 2d. not needed]
* next235even for nf's
* switched pre/post-amp correction from DFT of kernel to F series (FT) of kernel, more accurate
* Gauss-Legendre quadrature for direct eval of kernel FT, openmp since cexp slow
* optimize q (# G-L nodes) for kernel FT eval on reg and irreg grids (common.cpp). Needs q a bit bigger than like (2-3x the PTR, when 1.57x is expected). Why?
* type 3 segfault in dumb case of nj=1 (SX product = 0). By keeping gam>1/S
* optimize that phi(z) kernel support is only +-(nspread-1)/2, so w/ prob 1 you only use nspread-1 pts in the support. Could gain several % speed for same acc.
* new simpler kernel entirely
* cleaned up set_nf calls and removed params from within core libs
* test isign=-1
* type 3 in 2d, 3d