cmake_minimum_required(VERSION 2.8.8)
project(CatalystCxxNonOverlappingAMRExample)

find_package(ParaView 4.1 REQUIRED COMPONENTS vtkPVPythonCatalyst)
include("${PARAVIEW_USE_FILE}")
set(Adaptor_SRCS
  FEAdaptor.cxx
  )
add_library(CxxNonOverlappingAMRExampleAdaptor ${Adaptor_SRCS})
target_link_libraries(CxxNonOverlappingAMRExampleAdaptor vtkPVPythonCatalyst)
add_definitions("-DUSE_CATALYST")

add_executable(CxxNonOverlappingAMRExample FEDriver.cxx)
target_link_libraries(CxxNonOverlappingAMRExample CxxNonOverlappingAMRExampleAdaptor)

option(BUILD_TESTING "Build Testing" OFF)
# Setup testing.
if (BUILD_TESTING)
  include(CTest)
  add_test(CxxNonOverlappingAMRExampleTest CxxNonOverlappingAMRExample ${CMAKE_CURRENT_SOURCE_DIR}/SampleScripts/feslicescript.py)
endif()
