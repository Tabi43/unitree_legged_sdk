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

# Workaround for Boost.Bind placeholders deprecation in Boost 1.73+
add_compile_definitions(BOOST_BIND_GLOBAL_PLACEHOLDERS)

# Imported static library provided by the SDK
add_library(unitree_legged_sdk STATIC IMPORTED GLOBAL)
set_target_properties(unitree_legged_sdk PROPERTIES
  IMPORTED_LOCATION "${CMAKE_CURRENT_SOURCE_DIR}/lib/cpp/${ARCH}/libunitree_legged_sdk.a"
  INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
  # Propagate -no-pie to avoid relocation errors when linking a non-PIC static library into PIE executables
  INTERFACE_LINK_OPTIONS "-no-pie"
)

# Public include dir for examples / downstream users
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

# Common libs to link (pthread is required by the SDK loop)
set(EXTRA_LIBS unitree_legged_sdk pthread)

# Examples (kept as in the original layout)
add_executable(example_position example/example_position.cpp)
target_link_libraries(example_position PRIVATE ${EXTRA_LIBS})
install(TARGETS example_position RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_velocity example/example_velocity.cpp)
target_link_libraries(example_velocity PRIVATE ${EXTRA_LIBS})
install(TARGETS example_velocity RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_torque example/example_torque.cpp)
target_link_libraries(example_torque PRIVATE ${EXTRA_LIBS})
install(TARGETS example_torque RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_walk example/example_walk.cpp)
target_link_libraries(example_walk PRIVATE ${EXTRA_LIBS})
install(TARGETS example_walk RUNTIME DESTINATION lib/${PROJECT_NAME})

add_executable(example_joystick example/example_joystick.cpp)
target_link_libraries(example_joystick PRIVATE ${EXTRA_LIBS})
install(TARGETS example_joystick RUNTIME DESTINATION lib/${PROJECT_NAME})

# Install headers so downstream packages can include the SDK headers
install(DIRECTORY include/ DESTINATION include)

# (Optional) expose include dir to downstream packages
ament_export_include_directories(include)

ament_package()