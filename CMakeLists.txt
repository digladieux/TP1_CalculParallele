cmake_minimum_required(VERSION 3.13)

project(TP1)

set(CMAKE_C_FLAGS "-fopenmp")

include_directories(.)

add_executable(hello_omp hello_omp.c)
add_executable(matrice prodMatriciel_dynamic_seq.c)

# target_compile_features(tp_5 PUBLIC cxx_std_17)


