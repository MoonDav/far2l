cmake_minimum_required (VERSION 3.0.2)

add_custom_target(bootstrap 
  DEPENDS copyright.inc farversion.inc farlang.templ lang.inc
)

message(STATUS "generating headers and languages")

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(LBITS 64)
	set(DIRBIT 64)
else ()
	set(LBITS 32)
	set(DIRBIT 32)
endif ()

set(BOOTSTRAP "${PROJECT_BINARY_DIR}/bootstrap")
set(SCRIPTS "${CMAKE_CURRENT_SOURCE_DIR}/scripts")
set(DEPENDENCIES
  "${SCRIPTS}/copyright.inc.m4"
  "${SCRIPTS}/farversion.m4"
  "${SCRIPTS}/tools.m4"
  "${SCRIPTS}/farlang.templ.m4"
)

set(HLFFILES
  "${BOOTSTRAP}/FarEng.hlf"
  "${BOOTSTRAP}/FarRus.hlf"
  "${BOOTSTRAP}/FarHun.hlf"
  "${BOOTSTRAP}/FarUkr.hlf"
)
set(LNGFILES
  "${BOOTSTRAP}/FarEng.lng"
  "${BOOTSTRAP}/FarRus.lng"
  "${BOOTSTRAP}/FarHun.lng"
  "${BOOTSTRAP}/FarUkr.lng"
)

add_custom_command(OUTPUT "${BOOTSTRAP}/copyright.inc"
   COMMAND ${M4} "${SCRIPTS}/copyright.inc.m4" | ${GAWK} -f "${SCRIPTS}/enc.awk" > "${BOOTSTRAP}/copyright.inc"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating header
)

add_custom_command(OUTPUT "${BOOTSTRAP}/farversion.inc"
   COMMAND ${M4} "${SCRIPTS}/farversion.inc.m4" > "${BOOTSTRAP}/farversion.inc"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating header
)

add_custom_command(OUTPUT "${BOOTSTRAP}/farlang.templ"
   COMMAND ${M4} "${SCRIPTS}/farlang.templ.m4" > "${BOOTSTRAP}/farlang.templ"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating language template
)

add_custom_command(OUTPUT "${BOOTSTRAP}/lang.inc"
   COMMAND "${SCRIPTS}/farlng.pl" "${BOOTSTRAP}/farlang.templ" "${BOOTSTRAP}"
   WORKING_DIRECTORY "${BOOTSTRAP}"
   DEPENDS "${BOOTSTRAP}/farlang.templ"
   COMMENT generating languages
)


add_custom_command(TARGET bootstrap
   COMMAND ${GAWK} -f "${SCRIPTS}/mkhlf.awk" "${SCRIPTS}/FarEng.hlf.m4" | ${M4} > "${BOOTSTRAP}/FarEng.hlf"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating help eng
)
add_custom_command(TARGET bootstrap
   COMMAND ${GAWK} -f "${SCRIPTS}/mkhlf.awk" "${SCRIPTS}/FarRus.hlf.m4" | ${M4} > "${BOOTSTRAP}/FarRus.hlf"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating help rus
)
add_custom_command(TARGET bootstrap
   COMMAND ${GAWK} -f "${SCRIPTS}/mkhlf.awk" "${SCRIPTS}/FarHun.hlf.m4" | ${M4} > "${BOOTSTRAP}/FarHun.hlf"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating help hun
)
add_custom_command(TARGET bootstrap
   COMMAND ${GAWK} -f "${SCRIPTS}/mkhlf.awk" "${SCRIPTS}/FarUkr.hlf.m4" | ${M4} > "${BOOTSTRAP}/FarUkr.hlf"
   WORKING_DIRECTORY "${SCRIPTS}"
   DEPENDS ${DEPENDENCIES}
   COMMENT generating help ukr
)

# older cmake versions does not support copying of multiple files,
# so we use simple stupid /bin/cp and wildcards

add_custom_command(TARGET bootstrap
   POST_BUILD
   DEPENDS "${LNGFILES}"
   COMMAND "cp" "${BOOTSTRAP}/*.lng" "${INSTALL_DIR}"
)

add_custom_command(TARGET bootstrap
   POST_BUILD
   DEPENDS "${HLFFILES}"
   COMMAND "cp" "${BOOTSTRAP}/*.hlf" "${INSTALL_DIR}"
)

add_custom_command(TARGET bootstrap
   POST_BUILD
   COMMAND "cp" "${CMAKE_CURRENT_SOURCE_DIR}/*.sh" "${INSTALL_DIR}"
   COMMAND "cp" "${CMAKE_CURRENT_SOURCE_DIR}/*.ini" "${INSTALL_DIR}"
)
