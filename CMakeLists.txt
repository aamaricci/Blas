CMAKE_MINIMUM_REQUIRED(VERSION 2.8.8)
PROJECT(blas Fortran)
SET(VERSION 3.8.0)

SET(BLASLIB blas)

SET(BUILD_TYPE "RELEASE" CACHE STRING "Build type")
MESSAGE(STATUS "BUILD_TYPE=${BUILD_TYPE}")
SET(CMAKE_BUILD_TYPE ${BUILD_TYPE} CACHE INTERNAL "Build type" FORCE)

# Add our local CMAKE modules to the module path
SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules")

INCLUDE(${CMAKE_MODULE_PATH}/ColorsMsg.cmake)

# Check that Fortran 90 is supported
IF(NOT CMAKE_Fortran_COMPILER_SUPPORTS_F90)
   MESSAGE(FATAL_ERROR "Fortran compiler does not support F90")
ENDIF(NOT CMAKE_Fortran_COMPILER_SUPPORTS_F90)

# Set some options the user may choose
OPTION(USE_MPI "Use the MPI library for parallelization" OFF)
OPTION(VERBOSE "Build a verbose Makefile" OFF)

# reSet compiler to MPI is required 
IF(USE_MPI)
  FIND_PACKAGE(MPI REQUIRED)
  SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
  SET(MPI_CPP "MPI")
  MESSAGE(STATUS "${Yellow}Set Fortran compiler FC to ${ColourReset}${CMAKE_Fortran_COMPILER}")
ELSE(USE_MPI)
  SET(MPI_CPP "")
ENDIF(USE_MPI)

# Set verbosity of the Makefile according to option
IF(VERBOSE)
  SET(CMAKE_VERBOSE_MAKEFILE ON)
ELSE(VERBOSE)
  SET(CMAKE_VERBOSE_MAKEFILE OFF)
ENDIF(VERBOSE)

# This executes code that sets the compile flags for DEBUG, RELEASE, and TESTING.
# This executes code that sets the compile flags for DEBUG, RELEASE, and TESTING.
# INCLUDE(${CMAKE_MODULE_PATH}/SetFortranFlags.cmake)
if(CMAKE_Fortran_COMPILER_ID MATCHES GNU) # this is gfortran
  SET(CMAKE_Fortran_MODDIR_FLAG   "-J")
  SET(CMAKE_Fortran_FLAGS         "-cpp -ffree-line-length-none") # -mcmodel=large  this is to remove the 2Gb limit of virtual memory allocation
  SET(CMAKE_Fortran_FLAGS_TESTING "-O2 -funroll-loops")
  SET(CMAKE_Fortran_FLAGS_DEBUG   "-O0 -p -g -Wsurprising -Waliasing -fwhole-file -fcheck=all -pedantic -fbacktrace -fbounds-check")
  SET(CMAKE_Fortran_FLAGS_RELEASE "-O3 -march=native -mtune=native -funroll-loops") 
  
elseif(CMAKE_Fortran_COMPILER_ID MATCHES Intel)
  SET(CMAKE_Fortran_MODDIR_FLAG   "-module ") #remember the ending white space here 
  SET(CMAKE_Fortran_FLAGS         "-fpp")
  SET(CMAKE_Fortran_FLAGS_TESTING "-O2 -ftz")
  SET(CMAKE_Fortran_FLAGS_DEBUG   "-p -O0 -g -fpe1 -warn -debug extended -traceback -check all,noarg_temp_created")
  SET(CMAKE_Fortran_FLAGS_RELEASE "-O3 -ftz")

