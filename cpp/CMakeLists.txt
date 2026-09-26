cmake_minimum_required(VERSION 3.16)

project(robocon_cpp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)

add_executable(cpp_task
    src/main.cpp
    src/transform.cpp
)

target_include_directories(cpp_task PRIVATE
    include
    ${OpenCV_INCLUDE_DIRS}
)

target_link_libraries(cpp_task PRIVATE
    ${OpenCV_LIBS}
    Eigen3::Eigen
)
