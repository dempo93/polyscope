# Maybe stop from CMAKEing in the wrong place
if(CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
    message(FATAL_ERROR "Source and build directories cannot be the same. Go use the /build directory.")
endif()

set(INCLUDE_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../include/polyscope/")

find_package(args CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(Stb REQUIRED)
find_path(HAPPLY_INCLUDE_DIRS "happly.h")
find_package(imgui CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

list(APPEND BACKEND_SRCS render/opengl/gl_engine.cpp render/mock_opengl/mock_gl_engine.cpp)

# Configure the render backend
if("${POLYSCOPE_BACKEND_OPENGL3_GLFW}")
    message("Polyscope backend openGL3_glfw enabled")

    list(
        APPEND
        BACKEND_SRCS
        render/opengl/gl_engine.cpp
        render/opengl/shaders/texture_draw_shaders.cpp
        render/opengl/shaders/lighting_shaders.cpp
        render/opengl/shaders/ground_plane_shaders.cpp
        render/opengl/shaders/gizmo_shaders.cpp
        render/opengl/shaders/histogram_shaders.cpp
        render/opengl/shaders/surface_mesh_shaders.cpp
        render/opengl/shaders/volume_mesh_shaders.cpp
        render/opengl/shaders/vector_shaders.cpp
        render/opengl/shaders/sphere_shaders.cpp
        render/opengl/shaders/ribbon_shaders.cpp
        render/opengl/shaders/cylinder_shaders.cpp
        render/opengl/shaders/rules.cpp
        render/opengl/shaders/common.cpp)

    list(APPEND BACKEND_HEADERS ${INCLUDE_ROOT}render/opengl/gl_engine.h ${INCLUDE_ROOT}render/opengl/shaders/common.h)

    # Link settings
    list(APPEND BACKEND_LIBS glfw glad::glad)

    if(APPLE)
        # On macOS, get openGL & friends from Frameworks; do not use GLAD at all

        # NOTE: This code is essentially duplicated here and in deps/imgui/CMakeLists.txt

        # Apple is playing hardball and deprecating openGL... we'll cross that bridge when we come to it
        # Silence warnings about openGL deprecation
        add_definitions(-DGL_SILENCE_DEPRECATION)

        find_library(cocoa_library Cocoa)
        find_library(opengl_library OpenGL)
        find_library(corevideo_library CoreVideo)
        find_library(iokit_library IOKit)
        list(APPEND BACKEND_LIBS ${cocoa_library} ${opengl_library} ${corevideo_library} ${iokit_library})
    else()
        # On Windows/Linux, use the glad openGL loader

        list(APPEND BACKEND_LIBS glad::glad)
    endif()

    list(APPEND BACKEND_DEFINITIONS POLYSCOPE_BACKEND_OPENGL3_GLFW_ENABLED)

endif()

if("${POLYSCOPE_BACKEND_OPENGL_MOCK}")
    message("Polyscope backend openGL_mock enabled")

    list(
        APPEND
        BACKEND_SRCS
        render/mock_opengl/mock_gl_engine.cpp
        render/opengl/shaders/texture_draw_shaders.cpp
        render/opengl/shaders/lighting_shaders.cpp
        render/opengl/shaders/ground_plane_shaders.cpp
        render/opengl/shaders/gizmo_shaders.cpp
        render/opengl/shaders/histogram_shaders.cpp
        render/opengl/shaders/surface_mesh_shaders.cpp
        render/opengl/shaders/volume_mesh_shaders.cpp
        render/opengl/shaders/vector_shaders.cpp
        render/opengl/shaders/sphere_shaders.cpp
        render/opengl/shaders/ribbon_shaders.cpp
        render/opengl/shaders/cylinder_shaders.cpp
        render/opengl/shaders/rules.cpp
        render/opengl/shaders/common.cpp)

    list(APPEND BACKEND_HEADERS ${INCLUDE_ROOT}render/mock_opengl/mock_gl_engine.h
         ${INCLUDE_ROOT}render/opengl/shaders/common.h)

    list(APPEND BACKEND_DEFINITIONS POLYSCOPE_BACKEND_OPENGL_MOCK_ENABLED)
endif()

set(SRCS
    # Core functionality
    polyscope.cpp
    options.cpp
    internal.cpp
    state.cpp
    structure.cpp
    utilities.cpp
    view.cpp
    screenshot.cpp
    messages.cpp
    pick.cpp
    widget.cpp
    # Rendering stuff
    render/engine.cpp
    render/color_maps.cpp
    render/ground_plane.cpp
    render/materials.cpp
    render/initialize_backend.cpp
    render/shader_builder.cpp
    # General utilities
    disjoint_sets.cpp
    file_helpers.cpp
    camera_parameters.cpp
    histogram.cpp
    persistent_value.cpp
    color_management.cpp
    transformation_gizmo.cpp
    slice_plane.cpp
    # Structures
    # Point cloud
    point_cloud.cpp
    point_cloud_color_quantity.cpp
    point_cloud_scalar_quantity.cpp
    point_cloud_vector_quantity.cpp
    point_cloud_parameterization_quantity.cpp
    # Surface
    surface_mesh.cpp
    surface_mesh_io.cpp
    surface_scalar_quantity.cpp
    surface_color_quantity.cpp
    surface_distance_quantity.cpp
    surface_parameterization_quantity.cpp
    surface_vector_quantity.cpp
    surface_count_quantity.cpp
    surface_graph_quantity.cpp
    #surface_subset_quantity.cpp
    #surface_selection_quantity.cpp
    #surface_input_curve_quantity.cpp
    # Curve network
    curve_network.cpp
    curve_network_scalar_quantity.cpp
    curve_network_color_quantity.cpp
    curve_network_vector_quantity.cpp
    # Volume mesh
    volume_mesh.cpp
    volume_mesh_color_quantity.cpp
    volume_mesh_scalar_quantity.cpp
    volume_mesh_vector_quantity.cpp
    # Rendering utilities
    imgui_config.cpp
    vector_artist.cpp
    trace_vector_field.cpp
    ribbon_artist.cpp
    image_scalar_artist.cpp
    stbi_impl.cpp
    # Embedded binary data
    render/bindata/bindata_font_lato_regular.cpp
    render/bindata/bindata_font_cousine_regular.cpp
    render/bindata/concrete_seamless.cpp
    render/bindata/bindata_clay.cpp
    render/bindata/bindata_wax.cpp
    render/bindata/bindata_candy.cpp
    render/bindata/bindata_flat.cpp
    render/bindata/bindata_mud.cpp
    render/bindata/bindata_ceramic.cpp
    render/bindata/bindata_jade.cpp
    render/bindata/bindata_normal.cpp)

# Setting headers is useful for some build systems. For instance, in Visual Studio this is necessary for the header files to be indexed as a part of the project.
set(HEADERS
    ${INCLUDE_ROOT}/affine_remapper.h
    ${INCLUDE_ROOT}/affine_remapper.ipp
    ${INCLUDE_ROOT}/camera_parameters.h
    ${INCLUDE_ROOT}/color_management.h
    ${INCLUDE_ROOT}/colors.h
    ${INCLUDE_ROOT}/combining_hash_functions.h
    ${INCLUDE_ROOT}/curve_network.h
    ${INCLUDE_ROOT}/curve_network.ipp
    ${INCLUDE_ROOT}/curve_network_color_quantity.h
    ${INCLUDE_ROOT}/curve_network_quantity.h
    ${INCLUDE_ROOT}/curve_network_scalar_quantity.h
    ${INCLUDE_ROOT}/curve_network_vector_quantity.h
    ${INCLUDE_ROOT}/disjoint_sets.h
    ${INCLUDE_ROOT}/file_helpers.h
    ${INCLUDE_ROOT}/histogram.h
    ${INCLUDE_ROOT}/image_scalar_artist.h
    ${INCLUDE_ROOT}/imgui_config.h
    ${INCLUDE_ROOT}/messages.h
    ${INCLUDE_ROOT}/options.h
    ${INCLUDE_ROOT}/persistent_value.h
    ${INCLUDE_ROOT}/pick.h
    ${INCLUDE_ROOT}/pick.ipp
    ${INCLUDE_ROOT}/point_cloud.h
    ${INCLUDE_ROOT}/point_cloud.ipp
    ${INCLUDE_ROOT}/point_cloud_color_quantity.h
    ${INCLUDE_ROOT}/point_cloud_quantity.h
    ${INCLUDE_ROOT}/point_cloud_scalar_quantity.h
    ${INCLUDE_ROOT}/point_cloud_parameterization_quantity.h
    ${INCLUDE_ROOT}/point_cloud_vector_quantity.h
    ${INCLUDE_ROOT}/polyscope.h
    ${INCLUDE_ROOT}/quantity.h
    ${INCLUDE_ROOT}/quantity.ipp
    ${INCLUDE_ROOT}/render/color_maps.h
    ${INCLUDE_ROOT}/render/engine.h
    ${INCLUDE_ROOT}/render/engine.ipp
    ${INCLUDE_ROOT}/render/ground_plane.h
    ${INCLUDE_ROOT}/render/material_defs.h
    ${INCLUDE_ROOT}/render/materials.h
    ${INCLUDE_ROOT}/ribbon_artist.h
    ${INCLUDE_ROOT}/scaled_value.h
    ${INCLUDE_ROOT}/screenshot.h
    ${INCLUDE_ROOT}/slice_plane.h
    ${INCLUDE_ROOT}/standardize_data_array.h
    ${INCLUDE_ROOT}/structure.h
    ${INCLUDE_ROOT}/structure.ipp
    ${INCLUDE_ROOT}/surface_color_quantity.h
    ${INCLUDE_ROOT}/surface_count_quantity.h
    ${INCLUDE_ROOT}/surface_distance_quantity.h
    ${INCLUDE_ROOT}/surface_graph_quantity.h
    ${INCLUDE_ROOT}/surface_input_curve_quantity.h
    ${INCLUDE_ROOT}/surface_mesh.h
    ${INCLUDE_ROOT}/surface_mesh.ipp
    ${INCLUDE_ROOT}/surface_mesh_io.h
    ${INCLUDE_ROOT}/surface_mesh_quantity.h
    ${INCLUDE_ROOT}/surface_parameterization_enums.h
    ${INCLUDE_ROOT}/surface_parameterization_quantity.h
    ${INCLUDE_ROOT}/surface_scalar_quantity.h
    ${INCLUDE_ROOT}/surface_selection_quantity.h
    ${INCLUDE_ROOT}/surface_subset_quantity.h
    ${INCLUDE_ROOT}/surface_vector_quantity.h
    ${INCLUDE_ROOT}/trace_vector_field.h
    ${INCLUDE_ROOT}/types.h
    ${INCLUDE_ROOT}/utilities.h
    ${INCLUDE_ROOT}/view.h
    ${INCLUDE_ROOT}/volume_mesh.h
    ${INCLUDE_ROOT}/volume_mesh.ipp
    ${INCLUDE_ROOT}/volume_mesh_quantity.h)

# Create a single library for the project
add_library(polyscope STATIC)
target_sources(polyscope PUBLIC ${HEADERS} ${SRCS} ${BACKEND_HEADERS} ${BACKEND_SRCS})

# Required compiler settings
set_target_properties(
    polyscope
    PROPERTIES POSITION_INDEPENDENT_CODE TRUE
               CXX_STANDARD 11
               CXX_STANDARD_REQUIRED TRUE)

target_link_libraries(polyscope PUBLIC ${BACKEND_LIBS} glm::glm taywee::args imgui::imgui nlohmann_json::nlohmann_json)

target_include_directories(polyscope PUBLIC ${HAPPLY_INCLUDE_DIRS} ${Stb_INCLUDE_DIR} "${CMAKE_SOURCE_DIR}/include/")

target_compile_definitions(polyscope PUBLIC NOMINMAX ${BACKEND_DEFINITIONS})

# Mark the library target and its headers as installable
install(TARGETS polyscope)
install(FILES ${HEADERS} DESTINATION "${CMAKE_INSTALL_PREFIX}/include/polyscope")
