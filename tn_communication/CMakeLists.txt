if(GEN_PROTO)
        get_filename_component(tn_communication_proto "../protos/tn_communication.proto" ABSOLUTE)
        get_filename_component(tn_communication_proto_path "${tn_communication_proto}" PATH)

        set(tn_communication_proto_sources "${CMAKE_CURRENT_SOURCE_DIR}/tn_communication.pb.cc")
        set(tn_communication_proto_headers "${CMAKE_CURRENT_SOURCE_DIR}/tn_communication.pb.h")
        set(tn_communication_grpc_sources "${CMAKE_CURRENT_SOURCE_DIR}/tn_communication.grpc.pb.cc")
        set(tn_communication_grpc_headers "${CMAKE_CURRENT_SOURCE_DIR}/tn_communication.grpc.pb.h")
        add_custom_command(
                OUTPUT "${tn_communication_proto_sources}" "${tn_communication_proto_headers}" "${tn_communication_grpc_sources}" "${tn_communication_grpc_headers}"
                COMMAND ${_PROTOBUF_PROTOC}
                ARGS --grpc_out "${CMAKE_CURRENT_SOURCE_DIR}" --cpp_out "${CMAKE_CURRENT_SOURCE_DIR}"
                -I "${tn_communication_proto_path}"
                --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
                "${tn_communication_proto}"
                DEPENDS "${tn_communication_proto}")

        add_custom_target(finish_gen DEPENDS "${tn_communication_proto_sources}" "${tn_communication_proto_headers}" "${tn_communication_grpc_sources}" "${tn_communication_grpc_headers}")
        message("gen ...")
        message("current " "${CMAKE_CURRENT_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}")

        

else()
        include_directories("${CMAKE_CURRENT_SOURCE_DIR}")

        add_library(tn_communication_grpc_proto
                ${tn_communication_proto_sources}
                ${tn_communication_proto_headers}
                ${tn_communication_grpc_sources}
                ${tn_communication_grpc_headers})
        target_link_libraries(tn_communication_grpc_proto
                ${_PROTOBUF_LIBPROTOBUF}
                ${_REFLECTION}
                ${_GRPC_GRPCPP})

        foreach(_target tn_communication_client tn_communication_server tn_communication_async_client tn_communication_async_server)
        add_executable(${_target} "${_target}.cc")
        target_link_libraries(${_target}
                tn_communication_grpc_proto
                ${_PROTOBUF_LIBPROTOBUF}
                ${_REFLECTION}
                ${_GRPC_GRPCPP})
        endforeach()
endif()