# Copyright (c) 2016 Sebastian Weber, Henri Menke. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Use OpenMP

find_package(OpenMP REQUIRED)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

if(CMAKE_BUILD_TYPE MATCHES Release)
  if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /Ox")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic -Ofast -pipe")
  endif()
endif()

# Add current directory to search path
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Add the executables

file(GLOB pairinteraction_SRCS *.cpp)
set(pairinteraction_matrixelement_SRCS ${pairinteraction_SRCS})
set(wignerSymbols_SRCS wignerSymbols/src/wignerSymbols-cpp.cpp)
list(REMOVE_ITEM pairinteraction_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/mainMatrixelement.cpp)
list(REMOVE_ITEM pairinteraction_matrixelement_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)

add_executable(pairinteraction-real ${pairinteraction_SRCS} ${wignerSymbols_SRCS})
add_executable(pairinteraction-complex ${pairinteraction_SRCS} ${wignerSymbols_SRCS})
add_executable(matrixelements ${pairinteraction_matrixelement_SRCS} ${wignerSymbols_SRCS})
set_target_properties(pairinteraction-complex PROPERTIES COMPILE_DEFINITIONS "USE_COMPLEX")

if(MSVC)
  # TODO set_source_files_properties(${wignerSymbols_SRCS} PROPERTIES COMPILE_FLAGS /wdSOMENUMBER) maybe /wd0169 or /wd0649
else()
  set_source_files_properties(${wignerSymbols_SRCS} PROPERTIES COMPILE_FLAGS -Wno-unused-parameter)
endif()

# Add external libraries

find_package(Eigen3 REQUIRED)
include_directories(SYSTEM ${EIGEN3_INCLUDE_DIR})

find_package(Sqlite3 REQUIRED)
include_directories(${SQLITE3_INCLUDE_DIR})
list(APPEND LIBRARIES ${SQLITE3_LIBRARY})

find_package(Boost COMPONENTS filesystem system program_options REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
list(APPEND LIBRARIES ${Boost_LIBRARIES})

find_package(GSL REQUIRED)
include_directories(${GSL_INCLUDE_DIRS})
list(APPEND LIBRARIES ${GSL_LIBRARIES})

target_link_libraries(pairinteraction-real ${LIBRARIES})
target_link_libraries(pairinteraction-complex ${LIBRARIES})
target_link_libraries(matrixelements ${LIBRARIES})

# Include test cases

add_subdirectory( unit_test )

# CPack

if( WITH_DMG )
  find_package(PythonInterp 3 REQUIRED)

  install(TARGETS pairinteraction-real RUNTIME DESTINATION .)
  install(TARGETS pairinteraction-complex RUNTIME DESTINATION .)
  install(TARGETS matrixelements RUNTIME DESTINATION .)
  
  set(bin1 \${CMAKE_INSTALL_PREFIX}/pairinteraction-real)
  set(bin2 \${CMAKE_INSTALL_PREFIX}/pairinteraction-complex)
  set(bin3 \${CMAKE_INSTALL_PREFIX}/matrixelements)
  
  install(CODE "execute_process(COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_MACOSX_GOODIES_PATH}/standalone.py ${bin1} ${bin2} ${bin3})")

else( )

  install(TARGETS pairinteraction-real RUNTIME DESTINATION share/pairinteraction/calc)
  install(TARGETS pairinteraction-complex RUNTIME DESTINATION share/pairinteraction/calc)
  install(TARGETS matrixelements RUNTIME DESTINATION share/pairinteraction/calc)

endif( )
