CMakeLists.txt
```cmake
cmake_minimum_required(VERSION 3.21)  
project(learn20220401_1)  
  
set(CMAKE_CXX_STANDARD 14)  
set(CMAKE_AUTOMOC ON)  
set(CMAKE_AUTORCC ON)  
set(CMAKE_AUTOUIC ON)  
  
set(CMAKE_PREFIX_PATH "D:/Qt/6.2.4/mingw_64/lib/cmake")  
set(BOOST_ROOT "D:/c++ library/boost_1_78_0/build")  
  
find_package(Qt6 COMPONENTS  
        Core        Gui        Widgets        REQUIRED)  
find_package(Boost)  
  
include_directories(${Boost_INCLUDE_DIRS})  
link_directories(${Boost_LIBRARY_DIRS})  
  
add_executable(learn20220401_1 main.cpp uuidtool.cpp uuidtool.h uuidtool.ui)  
target_link_libraries(learn20220401_1  
        Qt::Core        Qt::Gui        Qt::Widgets        )  
  
if (WIN32)  
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")  
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")  
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")  
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")  
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")  
        endif ()  
    endif ()  
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")  
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD  
                COMMAND ${CMAKE_COMMAND} -E make_directory  
                "$${PROJECT_NAME}>/plugins/platforms/")  
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD  
                COMMAND ${CMAKE_COMMAND} -E copy  
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"  
                "$${PROJECT_NAME}>/plugins/platforms/")  
    endif ()  
    foreach (QT_LIB Core Gui Widgets)  
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD  
                COMMAND ${CMAKE_COMMAND} -E copy  
                "${QT_INSTALL_PATH}/bin/Qt6${QT_LIB}${DEBUG_SUFFIX}.dll"  
                "$${PROJECT_NAME}>")  
    endforeach (QT_LIB)  
endif ()
```