elseif(CMAKE_Fortran_COMPILER_ID MATCHES G95)
  SET(CMAKE_Fortran_MODDIR_FLAG   "-fmod=")
  SET(CMAKE_Fortran_FLAGS         "-cpp")
  SET(CMAKE_Fortran_FLAGS_TESTING "-O1  -fsloppy-char")
  SET(CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g -ftrace=full")
  SET(CMAKE_Fortran_FLAGS_RELEASE "-O3")
  
elseif(CMAKE_Fortran_COMPILER_ID MATCHES PGI)
  SET(CMAKE_Fortran_MODDIR_FLAG   "-module ")
  SET(CMAKE_Fortran_FLAGS         "-")
  SET(CMAKE_Fortran_FLAGS         "")
  SET(CMAKE_Fortran_FLAGS_DEBUG   "-g -O0 -Mframe")
  SET(CMAKE_Fortran_FLAGS_RELEASE "-O3 -mcmodel=medium -fast -Munroll")
endif()




############################################################
# Define the actual files and folders that make up the build
############################################################
SET(USER_HOME $ENV{HOME})
SET(USER $ENV{USER})


#make sure FC_PLAT is lowecase
STRING(TOLOWER "${CMAKE_Fortran_COMPILER_ID}" FC_PLAT)

SET(PREFIX  "$ENV{HOME}/opt/${PROJECT_NAME}/${VERSION}/${FC_PLAT}" CACHE PATH "Prefix prepended to install directories")

SET(CMAKE_INSTALL_PREFIX "${PREFIX}" CACHE INTERNAL "Prefix prepended to install directories" FORCE)



# Define some directories
SET(LIB_SRC ${CMAKE_SOURCE_DIR}/src)
SET(LIB_ETC ${CMAKE_SOURCE_DIR}/etc)
SET(LIB_ENV ${CMAKE_SOURCE_DIR}/etc/environment_modules)
SET(LIB_BIN ${CMAKE_SOURCE_DIR}/bin)
SET(LIB_TMP_INC ${CMAKE_BINARY_DIR}/include)
SET(LIB_TMP_ETC ${CMAKE_BINARY_DIR}/etc)
SET(LIB_TMP_BIN ${CMAKE_BINARY_DIR}/bin)
SET(LIB_TARGET_LIB ${CMAKE_INSTALL_PREFIX}/lib)
SET(LIB_TARGET_INC ${CMAKE_INSTALL_PREFIX}/include)
SET(LIB_TARGET_ETC ${CMAKE_INSTALL_PREFIX}/etc)
SET(LIB_TARGET_BIN ${CMAKE_INSTALL_PREFIX}/bin)



# Have the .mod files placed in the INC folder
SET(CMAKE_Fortran_MODULE_DIRECTORY ${LIB_TMP_INC})
INCLUDE_DIRECTORIES(BEFORE ${CMAKE_Fortran_MODULE_DIRECTORY})



# Add a distclean target to the Makefile
ADD_CUSTOM_TARGET(distclean 
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_SOURCE_DIR}/distclean.cmake
)

set(SBLAS1 isamax.f sasum.f saxpy.f scopy.f sdot.f snrm2.f srot.f srotg.f sscal.f sswap.f sdsdot.f srotmg.f srotm.f)

set(CBLAS1 scabs1.f scasum.f scnrm2.f icamax.f caxpy.f ccopy.f cdotc.f cdotu.f csscal.f crotg.f cscal.f cswap.f csrot.f)
set(DBLAS1 idamax.f dasum.f daxpy.f dcopy.f ddot.f dnrm2.f drot.f drotg.f dscal.f dsdot.f dswap.f drotmg.f drotm.f)
set(ZBLAS1 dcabs1.f dzasum.f dznrm2.f izamax.f zaxpy.f zcopy.f zdotc.f zdotu.f zdscal.f zrotg.f zscal.f zswap.f zdrot.f)
set(CB1AUX isamax.f sasum.f saxpy.f scopy.f snrm2.f sscal.f)
set(ZB1AUX idamax.f dasum.f daxpy.f dcopy.f dnrm2.f dscal.f)
set(ALLBLAS  lsame.f xerbla.f)
set(SBLAS2 sgemv.f sgbmv.f ssymv.f ssbmv.f sspmv.f strmv.f stbmv.f stpmv.f strsv.f stbsv.f stpsv.f sger.f ssyr.f sspr.f ssyr2.f sspr2.f)
set(CBLAS2 cgemv.f cgbmv.f chemv.f chbmv.f chpmv.f ctrmv.f ctbmv.f ctpmv.f ctrsv.f ctbsv.f ctpsv.f cgerc.f cgeru.f cher.f chpr.f cher2.f chpr2.f)
set(DBLAS2 dgemv.f dgbmv.f dsymv.f dsbmv.f dspmv.f dtrmv.f dtbmv.f dtpmv.f dtrsv.f dtbsv.f dtpsv.f dger.f dsyr.f dspr.f dsyr2.f dspr2.f)
set(ZBLAS2 zgemv.f zgbmv.f zhemv.f zhbmv.f zhpmv.f ztrmv.f ztbmv.f ztpmv.f ztrsv.f ztbsv.f ztpsv.f zgerc.f zgeru.f zher.f zhpr.f zher2.f zhpr2.f)
set(SBLAS3 sgemm.f ssymm.f ssyrk.f ssyr2k.f strmm.f strsm.f)
set(CBLAS3 cgemm.f csymm.f csyrk.f csyr2k.f ctrmm.f ctrsm.f chemm.f cherk.f cher2k.f)
set(DBLAS3 dgemm.f dsymm.f dsyrk.f dsyr2k.f dtrmm.f dtrsm.f)
set(ZBLAS3 zgemm.f zsymm.f zsyrk.f zsyr2k.f ztrmm.f ztrsm.f zhemm.f zherk.f zher2k.f)

