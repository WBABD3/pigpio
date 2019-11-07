cmake_minimum_required(VERSION 3.0)

project(pigpio)
find_package(Threads REQUIRED)

set(PIGPIO_FLAGS "-L. -lrt")
#set(DESTDIR ${CMAKE_CURRENT_SOURCE_DIR}/build/dest)

if(NOT DEFINED BUILD_SHARED_LIBS)
set(BUILD_SHARED_LIBS "ON")
endif(NOT DEFINED BUILD_SHARED_LIBS)

# libpigpio.(so|a)
add_library(pigpio pigpio.c command.c custom.cext)
target_link_libraries(pigpio Threads::Threads)
# libpigpiod_if.(so|a)
add_library(pigpiod_if pigpiod_if.c command.c )
set_target_properties(pigpio PROPERTIES VERSION 0.0.1 SOVERSION 0.0.1)
set_target_properties(pigpiod_if PROPERTIES VERSION 0.0.1 SOVERSION 0.0.1)
# libpigpiod_if2.(so|a)
add_library(pigpiod_if2 pigpiod_if2.c command.c )
set_target_properties(pigpiod_if2 PROPERTIES VERSION 0.0.1 SOVERSION 0.0.1)

# x_pigpio
add_executable(x_pigpio x_pigpio.c)
add_dependencies(x_pigpio pigpio)
target_link_libraries(x_pigpio
	${PIGPIO_FLAGS}
	-lpigpio Threads::Threads
)

# x_pigpiod_if
add_executable(x_pigpiod_if x_pigpiod_if.c)
add_dependencies(x_pigpiod_if pigpiod_if)
target_link_libraries(x_pigpiod_if
	${PIGPIO_FLAGS}
	-lpigpiod_if Threads::Threads
)

# x_pigpiod_if2
add_executable(x_pigpiod_if2 x_pigpiod_if2.c)
add_dependencies(x_pigpiod_if2 pigpiod_if2)
target_link_libraries(x_pigpiod_if2
	${PIGPIO_FLAGS}
	-lpigpiod_if2
	Threads::Threads
)

# pigpiod
add_executable(pigpiod pigpiod.c)
add_dependencies(pigpiod pigpio)
target_link_libraries(pigpiod
	${PIGPIO_FLAGS}
	-lpigpio
	Threads::Threads
	)

# pigs
add_executable(pigs pigs.c command.c)

# pig2vcd
add_executable(pig2vcd pig2vcd.c command.c)

# install
install(DIRECTORY
	DESTINATION ${DESTDIR}/opt/pigpio/cgi
	PATTERN ""
	PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
		GROUP_READ GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE
)

install(TARGETS pigpio pigpiod_if pigpiod_if2 pig2vcd pigpiod pigs
	LIBRARY DESTINATION ${DESTDIR}/lib
	RUNTIME DESTINATION ${DESTDIR}/bin
	ARCHIVE DESTINATION ${DESTDIR}/lib
)

install(FILES pigpio.h pigpiod_if.h pigpiod_if2.h
	DESTINATION ${DESTDIR}/include
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_1_SRC "*.1")
install(FILES ${man_1_SRC}
	DESTINATION ${DESTDIR}/man/man1
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_3_SRC "*.3")
install(FILES ${man_3_SRC}
	DESTINATION ${DESTDIR}/man/man3
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB setup_SRC "setup.py")
find_program(PYTHON2_FOUND python2)
if(PYTHON2_FOUND)
	install(CODE "execute_process(COMMAND cd ${CMAKE_SOURCE_DIR} && python2 ${setup_SRC} install)")
endif()
find_program(PYTHON3_FOUND python3)
if(PYTHON3_FOUND)
	install(CODE "execute_process(COMMAND cd ${CMAKE_SOURCE_DIR} && python3 ${setup_SRC} install)")
endif()

install(CODE "execute_process(COMMAND ldconfig)")

# uninstall
if(PYTHON2_FOUND)
	set(PY2_CMD python2 ${setup_SRC} install --record /tmp/pigpio > /dev/null)
	set(PY2_CMD ${PY2_CMD} && xargs rm -f < /tmp/pigpio > /dev/null)
endif()

if(PYTHON3_FOUND)
	set(PY3_CMD python3 ${setup_SRC} install --record /tmp/pigpio > /dev/null)
	set(PY3_CMD ${PY3_CMD} && xargs rm -f < /tmp/pigpio > /dev/null)
endif()

