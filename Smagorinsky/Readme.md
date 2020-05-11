## Implement the Smagorinsky model in OF6
Original: https://pingpong.chalmers.se/public/pp/public_courses/course07056/published/1483976028698/resourceId/3473555/content/41433022-b456-4cd1-b467-4f7ef688841e/41433022-b456-4cd1-b467-4f7ef688841e.html?language=sv

As noticed above, the implementation is now templated (different from v2.4.0). That makes it difficult to only copy the directory of the turbulence model we are modifying. Instead we copy the entire TurbulenceModels directory, and we have the possibility to modify whatever we like in that entire directory structure:

	source /home/$USER/OpenFOAM/OpenFOAM6/etc/bashrc 
	foam
	cp -r --parents src/TurbulenceModels $WM_PROJECT_USER_DIR
	cd $WM_PROJECT_USER_DIR/src/TurbulenceModels/
	
Find all the Make directories:

	find . -name Make
	./incompressible/Make
	./compressible/Make
	./turbulenceModels/Make
	
Change the location of the compiled files in all the Make/files files:

	sed -i s/FOAM_LIBBIN/FOAM_USER_LIBBIN/g ./*/Make/files

Compile (this will take 10min):

	./Allwmake
	
Now there are three new shared-object files in $WM_PROJECT_USER_DIR/platforms/linux64GccDPInt32Opt/lib:

	libcompressibleTurbulenceModels.so
	libincompressibleTurbulenceModels.so
	libturbulenceModels.so

Those will be used instead of the ones in the main installation, since the environment variable LD_LIBRARY_PATH points at that directory before pointing at the original directory $WM_PROJECT_DIR/platforms/linux64GccDPInt32Opt/lib. We can check this by:

	ldd `which simpleFoam`
	
which returns pointers to the files in your working directory (amongst a lot of other lines):

	libturbulenceModels.so => /home/oscfd/OpenFOAM/oscfd-4.x/platforms/linux64GccDPInt32Opt/lib/libturbulenceModels.so (0x00007fe8b5f79000)
	libincompressibleTurbulenceModels.so => /home/oscfd/OpenFOAM/oscfd-4.x/platforms/linux64GccDPInt32Opt/lib/libincompressibleTurbulenceModels.so (0x00007fe8b5aae000)
	libcompressibleTurbulenceModels.so => /home/oscfd/OpenFOAM/oscfd-4.x/platforms/linux64GccDPInt32Opt/lib/libcompressibleTurbulenceModels.so (0x00007fe8aff8b000)

We can now copy the Smagorinsky model and do our modifications (first rename files and class name):

	cp -r turbulenceModels/LES/Smagorinsky turbulenceModels/LES/SmagorinskyC
	mv turbulenceModels/LES/SmagorinskyC/Smagorinsky.C turbulenceModels/LES/SmagorinskyC/SmagorinskyC.C
	mv turbulenceModels/LES/SmagorinskyC/Smagorinsky.H turbulenceModels/LES/SmagorinskyC/SmagorinskyC.H
	sed -i s/Smagorinsky/SmagorinskyC/g turbulenceModels/LES/SmagorinskyC/SmagorinskyC.*

Now we need to make sure that our class is compiled. The usual way of doing that is to modify the Make/files file of the library, in this case $WM_PROJECT_USER_DIR/src/TurbulenceModels/turbulenceModels/Make/files. However, we can't find the original Smagorinsky.C file listed there. That is because the class is templated, and the compilation is instead taken care of by macros. We look for the string Smagorinsky except in the explicit turbulencce model classes and in the binary files:
	
	grep -r Smagorinsky . | grep -v Base | grep -v RAS | grep -v LES | grep -v lnInclude | grep -v linux
	
We find that for compressible turbulence modeling there is a file compressible/turbulentFluidThermoModels/turbulentFluidThermoModels.C that includes the lines:

	#include "Smagorinsky.H"
	makeLESModel(Smagorinsky);

The second line is a macro that makes sure that the Smagorinsky model is compiled as one of the alternatives of the templating. We simply add in that file, under the above lines:

	#include "SmagorinskyC.H"
	makeLESModel(SmagorinskyC)
	
Try typing ./Allwmake and it will complain that it can't find the file named SmagorinskyC.H. The reason is that the linking in the lnInclude directory was done without that file being present. We can update the linking by:

	wmakeLnInclude -u turbulenceModels
	
Try compiling again:

	./Allwmake
	
Notice that it will not be shown in the output of the compilation that the SmagorinskyC model is compiled, but it is.

Make sure that it worked by running the simpleFoam/pitzDaily case.
Look at the start of that log file and see that the SmagorinskyC model is used.	

## Now we will introduce how to recompile the .C file. 

Try compiling with 

	./Allwmake, 

and it will not recognize that we did any modifications. That is because we modified a file that is not listed in any Make/files file. The way we added the compilation of the SmagorinskyC model was to modify the file compressible/turbulentFluidThermoModels/turbulentFluidThermoModels.C. The model will thus be recompiled if that file is recompiled, i.e. type:

	touch compressible/turbulentFluidThermoModels/turbulentFluidThermoModels.C
	./Allwmake

Test using the simpleFoam/pitzDaily case.
