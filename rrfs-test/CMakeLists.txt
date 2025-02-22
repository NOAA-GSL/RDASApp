#!-------------------------------------------------------------------------
#!    NOAA/NCEP, National Centers for Environmental Prediction  RDASapp        !
#!-------------------------------------------------------------------------

include(funcs.cmake)

### Define CTests here ###

# FV3-JEDI tests
set(rrfs_fv3jedi_tests)
add_to_dictionary(rrfs_fv3jedi_tests "rrfs_fv3jedi_2024052700_Ens3Dvar"         "fv3jedi_var.x")
add_to_dictionary(rrfs_fv3jedi_tests "rrfs_fv3jedi_2024052700_getkf_observer"   "fv3jedi_letkf.x")
add_to_dictionary(rrfs_fv3jedi_tests "rrfs_fv3jedi_2024052700_getkf_solver"     "fv3jedi_letkf.x")

# MPAS-JEDI tests 
set(rrfs_mpasjedi_tests)
add_to_dictionary(rrfs_mpasjedi_tests "rrfs_mpasjedi_2024052700_Ens3Dvar"       "mpasjedi_variational.x")
add_to_dictionary(rrfs_mpasjedi_tests "rrfs_mpasjedi_2024052700_getkf_observer" "mpasjedi_enkf.x")
add_to_dictionary(rrfs_mpasjedi_tests "rrfs_mpasjedi_2024052700_getkf_solver"   "mpasjedi_enkf.x")
add_to_dictionary(rrfs_mpasjedi_tests "rrfs_mpasjedi_2024052700_bumploc"        "mpasjedi_error_covariance_toolbox.x")

message(STATUS "Use test data from rrfs-test-data repository")
set (rrfs-test_data_local "${CMAKE_SOURCE_DIR}/rrfs-test-data/")
set (src_yaml "${CMAKE_SOURCE_DIR}/rrfs-test/testinput")
# It's unclear if anyone is using the local data functionality, therefore comment it out
# for now and add in if someone complains / remove if completely if not.
#elseif (DEFINED ENV{FV3_JEDI_TESTFILES})
#  message(STATUS "Use test data from local directory $ENV{FV3_JEDI_TESTFILES}")
#  # A bit of guesswork here, I don't know if folks using this option stored it in the same directory structure
#  set (fv3-jedi_data_testinput_tier_1_local "$ENV{FV3_JEDI_TESTFILES}/fv3-jedi-data/testinput_tier_1")
if(FV3_DYCORE)

      message(STATUS "MACHINE_ID is " ${MACHINE_ID})
      set(RESTORE_MPI_ARGS ${MPI_ARGS})
      if("${MACHINE_ID}" STREQUAL "orion" OR "${MACHINE_ID}" STREQUAL "hercules")
         message(STATUS "Because MACHINE_ID is orion, adding exclusive MPI option" )
         set(MPI_ARGS "${MPI_ARGS} --exclusive")
      endif()   
      set(MPI_ARGS "${MPI_ARGS} --time=00:30:00")

      get_all_keys(rrfs_fv3jedi_tests caselist)
      foreach(case ${caselist})
         get_from_dictionary(rrfs_fv3jedi_tests case exe)
         set(casedir "${CMAKE_CURRENT_BINARY_DIR}/rundir-${case}")
         set(src_casedir "${rrfs-test_data_local}/rrfs-data_fv3jedi_2024052700")
         if (NOT EXISTS "${casedir}")
           file(MAKE_DIRECTORY ${casedir})
         endif()
         file(CREATE_LINK ${src_casedir}/DataFix ${casedir}/DataFix SYMBOLIC)
         file(CREATE_LINK ${src_casedir}/Data_static ${casedir}/Data_static SYMBOLIC)
         file(CREATE_LINK ${src_casedir}/INPUT ${casedir}/INPUT SYMBOLIC)
         file(CREATE_LINK ${src_casedir}/Data ${casedir}/Data SYMBOLIC)
         file(CREATE_LINK ${CMAKE_SOURCE_DIR}/rrfs-test/testoutput ${casedir}/testoutput SYMBOLIC)
         file(COPY ${src_yaml}/${case}.yaml DESTINATION ${casedir} )
         set(target_test ${case})
	 if ("${target_test}" STREQUAL "rrfs_fv3jedi_2024052700_getkf_solver")
           ecbuild_add_test( TARGET ${target_test}
                        MPI      160
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/rundir-${target_test}
                        ARGS     ${target_test}.yaml
			TEST_DEPENDS "rrfs_fv3jedi_2024052700_getkf_observer"
                        COMMAND  ${exe} )
	 else()
           ecbuild_add_test( TARGET ${target_test}
                        MPI      160
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/rundir-${target_test}
                        ARGS     ${target_test}.yaml
                        COMMAND  ${exe} )
	 endif()
      endforeach()
      set(MPI_ARGS ${RESTORE_MPI_ARGS})

