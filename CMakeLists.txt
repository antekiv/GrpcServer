cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

project(GrpcClientServerApp)
   
if(MSVC)
  add_definitions(-D_WIN32_WINNT=0x600)
endif()

find_package(Threads REQUIRED)

set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)
message(STATUS "Using protobuf ${Protobuf_VERSION}")

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION gRPC::grpc++_reflection)
if(CMAKE_CROSSCOMPILING)
  find_program(_PROTOBUF_PROTOC protoc)
else()
  set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
endif()

# Find gRPC installation
# Looks for gRPCConfig.cmake file installed by gRPC's cmake installation.
find_package(gRPC CONFIG REQUIRED)
message(STATUS "Using gRPC ${gRPC_VERSION}")

set(_GRPC_GRPCPP gRPC::grpc++)
if(CMAKE_CROSSCOMPILING)
  find_program(_GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)
else()
  set(_GRPC_CPP_PLUGIN_EXECUTABLE $<TARGET_FILE:gRPC::grpc_cpp_plugin>)
endif()
  
# Proto file
get_filename_component(hw_proto "Src/Proto/helloworld.proto" ABSOLUTE)
message("get_filename_component_absolute ${hw_proto}")
get_filename_component(hw_proto_path "${hw_proto}" PATH)
message("get_filename_component_path ${hw_proto_path}")

set(proto_out_dir "${CMAKE_CURRENT_BINARY_DIR}/Protos")

# Generated sources
set(hw_proto_srcs "${proto_out_dir}/helloworld.pb.cc")
set(hw_proto_hdrs "${proto_out_dir}/helloworld.pb.h")
set(hw_grpc_srcs "${proto_out_dir}/helloworld.grpc.pb.cc")
set(hw_grpc_hdrs "${proto_out_dir}/helloworld.grpc.pb.h")

add_custom_command(
      OUTPUT "${hw_proto_srcs}" "${hw_proto_hdrs}" "${hw_grpc_srcs}" "${hw_grpc_hdrs}"
      COMMAND ${_PROTOBUF_PROTOC}
      ARGS --grpc_out "${proto_out_dir}"
        --cpp_out "${proto_out_dir}"
        -I "${hw_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${hw_proto}"
      DEPENDS "${hw_proto}")

# Include generated *.pb.h files
include_directories("${proto_out_dir}")

# hw_grpc_proto
add_library(hw_grpc_proto
  ${hw_grpc_srcs}
  ${hw_grpc_hdrs}
  ${hw_proto_srcs}
  ${hw_proto_hdrs})
target_link_libraries(hw_grpc_proto
  ${_REFLECTION}
  ${_GRPC_GRPCPP}
  ${_PROTOBUF_LIBPROTOBUF})

# Targets greeter_client and greeter_server
set(_target_client greeter_client)
set(_target_server greeter_server)

add_executable(${_target_client} "Src/Client/${_target_client}.cc")
target_link_libraries(${_target_client}
  hw_grpc_proto
  ${_REFLECTION}
  ${_GRPC_GRPCPP}
  ${_PROTOBUF_LIBPROTOBUF}
)

add_executable(${_target_server} "Src/Server/${_target_server}.cc")
target_link_libraries(${_target_server}
  hw_grpc_proto
  ${_REFLECTION}
  ${_GRPC_GRPCPP}
  ${_PROTOBUF_LIBPROTOBUF}
)
