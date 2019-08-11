cmake_minimum_required(VERSION 2.4)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

##------------------------------ Celeste ------------------------------##

set(CELESTE_DIR ${PROJECT_SOURCE_DIR}/../CelesteEngine)

#add include directory for Celeste
INCLUDE_DIRECTORIES(
	${CELESTE_DIR}/CelesteEngine/Headers
	${CELESTE_DIR}/Lua/Headers)

#add library directories for Celeste
LINK_DIRECTORIES(${CELESTE_DIR}/CelesteEngine/bin/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE})

#add link libraries for Celeste
LINK_LIBRARIES(CelesteEngine.lib)

##---------------------------- Third Party ----------------------------##

#add include directory for Third Party
INCLUDE_DIRECTORIES(
	${CELESTE_DIR}/3rdParty/Include
	${CELESTE_DIR}/3rdParty/Include/Assimp
	${CELESTE_DIR}/3rdParty/Include/freetype2
	${CELESTE_DIR}/3rdParty/Include/ffmpeg)

#add library directories for Third Party
LINK_DIRECTORIES(${CELESTE_DIR}/3rdParty/Lib/${CMAKE_GENERATOR_PLATFORM})

#add link libraries for Third Party
LINK_LIBRARIES(
	swscale.lib
	avutil.lib
	avcodec.lib
	avformat.lib
	assimp-vc140-mt.lib
	liblua53.a
	tinyxml2.lib
	alut.lib
	OpenAL32.lib
	SOIL.lib
	glew32.lib
	opengl32.lib
	glfw3dll.lib
	freetype.lib)

MACRO(SOURCE_GROUP_BY_FOLDER target)
  SET(SOURCE_GROUP_DELIMITER "/")
  SET(last_dir "")
  SET(files "")
  
  FOREACH(file ${SOURCES} ${${target}_HEADERS})
    file(RELATIVE_PATH relative_file "${PROJECT_SOURCE_DIR}" "${file}")
    GET_FILENAME_COMPONENT(dir "${relative_file}" PATH)
    IF (NOT "${dir}" STREQUAL "${last_dir}")
      IF (files)
        SOURCE_GROUP("${last_dir}" FILES ${files})
      ENDIF (files)
      SET(files "")
    ENDIF (NOT "${dir}" STREQUAL "${last_dir}")
    SET(files ${files} ${file})
    SET(last_dir "${dir}")
  ENDFOREACH(file)
  IF (files)
    SOURCE_GROUP("${last_dir}" FILES ${files})
  ENDIF (files)
ENDMACRO(SOURCE_GROUP_BY_FOLDER)

##------------------------------ Groups Game Project ------------------------------##

project(Groups)

#limit cmake to only make these configuration types
set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

#set platforms we will generate the project on
set(CMAKE_GENERATOR_PLATFORM "x64")

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE})

#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE})

#add include directory for this project
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/Headers)

#set(SOURCES 
#	${SOURCES}
#	PUT CPPs HERE)

file(GLOB_RECURSE SOURCES ${PROJECT_SOURCE_DIR}/Source/*.cpp)

SOURCE_GROUP_BY_FOLDER(${PROJECT_NAME} ${SOURCES})

add_executable(${PROJECT_NAME} ${SOURCES})

set_target_properties (${PROJECT_NAME} PROPERTIES FOLDER ${PROJECT_NAME})