if(CMAKE_VERSION VERSION_GREATER 3.11)
  include(FetchContent)
  FetchContent_Declare(pybind11
    GIT_REPOSITORY https://github.com/pybind/pybind11
    GIT_TAG v2.9.2)
  FetchContent_GetProperties(pybind11)
  if(NOT pybind11_POPULATED)
    FetchContent_Populate(pybind11)
    add_subdirectory(${pybind11_SOURCE_DIR} ${pybind11_BINARY_DIR})

    #pybind11_add_module(cpp2pybind11 cpp2pybind11.cpp) # BUG: might not work out of the box on OSX with conda: https://github.com/pybind/pybind11/issues/3081
    add_library(cpp2pybind11 MODULE cpp2pybind11.cpp)
    target_link_libraries(cpp2pybind11 PRIVATE pinocchio_pywrap pybind11::module)

    SET_TARGET_PROPERTIES(cpp2pybind11
      PROPERTIES
      PREFIX ""
      SUFFIX ${PYTHON_EXT_SUFFIX}
    )

    IF(BUILD_WITH_OPENMP_SUPPORT AND NOT LINK_PYTHON_INTERFACE_TO_OPENMP)
      TARGET_LINK_LIBRARIES(cpp2pybind11 PRIVATE ${OpenMP_CXX_LIBRARIES})
    ENDIF()

    if(CMAKE_CXX_STANDARD LESS 14)
      message(STATUS "CXX_STANDARD for cpp2pybind11 changed from ${CMAKE_CXX_STANDARD} to 14")
      set_target_properties(cpp2pybind11 PROPERTIES CXX_STANDARD 14)
    endif()

    if(WIN32)
      target_compile_definitions(cpp2pybind11 PRIVATE -DNOMINMAX)
    endif(WIN32)

    add_python_unit_test("test-py-cpp2pybind11" "unittest/python/pybind11/test-cpp2pybind11.py" "bindings/python" "unittest/python/pybind11")
  endif()
endif()
