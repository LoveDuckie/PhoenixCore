############################################
#
# oenixCore CMake
#
############################################

CMAKE_MINIMUM_REQUIRED( VERSION 2.6 )

PROJECT( PhoenixCore )


############################################
# Disallow in-source builds
############################################

IF( "${PhoenixCore_SOURCE_DIR}" STREQUAL "${PhoenixCore_BINARY_DIR}" )
	MESSAGE( FATAL_ERROR "\nPhoenixCore likes to be built out-of-source.\n"
			"Please create a separate build directory and\n"
			"run cmake there ( cmake .. )" )
ENDIF ( "${PhoenixCore_SOURCE_DIR}" STREQUAL "${PhoenixCore_BINARY_DIR}" )

############################################
# Check Dependencies
############################################

MESSAGE ( "-- Checking Dependencies: " )

set(Boost_USE_STATIC_LIBS	ON)
set(Boost_USE_MULTITHREADED	 ON)
FIND_PACKAGE( Boost 1.38.0 REQUIRED COMPONENTS date_time thread )
IF( Boost_FOUND )
    MESSAGE( " Found Boost" )
else( Boost_FOUND )
    MESSAGE( FATAL_ERROR " Boost Not Found!" )
endif( Boost_FOUND )
include_directories( ${Boost_INCLUDE_DIRS} )
SET( LIBRARIES ${LIBRARIES} ${Boost_LIBRARIES} )
link_directories ( ${Boost_LIBRARY_DIRS} )

FIND_PACKAGE( OpenGL REQUIRED )
if( OPENGL_FOUND )
    MESSAGE( " Found OpenGL" )
else(OPENGL_FOUND)
    MESSAGE( FATAL_ERROR " OpenGL Not Found!" )
endif(OPENGL_FOUND)
SET( INCLUDES ${INCLUDES} ${OPENGL_INCLUDE_DIR} )
SET( LIBRARIES ${LIBRARIES} ${OPENGL_LIBRARIES} )

# FIND_PACKAGE( GLU REQUIRED )
# if( GLU_FOUND )
#     MESSAGE( " Found Glu" )
# else(GLU_FOUND)
#     MESSAGE( FATAL_ERROR " GLu Not Found!" )
# endif(GLU_FOUND)
# SET( INCLUDES ${INCLUDES} ${GLU_INCLUDE_DIR} )
# SET( LIBRARIES ${LIBRARIES} ${GLU_LIBRARIES} )

############################################
# Find GLFW
############################################

set( SEARCH_PATHS 
	${GLFW_PATH}
	${GLFW_PATH}/include
	${GLFW_PATH}/lib
	)
	
FIND_PATH( GLFW_INCLUDE_DIR GL/glfw.h PATHS ${SEARCH_PATHS} )
FIND_LIBRARY( GLFW_LIBRARY glfw PATHS ${SEARCH_PATHS} )

IF( GLFW_INCLUDE_DIR AND GLFW_LIBRARY )
	SET( GLFW_FOUND TRUE )
ENDIF( GLFW_INCLUDE_DIR AND GLFW_LIBRARY )

IF( GLFW_FOUND )
	MESSAGE (" Found GLFW ")
ELSE( GLFW_FOUND )
	MESSAGE( FATAL_ERROR "Could not find GLFW! Please provide the path \n"
						"containing the 'lib' and 'include' subdirectories \n"
						"that have the glfw header and static library via \n"
						"-DGLFW_PATH=\"path\"" )
ENDIF( GLFW_FOUND )

SET( INCLUDES ${INCLUDES} ${GLFW_INCLUDE_DIR} )
SET( LIBRARIES ${LIBRARIES} ${GLFW_LIBRARY} )

############################################
# Box2d
############################################

if( BOX2D )
    MESSAGE( "Box2d Enabled, Finding Library" )
    
    set( SEARCH_PATHS 
	${BOX2D_PATH}
	${BOX2D_PATH}/Include
	${BOX2D_PATH}/Library
	)
    
    FIND_PATH( BOX2D_INCLUDE_DIR Box2d.h PATHS ${SEARCH_PATHS} )
    FIND_LIBRARY( BOX2D_LIBRARY box2d PATHS ${SEARCH_PATHS} )
    
    if( BOX2D_INCLUDE_DIR AND BOX2D_LIBRARY )
	    SET( BOX2D_FOUND TRUE )
    endif()
	
    IF( BOX2D_FOUND )
	    MESSAGE (" Found Box2d ")
    ELSE( BOX2D_FOUND )
	    MESSAGE( FATAL_ERROR "Could not find Box2d! Please provide the path \n"
						    "containing the 'Library' and 'Include' subdirectories \n"
						    "that have the Box2d header and static library via \n"
						    "-DBOX2D_PATH=\"path\"" )
    ENDIF()

    SET( INCLUDES ${INCLUDES} ${BOX2D_INCLUDE_DIR} )
    
endif()

############################################
# Platform Stuff
############################################

#WINDOWS (MSVC)
if(WIN32)
	
	if(MSVC)
	    MESSAGE( "Building for MSVC\n" )
        ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS) # No stupid warning about fopen and sprintf
    endif()

	ADD_DEFINITIONS(-DWIN32 -DMSVC)
	

endif(WIN32)


# Linux
if(UNIX)

	MESSAGE( "-- Finding Unix Specific Libraries" )

	FIND_LIBRARY( XRANDR_LIBRARY Xrandr )
	IF( XRANDR_LIBRARY )
		MESSAGE(" Found Xrandr ")
	ELSE( XRANDR_LIBRARY)
		MESSAGE( FATAL_ERROR " Could not fix XRandR! \n"
				"GLFW requires XRandR on linux.\n" )
	ENDIF( XRANDR_LIBRARY )

	SET( LIBRARIES ${LIBRARIES} ${XRANDR_LIBRARY} )

endif(UNIX )

############################################
# Everything looks good.
############################################

############################################
# Change the output directories
############################################
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_OUTPUT_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

############################################
# Include Directories
############################################

include_directories( ${PhoenixCore_SOURCE_DIR}/source ${INCLUDES})

############################################
# Set the path for the data dir
############################################

ADD_DEFINITIONS( -DDATA_DIR="${PhoenixCore_SOURCE_DIR}/data/" )

############################################
# Include the source CMakeLists.txt
############################################

subdirs( source )

############################################
# Include the tests CMakeLists.txt
############################################

subdirs( tests )

############################################
# Include the examples CMakeLists.txt
############################################

subdirs( examples )