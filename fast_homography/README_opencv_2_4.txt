
This is the companion software to the paper:

"Speeding-up homography estimation in resource-limited computing devices"
Pablo Márquez-Neila, Javier López-Alberca, José M. Buenaposada, Luis Baumela

Structure of the software:
-------------------------------------------------------------------------------
We have taken the following files from OpenCV 2.4 source code:

OpenCV-2.4.0/modules/calib3d/src/modelest.cpp (Definition of class ModelEstimator2)
OpenCV-2.4.0/modules/calib3d/src/fundam.cpp (Definition of class cvHomographyEstimator 
and function cvFindHomography)

and we have re-factored the code (without changing it) into a function 
ransac_test::cvFindHomography and a class ransac_test::cvHomographyEstimator 
(fusion of OpenCV's 2.4 ModelEstimator2 and cvHomographyEstimator classes). This
way the code is easier to follow.

Our files in the example software are:

opencv_2.4_ransac_findhomography.h(.cpp) ---> ransac_test::cvFindHomography
opencv_2.4_ransac_modelest.h(.cpp) ---> ransac_test::cvHomographyEstimator

The main modification is the inclussion in method 
ransac_test::cvHomographyEstimator::runRANSAC of a call to isMinimalSetConsistent method:

          // Check whether the minimal set of four points satisfies the 
	  // same circular order in both images. It avoids to compute 
	  // the homography (runKernel) and checking for the 
	  // inliers if the constraint is not satisfied.
	  if ( use_geometrical_constraint )
            if ( ! isMinimalSetConsistent(ms1, ms2) )
	      continue;


Our constraints are implemented in the new methods: cvHomographyEstimator::weakConstraint
and cvHomographyEstimator::isMinimalSetConsistent.

We have chosen a test program from OpenCV 2.4 samples (find_obj.cpp). The program 
extracts SURF descriptors and then computes homographies using cvFindHomography.
We have modified the original program to test our cvFindHomography modifications. 

Testing the software
-------------------------------------------------------------------------------

The requirements to compile the software are: OpenCV 2.3 (minimum) and CMake. 

IMPORTANT: First be sure to choose whether to use our constraint or not by defining
USE_OUR_GEOMETRICAL_CONSTRAINT in find_obj.cpp or undefining it.

To compile:

$ cd constrained_ransac_homography_2_4
$ mkdir build
$ cd build
$ cmake ..
$ make

To test: 

$ cd constrained_ransac_homography
$ cd build
$ ./find_obj ../box3.png ../box_in_scene3.png

Please be sure of checking the execution time for ransac_test::cvFindHomography
printed in the console (times in the example are from an Intel core i5). There is a random 
component for the RANSAC algorithm that affects the execution times. In any case our 
constraint performs between 2x and 5x (some times even more) faster than the original OpenCV 
implementation:

  With #undef USE_OUR_GEOMETRICAL_CONSTRAINT:

        $./find_obj ../box3.png box_in_scene3.png 
        Object Descriptors: 1008
        Image Descriptors: 536
        Extraction time = 281,347ms
        Using approximate nearest neighbor search
        OpenCV Version of cvFindHomography time = 52,0687ms
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        Object Descriptors: 1008


  With #define USE_OUR_GEOMETRICAL_CONSTRAINT:

	$./find_obj ../box3.png box_in_scene3.png 
        Object Descriptors: 1008
        Image Descriptors: 536
        Extraction time = 281,599ms
        Using approximate nearest neighbor search
        Our Version of cvFindHomography time = 11,1907ms
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        Object Descriptors: 1008