set(SINGLE  ${SBLAS1} ${ALLBLAS} ${SBLAS2} ${SBLAS3} )

set(DOUBLE ${DBLAS1} ${ALLBLAS} ${DBLAS2} ${DBLAS3} )

set(COMPLEX ${CBLAS1} ${CB1AUX} ${ALLBLAS} ${CBLAS2} ${CBLAS3} )

set(COMPLEX16 ${ZBLAS1} ${ZB1AUX} ${ALLBLAS} ${ZBLAS2} ${ZBLAS3} )

set(BLAS_LOCAL_SRC ${DOUBLE} ${COMPLEX16})

SET(BLAS_src ${BLAS_LOCAL_SRC})
ADD_LIBRARY(blas STATIC ${BLAS_src})


#Contains the functions used in the following:
#BUILD_ENV_MODULE
#BUILD_CONFIGVARS
#BUILD_PKCONFIG
INCLUDE(${CMAKE_MODULE_PATH}/PostBuildLibrary.cmake)


#Build the Library module for environment modules
SET(TMP_ENV_MODULE_FILE ${LIB_TMP_ETC}/modules/${PROJECT_NAME}/${FC_PLAT})
BUILD_ENV_MODULE(${TMP_ENV_MODULE_FILE})

#Build the user config script (sourced in user shell config file, i.e. .bashrc)
SET(TMP_CONFIGVARS_USER_FILE ${LIB_TMP_BIN}/library_config.sh)
BUILD_CONFIGVARS_USER(${TMP_CONFIGVARS_USER_FILE})

#Build the .pc (pkg-config) configuration file
SET(TMP_PKCONFIG_FILE ${LIB_TMP_ETC}/${PROJECT_NAME}.pc)
BUILD_PKCONFIG(${TMP_PKCONFIG_FILE})


INSTALL(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY}/ DESTINATION ${LIB_TARGET_INC})

INSTALL(TARGETS blas DESTINATION ${LIB_TARGET_LIB})

INSTALL(DIRECTORY ${LIB_TMP_ETC}/ DESTINATION ${LIB_TARGET_ETC})

INSTALL(FILES ${TMP_CONFIGVARS_USER_FILE} DESTINATION ${LIB_TARGET_BIN}/
  PERMISSIONS ${PERMISSION_777} SETUID)

INSTALL(FILES ${TMP_CONFIGVARS_GLOBAL_FILE} DESTINATION ${LIB_TARGET_BIN}/
  PERMISSIONS ${PERMISSION_777} SETUID)


MESSAGE( STATUS "${Yellow}To conclude installation:${ColourReset} 
$ make
$ make install
$ make post-install
")


ADD_CUSTOM_TARGET(
  post-install
  COMMAND "${CMAKE_COMMAND}" -P ${CMAKE_MODULE_PATH}/HomeInstall.cmake ${CMAKE_SOURCE_DIR} ${CMAKE_INSTALL_PREFIX} ${PROJECT_NAME} ${FC_PLAT} 
  WORKING_DIRECTORY
  "${CMAKE_BINARY_DIR}"
)






