cmake_minimum_required(VERSION 3.24)

project("test-module")

include(GNUInstallDirs)

add_library(${PROJECT_NAME} "")
target_include_directories(${PROJECT_NAME}
        PUBLIC $<INSTALL_INTERFACE:${CmAKE_INSTALL_INCLUDEDIR}>)
target_sources(${PROJECT_NAME} PRIVATE "./src/Module.cpp")

set(EXPORT_ALL_TO_TARGET "${PROJECT_NAME}_alltargets")
install(TARGETS ${PROJECT_NAME}
        EXPORT  ${EXPORT_ALL_TO_TARGET}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/
)

set(project_config_filename "${PROJECT_NAME}Config.cmake")

include(CMakePackageConfigHelpers)
configure_package_config_file("project-config.cmake.in" ${project_config_filename} INSTALL_DESTINATION "share/${PROJECT_NAME}")
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${project_config_filename} DESTINATION share/${PROJECT_NAME})

install(EXPORT "${EXPORT_ALL_TO_TARGET}"
        NAMESPACE ${PROJECT_NAME}::
        FILE ${PROJECT_NAME}Targets.cmake
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}
)

# copy header-files
install(DIRECTORY "${CMAKE_SOURCE_DIR}/src/"
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
        FILES_MATCHING PATTERN "*.hpp*" PATTERN "*.hxx*" PATTERN "*.h*"
)

if (${BUILD_THE_TESTS})
    set(TEST_TARGET "test-build")
    add_executable("${TEST_TARGET}" "")
    target_sources(${TEST_TARGET} PRIVATE "./tests/main.cpp")
    target_include_directories(${TEST_TARGET} PRIVATE "./src")
    target_link_libraries(${TEST_TARGET} PRIVATE ${PROJECT_NAME})
endif ()
