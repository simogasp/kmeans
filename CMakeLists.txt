cmake_minimum_required(VERSION 2.8)
project(kmeans)

option(WITH_CUDA   		"Build the cuda example"            ON)
option(WITH_OPENMP  	"Build the OpenMP example" 			ON)
option(WITH_MPI   		"Build the OpenMP example" 			OFF)

if(NOT DEFINED BLOCK_SHARED_MEM_OPTIMIZATION)
	set (BLOCK_SHARED_MEM_OPTIMIZATION=1)
endif()
add_definitions(-DBLOCK_SHARED_MEM_OPTIMIZATION=${BLOCK_SHARED_MEM_OPTIMIZATION})
set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS})


#------   sequential version -----------------------------------------
set(SEQ_SRC 	seq_main.c  
              	seq_kmeans.c 
	      		file_io.c    
	      		wtime.c
	)

add_executable(seq_main ${SEQ_SRC})


# ------------------------------------------------------------------------------
if(WITH_CUDA)
	find_package(CUDA QUIET)
	if(CUDA_FOUND)
		
		set(CUDA_C_SRC  cuda_main.cu cuda_io.cu cuda_wtime.cu cuda_kmeans.cu)
		set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} --ptxas-options=-v )
		cuda_add_executable(cuda_main ${CUDA_C_SRC})

	else()
		message("CUDA has not been found... Not building the sample")
		message("Try to set up CUDA_TOOLKIT_ROOT_DIR with the install location of your CUDA toolkit")
		set(WITH_CUDA OFF)
	endif(CUDA_FOUND)

endif(WITH_CUDA)

#------   OpenMP version -----------------------------------------
if(WITH_OPENMP)
	find_package(OpenMP QUIET)
	if(OPENMP_FOUND)
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
		set(OMP_SRC    	omp_main.c 
	      				omp_kmeans.c
	      				file_io.c
	      	)
		add_executable(omp_main ${OMP_SRC})

	else()
		message("Apparently your compiler does not support OpenMP... Not building the sample")
		set(WITH_OPENMP OFF)
	endif(OPENMP_FOUND)
endif(WITH_OPENMP)

#------   MPI version -----------------------------------------
if(WITH_MPI)
	find_package(MPI QUIET)
	if(MPI_FOUND)
		set(MPI_SRC    	mpi_main.c   
              			mpi_kmeans.c 
              			mpi_io.c     
	      				file_io.c
	      	)
		include_directories(${MPI_INCLUDE_PATH})
		add_executable(mpi_main ${MPI_SRC})
		target_link_libraries(mpi_main ${MPI_LIBRARIES})
		if(MPI_COMPILE_FLAGS)
			set_target_properties(mpi_main PROPERTIES COMPILE_FLAGS "${MPI_COMPILE_FLAGS}")
		endif()

		if(MPI_LINK_FLAGS)
			set_target_properties(mpi_main PROPERTIES LINK_FLAGS "${MPI_LINK_FLAGS}")
		endif()

	else()
		message("MPI libraries have not been found... Not building the sample")
		set(WITH_MPI OFF)
	endif(MPI_FOUND)
endif(WITH_MPI)