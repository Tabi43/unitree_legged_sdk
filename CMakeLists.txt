cmake_minimum_required(VERSION 3.10)
project(unitree_legged_sdk)

find_package(ament_cmake REQUIRED)

# Detect architecture for precompiled static library
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64.*")
  set(ARCH amd64)
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64.*")
  set(ARCH arm64)
else()
  message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}. Expected x86_64 or aarch64.")
endif()

# Silence Boost.Bind global placeholders deprecation without touching sources
add_compile_definitions(BOOST_BIND_GLOBAL_PLACEHOLDERS)

# -------- Imported static library provided by the SDK (.a not PIC) --------
add_library(unitree_legged_sdk STATIC IMPORTED GLOBAL)
set_target_properties(unitree_legged_sdk PROPERTIES
  IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/lib/cpp/${ARCH}/libunitree_legged_sdk.a"
  INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
  # Avoid relocation errors when linking a non-PIC static library into PIE executables
  INTERFACE_LINK_OPTIONS "-no-pie"
)

# -------- Public wrapper INTERFACE target we can export via find_package --------
add_library(legged_sdk INTERFACE)
target_link_libraries(legged_sdk INTERFACE unitree_legged_sdk pthread)
target_include_directories(legged_sdk INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_link_options(legged_sdk INTERFACE -no-pie)

# -------- Examples (link to the INTERFACE wrapper) --------
add_executable(example_position example/example_position.cpp)
target_link_libraries(example_position PRIVATE legged_sdk)
install(TARGETS example_position RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_velocity example/example_velocity.cpp)
target_link_libraries(example_velocity PRIVATE legged_sdk)
install(TARGETS example_velocity RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_torque example/example_torque.cpp)
target_link_libraries(example_torque PRIVATE legged_sdk)
install(TARGETS example_torque RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_walk example/example_walk.cpp)
target_link_libraries(example_walk PRIVATE legged_sdk)
install(TARGETS example_walk RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_joystick example/example_joystick.cpp)
target_link_libraries(example_joystick PRIVATE legged_sdk)
install(TARGETS example_joystick RUNTIME DESTINATION lib/${PROJECT_NAME})

# -------- Install headers so downstream packages can include the SDK headers --------
install(DIRECTORY include/ DESTINATION include)

# -------- Export a CMake package config so consumers can do find_package(...) --------
install(TARGETS legged_sdk
  EXPORT unitree_legged_sdkTargets)

install(EXPORT unitree_legged_sdkTargets
  NAMESPACE unitree::
  DESTINATION share/${PROJECT_NAME}/cmake)

ament_export_targets(unitree_legged_sdkTargets)
ament_export_include_directories(include)

ament_package()
