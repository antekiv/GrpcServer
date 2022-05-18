cmake_minimum_required (VERSION 3.10)

project(example VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(PostgreSQL_ROOT "C:/Program Files/PostgreSQL/14" CACHE PATH "The default path to PostgreSQL directory")
set(libpqxx_DIR "C:/Program Files (x86)/libpqxx" CACHE PATH "The default path to libpqxx directory")

find_package(libpqxx)

include_directories (${PostgreSQL_ROOT}/include ${libpqxx_ROOT}/include)
link_directories(${PostgreSQL_ROOT}/lib ${libpqxx_ROOT}/lib)

add_executable(main main.cpp)
target_link_libraries(main libpqxx::pqxx)