INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/toolkit
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/asf
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/mpeg
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/ogg
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/ogg/vorbis
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/ogg/flac
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/flac
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/mpc
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/mp4
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/mpeg/id3v2
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/mpeg/id3v2/frames
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/wavpack
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/ogg/speex
		     ${CMAKE_CURRENT_SOURCE_DIR}/../../taglib/trueaudio
)


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/taglib_c.pc.cmake ${CMAKE_CURRENT_BINARY_DIR}/taglib_c.pc )
########### next target ###############

ADD_LIBRARY(tag_c SHARED tag_c.cpp)
if(ENABLE_STATIC)
    set_target_properties(tag_c PROPERTIES COMPILE_DEFINITIONS TAGLIB_STATIC)
endif(ENABLE_STATIC)

TARGET_LINK_LIBRARIES(tag_c  tag )

# On Solaris we need to explicitly add the C++ standard and runtime
# libraries to the libs used by the C bindings, because those C bindings
# themselves won't pull in the C++ libs -- and if a C application is
# using the C bindings then we get link errors.
CHECK_LIBRARY_EXISTS(Crun __RTTI___ "" HAVE_CRUN_LIB)
IF(HAVE_CRUN_LIB)
  # Which libraries to link depends critically on which
  # STL version is going to be used by your application
  # and which runtime is in use. While Crun is pretty much
  # the only game in town, the three available STLs -- Cstd,
  # stlport4 and stdcxx -- make this a mess. The KDE-Solaris
  # team supports stdcxx (Apache RogueWave stdcxx 4.1.3).
  
  # According to http://bugs.kde.org/show_bug.cgi?id=215225 the library can have the following two names:
  FIND_LIBRARY(ROGUEWAVE_STDCXX_LIBRARY NAMES stdcxx4 stdcxx)
  IF(NOT ROGUEWAVE_STDCXX_LIBRARY)
    MESSAGE(FATAL_ERROR "Did not find supported STL library (tried stdcxx4 and stdcxx)")
  ENDIF(NOT ROGUEWAVE_STDCXX_LIBRARY)
  TARGET_LINK_LIBRARIES(tag_c ${ROGUEWAVE_STDCXX_LIBRARY} Crun)
ENDIF(HAVE_CRUN_LIB)

SET_TARGET_PROPERTIES(tag_c PROPERTIES
  VERSION 0.0.0
  SOVERSION 0
  DEFINE_SYMBOL MAKE_TAGLIB_C_LIB
  INSTALL_NAME_DIR ${LIB_INSTALL_DIR}
  )
INSTALL(TARGETS tag_c
	LIBRARY DESTINATION ${LIB_INSTALL_DIR}
	RUNTIME DESTINATION ${BIN_INSTALL_DIR}
	ARCHIVE DESTINATION  ${LIB_INSTALL_DIR}
)


########### install files ###############

INSTALL( FILES  ${CMAKE_CURRENT_BINARY_DIR}/taglib_c.pc DESTINATION ${LIB_INSTALL_DIR}/pkgconfig)
INSTALL( FILES  tag_c.h DESTINATION ${INCLUDE_INSTALL_DIR}/taglib)
