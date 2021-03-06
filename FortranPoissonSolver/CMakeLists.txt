cmake_minimum_required(VERSION 2.6)
PROJECT(CatalystFortranPoissonSolver CXX Fortran)

set(SOURCES
  FEDriver.F90
  Box.F90
  SparseMatrix.F90
  ConjugateGradient.F90
  PoissonDiscretization.F90
)

set(USE_CATALYST ON CACHE BOOL "Link the simulator with Catalyst")
if(USE_CATALYST)
  find_package(ParaView 4.1 REQUIRED COMPONENTS vtkPVPythonCatalyst)
  include("${PARAVIEW_USE_FILE}")
  # we don't create a separate library for the adaptor here
  # since FEFortranAdaptor.F90 depends on Box.F90
  list(APPEND SOURCES FEFortranAdaptor.F90 FECxxAdaptor.cxx)
  add_definitions("-DUSE_CATALYST")
  if(NOT PARAVIEW_USE_MPI)
    message(SEND_ERROR "ParaView must be built with MPI enabled")
  endif()
else()
  find_package(MPI REQUIRED)
  include_directories(${MPI_Fortran_INCLUDE_PATH})
endif()

ADD_EXECUTABLE(FortranPoissonSolver ${SOURCES})
TARGET_LINK_LIBRARIES(FortranPoissonSolver ${MPI_Fortran_LIBRARIES})
if(USE_CATALYST)
  TARGET_LINK_LIBRARIES(FortranPoissonSolver vtkPVPythonCatalyst vtkParallelMPI)
endif()

option(BUILD_TESTING "Build Testing" OFF)
# Setup testing.
if (BUILD_TESTING)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/SampleScripts/coproc.py DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
  include(CTest)
  add_test(FortranPoissonSolverTest FortranPoissonSolver coproc.py)
endif()
