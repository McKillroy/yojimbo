CMAKE_MINIMUM_REQUIRED(VERSION 3.16)

project(Yojimbo C CXX)


SET(YOJIMBO_INSTALL_DIR "${CMAKE_CURRENT_LIST_DIR}/install/${CMAKE_BUILD_TYPE}")
SET(YOJIMBO_TESTS TRUE )

############################################################
## Settings
############################################################
##
  if(NOT DEFINED YOJIMBO_TESTS)
      SET(YOJIMBO_TESTS FALSE)
  endif()
  if(NOT DEFINED CMAKE_VERBOSE_MAKEFILES)
      SET(CMAKE_VERBOSE_MAKEFILES ON)
  endif()
  if(NOT DEFINED YOJIMBO_INSTALL_DIR)
      SET(YOJIMBO_INSTALL_DIR CMAKE_INSTALL_PREFIX)
  endif()

############################################################
## External imports (Currently only Windows)
############################################################
##
  add_library( sodium     STATIC IMPORTED)
  set_target_properties(sodium     
      PROPERTIES
          IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/windows/sodium.lib     
          PUBLIC_HEADER     ${CMAKE_CURRENT_LIST_DIR}/windows/sodium.h
  )
  add_library( mbedtls    STATIC IMPORTED)
  set_target_properties(mbedtls    
      PROPERTIES
          IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/windows/mbedtls.lib    
  )
  add_library( mbedx509   STATIC IMPORTED)
  set_target_properties(mbedx509   
      PROPERTIES
          IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/windows/mbedx509.lib   
  )
  add_library( mbedcrypto STATIC IMPORTED)
  set_target_properties(mbedcrypto 
      PROPERTIES
          IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/windows/mbedcrypto.lib 
  )



############################################################
## Yojimbo Static Library
############################################################
##
  add_library(yojimbo STATIC
      yojimbo.cpp
      tlsf/tlsf.c
      netcode.io/netcode.c
      reliable.io/reliable.c
  )
  target_link_libraries( yojimbo sodium mbedtls mbedx509 mbedcrypto )
  target_compile_definitions( yojimbo PUBLIC NETCODE_ENABLE_TESTS=0 RELIABLE_ENABLE_TESTS=0 )      
  target_include_directories( yojimbo PUBLIC "." "netcode.io" "reliable.io" )  
  if(WIN32)
    target_include_directories( yojimbo PRIVATE "windows"  )
  elseif(UNIX)
    target_include_directories( yojimbo PRIVATE "/usr/local/include" )
  else()
    MESSAGE(FATAL "Inrecognized Operating System. I understand either WIN32 or UNIX to be set to true.")
  endif()  

############################################################
## Yojimbo Static TEST Library and Test App
############################################################
##
  if(${YOJIMBO_TESTS})

    ## library
    add_library(yojimbo_test STATIC
      yojimbo.cpp
      tlsf/tlsf.c
      netcode.io/netcode.c
      reliable.io/reliable.c
    )
    target_link_libraries( yojimbo_test sodium mbedtls mbedx509 mbedcrypto )    
    target_compile_definitions( yojimbo_test PUBLIC NETCODE_ENABLE_TESTS=1 RELIABLE_ENABLE_TESTS=1 )    
    target_include_directories( yojimbo_test PUBLIC "." "netcode.io" "reliable.io" )
  
    if(WIN32)
        target_include_directories( yojimbo_test PRIVATE "windows"  )
    elseif(UNIX)
        target_include_directories( yojimbo_test PRIVATE "/usr/local/include" )
    else()
        MESSAGE(FATAL "Inrecognized Operating System. I understand either WIN32 or UNIX to be set to true.")
    endif()
    
    
    ## executable
    add_executable(test test.cpp )
    target_link_libraries(test yojimbo_test)    
  
  endif()  
############################################################
## Client App Target
############################################################
##
  add_executable(client client.cpp )
  target_link_libraries( client yojimbo )

############################################################
## Server App Target
############################################################
##
  add_executable(server server.cpp )
  target_link_libraries( server yojimbo )

############################################################
## Secure Client App Target
############################################################
##
  add_executable(secure_client secure_client.cpp )
  target_link_libraries( secure_client yojimbo )

############################################################
## Secure Server App Target
############################################################
##
  add_executable(secure_server secure_server.cpp )
  target_link_libraries( secure_server yojimbo )

############################################################
## Client Server App Target
############################################################
##
  add_executable(client_server client_server.cpp )
  target_link_libraries( client_server yojimbo )

############################################################
## Loopback App Target
############################################################
##
  add_executable(loopback loopback.cpp )
  target_link_libraries( loopback yojimbo )

############################################################
## Soak App Target
############################################################
##
  add_executable(soak soak.cpp )
  target_link_libraries( soak yojimbo )

############################################################
## Install
############################################################
##
  if(YOJIMBO_TESTS)
      SET(TEST_INSTALL test)
  endif()
  
  install(
      TARGETS
          yojimbo
          ${TEST_INSTALL}
          client
          server
          secure_client
          secure_server
          client_server
          loopback
          soak
      RUNTIME  DESTINATION ${YOJIMBO_INSTALL_DIR}/bin
      LIBRARY  DESTINATION ${YOJIMBO_INSTALL_DIR}/lib
      ARCHIVE  DESTINATION ${YOJIMBO_INSTALL_DIR}/lib     
      INCLUDES DESTINATION ${YOJIMBO_INSTALL_DIR}/include  
  )
  install(FILES yojimbo.h shared.h DESTINATION ${YOJIMBO_INSTALL_DIR}/include  )

#### DONE ####