endif()

if(MPAS_DYCORE)

      message(STATUS "MACHINE_ID is " ${MACHINE_ID})
      set(RESTORE_MPI_ARGS ${MPI_ARGS})
      if("${MACHINE_ID}" STREQUAL "orion" OR "${MACHINE_ID}" STREQUAL "hercules")
         message(STATUS "Because MACHINE_ID is orion, adding exclusive MPI option" )
         set(MPI_ARGS "${MPI_ARGS} --exclusive")
      endif()   
      set(MPI_ARGS "${MPI_ARGS} --ntasks-per-node=4")
      set(MPI_ARGS "${MPI_ARGS} --time=00:30:00")

      get_all_keys(rrfs_mpasjedi_tests caselist)
      foreach(case ${caselist})
	 get_from_dictionary(rrfs_mpasjedi_tests case exe)
         set(casedir "${CMAKE_CURRENT_BINARY_DIR}/rundir-${case}")
         set(src_casedir "${rrfs-test_data_local}/rrfs-data_mpasjedi_2024052700")
         if (NOT EXISTS "${casedir}")
           file(MAKE_DIRECTORY ${casedir})
         endif()
	 file(CREATE_LINK ${src_casedir}/data ${casedir}/data SYMBOLIC)
	 file(CREATE_LINK ${src_casedir}/graphinfo ${casedir}/graphinfo SYMBOLIC)
	 file(CREATE_LINK ${src_casedir}/stream_list ${casedir}/stream_list SYMBOLIC)
	 file(CREATE_LINK ${CMAKE_SOURCE_DIR}/rrfs-test/testoutput ${casedir}/testoutput SYMBOLIC)
	 file(COPY ${src_casedir}/streams.atmosphere DESTINATION ${casedir})
	 file(COPY ${src_casedir}/namelist.atmosphere DESTINATION ${casedir})
	 file(COPY ${src_casedir}/geovars.yaml DESTINATION ${casedir})
	 file(COPY ${src_casedir}/keptvars.yaml DESTINATION ${casedir})
	 file(COPY ${src_casedir}/obsop_name_map.yaml DESTINATION ${casedir})
	 file(GLOB bl_FILES "${src_casedir}/*.*BL")
	 file(COPY ${bl_FILES} DESTINATION ${casedir})
	 file(COPY ${src_yaml}/${case}.yaml DESTINATION ${casedir})
         set(target_test ${case})
	 if ("${target_test}" STREQUAL "rrfs_mpasjedi_2024052700_getkf_solver")
           ecbuild_add_test( TARGET ${target_test}
                        MPI      120
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/rundir-${target_test}
                        ARGS     ${target_test}.yaml
			TEST_DEPENDS "rrfs_mpasjedi_2024052700_getkf_observer"
                        COMMAND  ${exe} )
	 else()
           ecbuild_add_test( TARGET ${target_test}
                        MPI      120
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/rundir-${target_test}
                        ARGS     ${target_test}.yaml
                        COMMAND  ${exe} )
	 endif()
      endforeach()
      set(MPI_ARGS ${RESTORE_MPI_ARGS})

      ########################
      ### Bufr2ioda ctests ###
      ########################

      # YAML-based bufr2ioda
      set(case "rrfs_bufr2ioda_msonet")
      set(casedir "${CMAKE_CURRENT_BINARY_DIR}/rundir-${case}")
      set(src_casedir "${rrfs-test_data_local}/rrfs-data_mpasjedi_2024052700")
      if (NOT EXISTS "${casedir}")
         file(MAKE_DIRECTORY ${casedir})
      endif()
      file(CREATE_LINK ${src_casedir}/data ${casedir}/data SYMBOLIC)
      configure_file(${src_casedir}/data/obs/bufr/rap.t00z.prepbufr.tm00 ${casedir}/prepbufr COPYONLY)
      file(COPY ${src_yaml}/${case}.yaml DESTINATION ${casedir})
      set(MPI_ARGS "--time=00:05:00 --mem=64G")
      set(IODA_CONV_COMP_TOL "5.0e-4")
      ecbuild_add_test( TARGET  ${case}
	                MPI     1
                        TYPE    SCRIPT
                        COMMAND bash
          		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/rundir-${case}
           		ARGS    ${CMAKE_SOURCE_DIR}/rrfs-test/ush/iodaconv_comp_rrfs.sh
                                netcdf
                                "${CMAKE_BINARY_DIR}/bin/bufr2ioda.x ${case}.yaml"
                                data/obs/ioda_mesonet.nc
                                ioda_mesonet.nc 
			        ${IODA_CONV_COMP_TOL}
                        DEPENDS bufr2ioda.x )


endif()

