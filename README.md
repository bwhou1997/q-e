![q-e-logo](logo.jpg)

This is the distribution of the Quantum ESPRESSO suite of codes (ESPRESSO:
opEn-Source Package for Research in Electronic Structure, Simulation, and
Optimization)

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)

## USAGE
Quick installation instructions for CPU-based machines. For GPU execution, see
file [README_GPU.md](README_GPU.md). Go to the directory where this file is. 

Using "make"
(`[]` means "optional"):
```
./configure [options]
make all
```
"make" alone prints a list of acceptable targets. Optionally,
`make -jN` runs parallel compilation on `N` processors.
Link to binaries are found in bin/.

Using "CMake" (v.3.14 or later):

```
mkdir ./build
cd ./build
cmake -DCMAKE_Fortran_COMPILER=mpif90 -DCMAKE_C_COMPILER=mpicc [-DCMAKE_INSTALL_PREFIX=/path/to/install] ..
make [-jN]
[make install]
```
Although CMake has the capability to guess compilers, it is strongly recommended to specify
the intended compilers or MPI compiler wrappers as `CMAKE_Fortran_COMPILER` and `CMAKE_C_COMPILER`.
"make" builds all targets. Link to binaries are found in build/bin.
If `make install` is invoked, directory `CMAKE_INSTALL_PREFIX`
is prepended onto all install directories.

