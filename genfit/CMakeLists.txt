If (Boost_Avail)
    INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIR})
    MESSAGE("-- Boost is found at: ${Boost_INCLUDE_DIR}")
Else (Boost_Avail)
    MESSAGE("-- Boost is not found at... ERROR")
EndIf (Boost_Avail)

# Set the build type. Possibilities are None, Debug, Release,
# RelWithDebInfo and MinSizeRel
#SET(CMAKE_BUILD_TYPE Debug)

# either, the environment variable RAVEPATH has to be specified, and RAVE's pkgConfig will be used to determine everything we need
# or, RAVE_LDFLAGS, RAVE_INCLUDE_DIRS and RAVE_CFLAGS have to be set via the command-line

IF(DEFINED ENV{RAVEPATH})
	MESSAGE("Environment variable RAVEPATH is set. Try to build GFRave.")
	# get compiler flags from rave
	INCLUDE(FindPkgConfig)
	SET(ENV{PKG_CONFIG_PATH} $ENV{RAVEPATH})
	pkg_search_module(RAVE rave REQUIRED)

	SET(RAVE True)
ELSEIF(DEFINED RAVE_LDFLAGS)
	IF(DEFINED RAVE_INCLUDE_DIRS)
		IF(DEFINED RAVE_CFLAGS)
			MESSAGE("RAVE_LDFLAGS, RAVE_INCLUDE_DIRS and RAVE_CFLAGS are set. Try to build GFRave.")
			SET(RAVE True)
		ENDIF()
	ENDIF()
ELSE()
   	MESSAGE("You did not define the environment variable RAVEPATH which is needed to find rave. GFRave will not be built.")
ENDIF()

IF(DEFINED RAVE)
	# turn comma separated lists into space separated strings
	string (REPLACE ";" " " RAVE_LDFLAGS_STR "${RAVE_LDFLAGS}")
	string (REPLACE ";" " " RAVE_INCLUDE_DIRS_STR "${RAVE_INCLUDE_DIRS}")
	string (REPLACE ";" " " RAVE_CFLAGS_STR "${RAVE_CFLAGS}")

	SET (CMAKE_CXX_FLAGS ${RAVE_CFLAGS_STR})

	SET(GF_INC_DIRS
		./GFRave/include/
	)

	AUX_SOURCE_DIRECTORY( ./GFRave/src  library_sources )
ENDIF()

# Set GENFIT include directories
SET(GF_INC_DIRS
	./core/include/
	./eventDisplay/include/
	./fields/include/
	./finitePlanes/include/
	./fitters/include/
	./GBL/include/
	./measurements/include/
	./trackReps/include/
	./utilities/include/
    ${GF_INC_DIRS}
)

IF (Boost_Avail)
    INSTALL( DIRECTORY ${GF_INC_DIRS}
    	DESTINATION ${INCLUDE_OUTPUT_DIRECTORY}
	 )
ENDIF()

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

# Uncomment for coverage analysis.
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
#SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --coverage")
#SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --coverage")

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag(-std=c++03 HAS_STD_FLAG)
check_cxx_compiler_flag(--std=c++03 HAS_STD_FLAG_ALTERNATIVE)
if (HAS_STD_FLAG)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++03")
elseif (HAS_STD_FLAG_ALTERNATIVE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++03")
else()
  message("Neither --std=c++03 nor -std=c++03 seem to work.  Moving on.")
endif()

# require proper c++, standard is C++03, flag reads: c++98
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wno-long-long -Wshadow -Werror=overloaded-virtual"   )
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -g3")
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3")
SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS} -O3 -g3")

# include directories
INCLUDE_DIRECTORIES(
	BEFORE
	${GF_INC_DIRS}
	${ROOT_INCLUDE_DIR})

# add a target to generate API documentation with Doxygen
FIND_PACKAGE(Doxygen)

if(DOXYGEN_FOUND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile @ONLY)

    ADD_CUSTOM_TARGET(doc
        ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/doc
        COMMENT "Generating API documentation with Doxygen" VERBATIM)
ELSE()
    MESSAGE( STATUS "Doxygen not installed or found -- API documentation can't be generated!" )
