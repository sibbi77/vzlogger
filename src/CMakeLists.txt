# -*- mode: cmake; -*-

#AM_CFLAGS = -Wall -D_REENTRANT -std=gnu99 $(DEPS_VZ_CFLAGS)
#AM_CPPFLAGS = -I $(top_srcdir)/include
#AM_LDFLAGS =

add_subdirectory(api)
add_subdirectory(protocols)

## local interface support
#####################################################################
if(LOCAL_SUPPORT)
  set(local_srcs local.cpp)
  include_directories(${MICROHTTPD_INCLUDE_DIR})
else(LOCAL_SUPPORT)
  set(local_srcs "")
endif(LOCAL_SUPPORT)

if(ENABLE_MQTT)
  set(mqtt_srcs mqtt.cpp)
else(ENABLE_MQTT)
  set(mqtt_srcs "")
endif(ENABLE_MQTT)

configure_file("${CMAKE_SOURCE_DIR}/src/gitSha1.cpp.in" "${CMAKE_BINARY_DIR}/gitSha1.cpp" @ONLY)

set(vzlogger_srcs
  vzlogger.cpp
  ltqnorm.cpp
  Meter.cpp
  ${CMAKE_BINARY_DIR}/gitSha1.cpp
  CurlSessionProvider.cpp
  PushData.cpp ../include/PushData.hpp
)

set(libvz_srcs
  Channel.cpp
  Config_Options.cpp
  threads.cpp
  Buffer.cpp
  Obis.cpp
  Options.cpp
  Reading.cpp
  exception.cpp
  ${local_srcs}
  ${mqtt_srcs}
  MeterMap.cpp
  )

add_library(vz ${libvz_srcs})

add_executable(vzlogger ${vzlogger_srcs})

target_link_libraries(vzlogger proto vz vz-api)
target_link_libraries(vzlogger ${JSON_LIBRARY})
if(SML_FOUND)
  target_link_libraries(vzlogger ${SML_LIBRARY})
endif(SML_FOUND)
if( MBUS_FOUND )
    target_link_libraries(vzlogger ${MBUS_LIBRARY})
endif( MBUS_FOUND )
target_link_libraries(vzlogger ${OCR_LIBRARIES})

if(LOCAL_SUPPORT)
target_link_libraries(vzlogger ${MICROHTTPD_LIBRARY})
endif(LOCAL_SUPPORT)

if(ENABLE_MQTT)
target_link_libraries(vzlogger ${MQTT_LIBRARY})
endif(ENABLE_MQTT)

target_link_libraries(vzlogger ${LIBGCRYPT})
target_link_libraries(vzlogger pthread m ${LIBUUID})
target_link_libraries(vzlogger dl)
if( TARGET )
  if( ${TARGET} STREQUAL "ar71xx")
    set_target_properties(vzlogger PROPERTIES LINK_FLAGS "-static")
    target_link_libraries(vzlogger ${OPENWRT_TARGET_DIR}/usr/lib/libssl.a)
    target_link_libraries(vzlogger ${OPENWRT_TARGET_DIR}/usr/lib/libcrypto.a)
    target_link_libraries(vzlogger ${OPENWRT_TARGET_DIR}/usr/lib/libz.a)
    target_link_libraries(vzlogger dl)
  endif( ${TARGET} STREQUAL "ar71xx")
endif( TARGET )
target_link_libraries(vzlogger ${CURL_STATIC_LIBRARIES} ${CURL_LIBRARIES} unistring ${GNUTLS_LIBRARIES} ${OPENSSL_LIBRARIES} )

# add programs to the install target
INSTALL(PROGRAMS
  ${CMAKE_CURRENT_BINARY_DIR}/vzlogger
  DESTINATION bin)
install(TARGETS vz
	ARCHIVE DESTINATION lib/static COMPONENT libraries
	LIBRARY DESTINATION lib COMPONENT libraries)
