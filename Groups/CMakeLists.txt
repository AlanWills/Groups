cmake_minimum_required(VERSION 3.6)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

project(Groups)

#make sure call to project(Groups) is before this otherwise ${PROJECT_SOURCE_DIR} won't be expanded properly
set(CELESTE_DIR ${PROJECT_SOURCE_DIR}/../CelesteEngine)

#limit cmake to only make these configuration types
set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

#set platforms we will generate the project on
set(CMAKE_GENERATOR_PLATFORM "x64")

##------------------------------ Testing ------------------------------##

INCLUDE_EXTERNAL_MSPROJECT(TestUtils ${CELESTE_DIR}/TestUtils/TestUtils.vcxproj)
set_target_properties(TestUtils PROPERTIES FOLDER Testing)

#CMake doesn't seem to support this projects type, but that's ok cos we don't really use it anyway
#INCLUDE_EXTERNAL_MSPROJECT(CelTestSharp ${CELESTE_DIR}/CelTestSharp/CelTestSharp.csproj)
#set_target_properties(CelTestSharp PROPERTIES FOLDER Testing)

##------------------------------ Celeste ------------------------------##

#add include directory for Celeste
INCLUDE_DIRECTORIES(
	${CELESTE_DIR}/CelesteEngine/Headers
	${CELESTE_DIR}/Lua/Headers)

#add library directories for Celeste
LINK_DIRECTORIES(${CELESTE_DIR}/CelesteEngine/bin/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE})

#add link libraries for Celeste
LINK_LIBRARIES(CelesteEngine.lib)

#include Celeste Engine projects
INCLUDE_EXTERNAL_MSPROJECT(CelesteEngine ${CELESTE_DIR}/CelesteEngine/CelesteEngine.vcxproj)
set_target_properties(CelesteEngine PROPERTIES FOLDER CelesteEngine)

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineTestUtils ${CELESTE_DIR}/CelesteEngineTestUtils/CelesteEngineTestUtils.vcxproj)
set_target_properties(CelesteEngineTestUtils PROPERTIES FOLDER CelesteEngine)
add_dependencies(CelesteEngineTestUtils 
	CelesteEngine 
	TestUtils)

INCLUDE_EXTERNAL_MSPROJECT(TestCelesteEngine ${CELESTE_DIR}/TestCelesteEngine/TestCelesteEngine.vcxproj)
set_target_properties(TestCelesteEngine PROPERTIES FOLDER CelesteEngine)
add_dependencies(CelesteEngineTestUtils 
	CelesteEngine 
	CelesteEngineTestUtils 
	TestUtils)

INCLUDE_EXTERNAL_MSPROJECT(TestCelesteEngineRuntime ${CELESTE_DIR}/TestCelesteEngineRuntime/TestCelesteEngineRuntime.vcxproj)
set_target_properties(TestCelesteEngineRuntime PROPERTIES FOLDER CelesteEngine)
add_dependencies(TestCelesteEngineRuntime 
	CelesteEngine 
	CelesteEngineTestUtils 
	TestUtils)

##------------------------------ Dev Tools ------------------------------##

INCLUDE_EXTERNAL_MSPROJECT(GuidGenerator ${CELESTE_DIR}/GuidGenerator/GuidGenerator.vcxproj)
set_target_properties(GuidGenerator PROPERTIES FOLDER DevTools)

##------------------------------ Bindings ------------------------------##

INCLUDE_EXTERNAL_MSPROJECT(BindingsGenerator ${CELESTE_DIR}/BindingsGenerator/BindingsGenerator.vcxproj)
set_target_properties(BindingsGenerator PROPERTIES FOLDER Bindings)

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineBindingsGenerator ${CELESTE_DIR}/CelesteEngineBindingsGenerator/CelesteEngineBindingsGenerator.vcxproj)
set_target_properties(CelesteEngineBindingsGenerator PROPERTIES FOLDER Bindings)
add_dependencies(CelesteEngineBindingsGenerator 
	BindingsGenerator)
	
INCLUDE_EXTERNAL_MSPROJECT(BindingsKernel ${CELESTE_DIR}/BindingsKernel/BindingsKernel.csproj)
set_target_properties(BindingsKernel PROPERTIES FOLDER Bindings)

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineBindingsLib ${CELESTE_DIR}/CelesteEngineBindingsLib/CelesteEngineBindingsLib.csproj)
set_target_properties(CelesteEngineBindingsLib PROPERTIES FOLDER Bindings)

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineBindings ${CELESTE_DIR}/CelesteEngineBindings/CelesteEngineBindings.csproj)
set_target_properties(CelesteEngineBindings PROPERTIES FOLDER Bindings)
add_dependencies(CelesteEngineBindings 
	BindingsKernel
	CelesteEngineBindingsLib)
	
##------------------------------ Editor ------------------------------##

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineEditorLibrary ${CELESTE_DIR}/CelesteEngineEditorLibrary/CelesteEngineEditorLibrary.csproj)
set_target_properties(CelesteEngineEditorLibrary PROPERTIES FOLDER Editor)
add_dependencies(CelesteEngineEditorLibrary
	BindingsKernel
	CelesteEngineBindingsLib
	CelesteEngineBindings)

INCLUDE_EXTERNAL_MSPROJECT(CelesteEngineEditor ${CELESTE_DIR}/CelesteEngineEditor/CelesteEngineEditor.csproj)
set_target_properties(CelesteEngineEditor PROPERTIES FOLDER Editor)
add_dependencies(CelesteEngineEditor
	BindingsKernel
	CelesteEngineBindingsLib
	CelesteEngineBindings
	CelesteEngineEditorLibrary)

##---------------------------- Third Party ----------------------------##

#add include directory for Third Party
INCLUDE_DIRECTORIES(
	${CELESTE_DIR}/3rdParty/Include
	${CELESTE_DIR}/3rdParty/Include/Assimp
	${CELESTE_DIR}/3rdParty/Include/freetype2
	${CELESTE_DIR}/3rdParty/Include/ffmpeg)

#add library directories for Third Party
LINK_DIRECTORIES(${CELESTE_DIR}/3rdParty/Lib/${CMAKE_GENERATOR_PLATFORM}/${CMAKE_BUILD_TYPE})

#add link libraries for Third Party
LINK_LIBRARIES(
	swscale.lib
	avutil.lib
	avcodec.lib
	avformat.lib
	assimp-vc140-mt.lib
	liblua53.lib
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
add_dependencies(${PROJECT_NAME} CelesteEngine)
add_custom_command(
	TARGET ${PROJECT_NAME}
	PRE_BUILD
	COMMAND call "$(ProjectDir)Build Events/CopyDependencyFiles.bat" "$(TargetDir)" $(Configuration) $(Platform)
	VERBATIM)

set_target_properties (${PROJECT_NAME} PROPERTIES FOLDER ${PROJECT_NAME})
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT Groups)