ENDIF(DOXYGEN_FOUND)

# declare the ROOT library path
LINK_DIRECTORIES(${ROOT_LIBRARY_DIR})

# add the list of source code files -- careful, when a new file is added!
AUX_SOURCE_DIRECTORY( ./core/src          library_sources )
AUX_SOURCE_DIRECTORY( ./eventDisplay/src  library_sources )
AUX_SOURCE_DIRECTORY( ./fields/src        library_sources )
AUX_SOURCE_DIRECTORY( ./finitePlanes/src  library_sources )
AUX_SOURCE_DIRECTORY( ./fitters/src       library_sources )
AUX_SOURCE_DIRECTORY( ./GBL/src           library_sources )
AUX_SOURCE_DIRECTORY( ./measurements/src  library_sources )
AUX_SOURCE_DIRECTORY( ./trackReps/src     library_sources )
AUX_SOURCE_DIRECTORY( ./utilities/src     library_sources )

SET(CORE_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsFinitePlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsFitterInfo.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsHMatrix.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsTrackRep.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/DetPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/FitStatus.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/MaterialProperties.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasuredStateOnPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementFactory.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementOnPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementProducer.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/StateOnPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/ThinScatterer.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/Track.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackCand.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackCandHit.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackPoint.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/FieldManager.h
  ${CMAKE_CURRENT_SOURCE_DIR}/finitePlanes/include/RectangularFinitePlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsFitter.h
  ${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsBField.h
)
ROOT_GENERATE_DICTIONARY("${CORE_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/src/coreLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/coreRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/coreRootDict.cc" "${library_sources}")

SET(EVENTDISPLAY_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/include/EventDisplay.h
)
ROOT_GENERATE_DICTIONARY("${EVENTDISPLAY_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/src/eventDisplayLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/eventDisplayRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/fitters/include;${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/eventDisplayRootDict.cc" "${library_sources}")

SET(FITTERS_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/AbsKalmanFitter.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFittedStateOnPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitStatus.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/ReferenceStateOnPlane.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitterRefTrack.h

  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitter.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitterInfo.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/DAF.h
)
ROOT_GENERATE_DICTIONARY("${FITTERS_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/fitters/src/fittersLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/fittersRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/fitters/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/fittersRootDict.cc" "${library_sources}")

SET(GBL_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GFGbl.h
)
ROOT_GENERATE_DICTIONARY("${GBL_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/GBL/src/GBLLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/GBLRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/GBL/include"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/GBLRootDict.cc" "${library_sources}")

SET(MEASUREMENTS_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixPhi.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixU.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixUnit.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixV.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixUV.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/FullMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/PlanarMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/ProlateSpacepointMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/SpacepointMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WireMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WirePointMeasurement.h
  ${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WireTrackCandHit.h
)
ROOT_GENERATE_DICTIONARY("${MEASUREMENTS_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/measurements/src/measurementsLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/measurementsRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/measurements/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/measurementsRootDict.cc" "${library_sources}")

SET(TRACKREPS_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/RKTools.h
  ${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/RKTrackRep.h
  ${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/TGeoMaterialInterface.h
  ${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/MaterialEffects.h
)
ROOT_GENERATE_DICTIONARY("${TRACKREPS_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/trackReps/src/trackRepsLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/trackRepsRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/trackRepsRootDict.cc" "${library_sources}")

SET(FIELD_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/fields/include/ConstField.h
  ${CMAKE_CURRENT_SOURCE_DIR}/fields/include/BellField.h
)
ROOT_GENERATE_DICTIONARY("${FIELD_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/fields/src/fieldsLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/fieldsRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/fields/include"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/fieldsRootDict.cc" "${library_sources}")

SET(UTILITIES_DICTIONARY_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/HelixTrackModel.h
  ${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/MeasurementCreator.h
  ${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/mySpacepointDetectorHit.h
  ${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/mySpacepointMeasurement.h
)
ROOT_GENERATE_DICTIONARY("${UTILITIES_DICTIONARY_SOURCES}"
			 "${CMAKE_CURRENT_SOURCE_DIR}/utilities/src/utilitiesLinkDef.h"
			 "${CMAKE_CURRENT_BINARY_DIR}/utilitiesRootDict.cc"
			 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/utilities/include;${CMAKE_CURRENT_SOURCE_DIR}/measurements/include;${Boost_INCLUDE_DIR}"
			)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/utilitiesRootDict.cc" "${library_sources}")

if(DEFINED RAVE)
	SET(GFRAVE_DICTIONARY_SOURCES
	  ${CMAKE_CURRENT_SOURCE_DIR}/GFRave/include/GFRaveTrackParameters.h
	  ${CMAKE_CURRENT_SOURCE_DIR}/GFRave/include/GFRaveVertex.h
	)
	ROOT_GENERATE_DICTIONARY("${GFRAVE_DICTIONARY_SOURCES}"
				 "${CMAKE_CURRENT_SOURCE_DIR}/GFRave/src/GFRaveLinkDef.h"
				 "${CMAKE_CURRENT_BINARY_DIR}/GFRaveRootDict.cc"
				 "${CMAKE_CURRENT_SOURCE_DIR}/core/include;${CMAKE_CURRENT_SOURCE_DIR}/GFRave/include"
				)
	SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/GFRaveRootDict.cc" "${library_sources}")
endif()

# link ROOT libraries
if(DEFINED RAVE)
	set(LINK_DIRECTORIES ${ROOT_LIBRARY_DIR}
	# add what is needed for RAVE
  	)
else()
	set(LINK_DIRECTORIES ${ROOT_LIBRARY_DIR})
endif()

link_directories( ${LINK_DIRECTORIES})

Set(SRCS ${library_sources})
Set(HEADERS)
Set(LIBRARY_NAME genfit)
Set(DEPENDENCIES Cint Core Geom Matrix Eve Gui Physics MathCore EG)

Set(LINKDEF
${CMAKE_CURRENT_SOURCE_DIR}/core/src/coreLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/src/eventDisplayLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/fitters/src/fittersLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/GBL/src/GBLLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/measurements/src/measurementsLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/trackReps/src/trackRepsLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/fields/src/fieldsLinkDef.h
${CMAKE_CURRENT_SOURCE_DIR}/utilities/src/utilitiesLinkDef.h
)
if(DEFINED RAVE)
  Set(LINKDEF ${LINKDEF} ${CMAKE_CURRENT_SOURCE_DIR}/GFRave/src/GFRaveLinkDef.h)
endif()
ROOT_GENERATE_ROOTMAP()

# to not call again ROOT_GENERATE_ROOTMAP
Set(LINKDEF)

GENERATE_LIBRARY()


# the examples, build with "make tests"
ADD_CUSTOM_TARGET( tests )
MACRO( ADD_GENFIT_TEST _testname )
   ADD_EXECUTABLE( ${_testname} EXCLUDE_FROM_ALL ${ARGN} )
   ADD_DEPENDENCIES( tests  ${_testname} )
   TARGET_LINK_LIBRARIES( ${_testname} ${PROJECT_NAME}  ${ROOT_LIBS} -lGeom )
    #INSTALL( TARGETS ${_testname} DESTINATION ${EXECUTABLE_INSTALL_DIR})
ENDMACRO( ADD_GENFIT_TEST )

ADD_GENFIT_TEST( fitterTests               test/fitterTests/main.cc)
ADD_GENFIT_TEST( minimalFittingExample     test/minimalFittingExample/main.cc)
ADD_GENFIT_TEST( measurementFactoryExample test/measurementFactoryExample/main.cc)
ADD_GENFIT_TEST( streamerTest              test/streamerTest/main.cc)
ADD_GENFIT_TEST( unitTests                 test/unitTests/main.cc)
if(DEFINED RAVE)
  ADD_GENFIT_TEST( vertexingTest           test/vertexingTest/main.cc)
  ADD_GENFIT_TEST( vertexingTestRead       test/vertexingTest/read.cc)
endif()

# generate and install following configuration files
#GENERATE_PACKAGE_CONFIGURATION_FILES( genfitConfig.cmake genfitConfigVersion.cmake )