For more information, see the general documentation in directory Doc/, 
package-specific documentation in \*/Doc/, and the web site 
http://www.quantum-espresso.org/. Technical documentation for users and
developers 
can be found on [Wiki page on gitlab](https://gitlab.com/QEF/q-e/-/wikis/home).

## PACKAGES

- PWscf: structural optimisation and molecular dynamics on the electronic ground state, with self-consistent solution of DFT equations;
- CP: Car-Parrinello molecular dynamics;
- PHonon: vibrational and dielectric properties from DFPT (Density-Functional Perturbation Theory);
- TD-DFPT: spectra from Time-dependent DFPT;
- HP: calculation of Hubbard parameters from DFPT;
- EPW: calculation of electron-phonon coefficients, carrier transport, phonon-limited superconductivity and phonon-assisted optical processes;
- PWCOND: ballistic transport;
- XSpectra: calculation of X-ray absorption spectra;
- PWneb: reaction pathways and transition states with the Nudged Elastic Band method;
- GWL: many-body perturbation theory in the GW approach using ultra-localised Wannier functions and Lanczos chains;
- QEHeat: energy current in insulators for thermal transport calculations in DFT.
- KCW: Koopmans-compliant functionals in a Wannier representation

## Modular libraries
The following libraries have been isolated and partially encapsulated in view of their release for usage in other codes as well:

- UtilXlib: performing basic MPI handling, error handling, timing handling.
- FFTXlib: parallel (MPI and OpenMP) distributed three-dimensional FFTs, performing also load-balanced distribution of data (plane waves, G-vectors and real-space grids) across processors.
- LAXlib: parallel distributed dense-matrix diagonalization, using ELPA, SCALapack, or a custom algorithm.
- KS Solvers: parallel iterative diagonalization for the Kohn-Sham Hamiltonian (represented as an operator),using block Davidson and band-by-band or block Conjugate-Gradient algorithms.
- LRlib: performs a variety of tasks connected with (time-dependent) DFPT, to be used also in connection with Many-Body Perturbation Theory.
- upflib: pseudopotential-related code.
- devXlib: low-level utilities for GPU execution

## Contributing
Quantum ESPRESSO is an open project: contributions are welcome.
Read the [Contribution Guidelines](CONTRIBUTING.md) to see how you
can contribute.

## LICENSE

All the material included in this distribution is free software;
you can redistribute it and/or modify it under the terms of the GNU
General Public License as published by the Free Software Foundation;
either version 2 of the License, or (at your option) any later version.

These programs are distributed in the hope that they will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License
for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
675 Mass Ave, Cambridge, MA 02139, USA.


## Update for develop_epw

Compile hdf5 with `--enable-fortran --enable-parallel` option. Then compile QE with the following steps:

```
git clone --single-branch --branch develop_epw https://github.com/krishnaa423/q-e.git
cd q-e
rm -rf ./external/devxlib ./external/mbd
./configure --prefix=<your_install_location> --enable-parallel --with-hdf5=yes
make all
make epw
make install
```

The configure script should list the location of HDF5_LIBS at the end. If not, HDF5 has not been
detected, and you must specify the location of the hdf5 libraries manually. You might also have to
specify the location of the HDF5 include paths. For more info, you can look at the QE user guide. 

Once QE has been compiled with HF5, make sure `epw.in` has the following options set to get 
the electron-phonon matrix elements on the coarse grid. 

```
epbwrite = .true.
```

This will write the a file `<QE_prefix>_elph_<pool_id>.h5` for each pool, with the following datasets. 

The datasets shape order is given assuming one reading the array in C order (this is the order
if you read the hdf5 file using the python library). 

- atomic_masses. Size is (3*nat). Rydberg atomic mass units (see section on units). 
- dynmat_real, dynmat_imag. Size is (nqs, nmodes, nmodes). Entry is (q, j, i). Assuming Ry^2.
- elph_cart_real, elph_cart_imag. Size is (nqs, 3*nat, nks_each_pool, nbands, nbands). Entry is (q, s\alpha, k, j, i). Not sure. 
- elph_nu_real, elph_nu_imag. Size is (nqs, nmodes, nks_each_pool, nbands, nbands). Entry is (q, nu, k, j, i). Ry. 
- ph_eigs. Size is (nqs, nmodes). Entry is (q, nu). Ry. 
- ph_evecs_real, ph_evecs_imag. Size is (nmodes, nqs, 3*nat). Entry is (nu, q, s\alpha). No units.   

### Equations and notation

Notation for the equations is:

- Unit cell index: $l$
- Atomic index: $s \alpha$
- Phonon mode: $\nu$
- Band index: $i, j$
- k-point: $k$
- q-point: $q$

Assumed expressions for the datasets written. 



Dynamical matrix:

$$
\Phi_{\alpha \beta} = \frac{\partial^2 V_{dft}}{ \partial R_{i\alpha} \partial R_{j \beta}}
$$

$$
D_{s\alpha, s'\alpha'}(q) = \frac{1}{\sqrt{M_s M_{s'}}} \sum_{l} \Phi_{\alpha \alpha'} (l s, 0 s') e^{-i q (R_{ls}^{0} - R_{0s'}^{0})} 
$$

Phonon eigenvectors and eigenvalues:

$$
D_{s\alpha, s'\alpha'}(q) \eta_{s' \alpha'} (q \nu) = \omega_{q \nu}^2 \eta_{s\alpha} (q \nu)
$$

$\eta$ is the eigenvector, $\omega$ is the eigenvalue.

Transformation factor definition:

$$
A_{s\alpha, q \nu} = \frac{\eta_{s\alpha} (q \nu)}{\sqrt{2 M_{s} \omega_{q \nu}}}
$$

Electron-phonon matrix elements in phonon mode basis:

$$
g_{i, j} (k, \nu, q) = \sum_{s\alpha} A_{s\alpha, q \nu} \langle i k + q| \frac{ \partial V_{dft} }{\partial u_{q s \alpha}} | j k \rangle
$$

### Units

For now, i'm assuming most of them are written in Rydberg atomic units. In these units, 

- $\hbar = 1$. 
- $m = \frac{1}{2}$. 
- $e = \sqrt{2}$. 
- $4 \pi \epsilon_0 = 1$. Not entirely sure about this one.  

With the above 4 choices, we get units for length, time, mass, charge. 

### Things to verify in develop_epw

- Are the units correct?
- Is the band dimension the slimmed down version?
- Is the divisionn by mass and frequency done at the correct times? 


