## Implement the Smagorinsky model in OF6
Original: https://pingpong.chalmers.se/public/pp/public_courses/course07056/published/1483976028698/resourceId/3473555/content/41433022-b456-4cd1-b467-4f7ef688841e/41433022-b456-4cd1-b467-4f7ef688841e.html?language=sv

As noticed above, the implementation is now templated (different from v2.4.0). That makes it difficult to only copy the directory of the turbulence model we are modifying. Instead we copy the entire TurbulenceModels directory, and we have the possibility to modify whatever we like in that entire directory structure:

	source /home/$USER/OpenFOAM/OpenFOAM-6.0/etc/bashrc 
	foam
	cp -r --parents src/TurbulenceModels $WM_PROJECT_USER_DIR
	cd $WM_PROJECT_USER_DIR/src/TurbulenceModels/
	
  
  
  Find all the Make directories:

$ find . -name Make
./incompressible/Make
./compressible/Make
./turbulenceModels/Make
  
  
	wmake libso
	
Move the tutorials to the desired folder e.g FOAM_RUN

	mv Tutorials $FOAM_RUN
