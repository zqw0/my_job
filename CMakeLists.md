```c++
cmake_minimum_required(VERSION 3.14)
project(project_OpenCV)

set(CMAKE_CXX_STANDARD 14)
FIND_PACKAGE(OpenCV REQUIRED)
add_executable(project_OpenCV main.cpp opt.h)
TARGET_LINK_LIBRARIES(project_OpenCV ${OpenCV_LIBS})
```
