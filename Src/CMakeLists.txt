# PoissonRecon as an application
cmake_minimum_required(VERSION 2.8)

project(PoissonRecon)

set(SAMPLE_PROJECT_VERSION_MAJOR 8)
set(SAMPLE_PROJECT_VERSION_MINOR 0)

set(COMMON_DEFINES _CRT_SECURE_NO_DEPRECATE _CRT_SECURE_NO_WARNINGS NOMINMAX)

# Try to enable OpenMP
option(POISSON_RECON_WITH_OPEN_MP "Compile PoissonRecon with OpenMP support" ON)
if (POISSON_RECON_WITH_OPEN_MP)
	find_package(OpenMP)
	if (OPENMP_FOUND)
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
		list(APPEND COMMON_DEFINES WITH_OPENMP)
	endif()
endif()

# Remove annoying "NO ARGUMENTS THAT DEPEND ON A TEMPLATE PARAMETER" errors with GCC
if(UNIX OR MINGW)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpermissive -w -std=c++11")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

FILE(GLOB LIBRARY_FILES_C "*.cpp")
FILE(GLOB LIBRARY_FILES_H "*.h" "*.inl")

FILE(GLOB POISSON_RECON_MAIN "PoissonRecon.cpp")
FILE(GLOB SURFACE_TRIMMER_MAIN "SurfaceTrimmer.cpp")

LIST(REMOVE_ITEM LIBRARY_FILES_C ${POISSON_RECON_MAIN} ${SURFACE_TRIMMER_MAIN})

add_library(PoissonReconLib STATIC ${LIBRARY_FILES_C} ${LIBRARY_FILES_H})
set_property(TARGET PoissonReconLib APPEND PROPERTY COMPILE_DEFINITIONS ${COMMON_DEFINES})

add_executable(PoissonRecon ${POISSON_RECON_MAIN})
target_link_libraries(PoissonRecon PoissonReconLib)
set_property(TARGET PoissonRecon APPEND PROPERTY COMPILE_DEFINITIONS ${COMMON_DEFINES})

add_executable(SurfaceTrimmer ${SURFACE_TRIMMER_MAIN})
target_link_libraries(SurfaceTrimmer PoissonReconLib)
set_property(TARGET SurfaceTrimmer APPEND PROPERTY COMPILE_DEFINITIONS ${COMMON_DEFINES})
