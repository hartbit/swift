set(swiftDriver_sources
  Action.cpp
  Compilation.cpp
  DarwinToolChains.cpp
  DependencyGraph.cpp
  SourceComparator.cpp
  Driver.cpp
  DriverIncrementalRanges.cpp
  FineGrainedDependencyDriverGraph.cpp
  FrontendUtil.cpp
  Job.cpp
  ParseableOutput.cpp
  PrettyStackTrace.cpp
  ToolChain.cpp
  ToolChains.cpp
  UnixToolChains.cpp
  WindowsToolChains.cpp
  )

set(swiftDriver_targetDefines)

add_swift_host_library(swiftDriver STATIC
  ${swiftDriver_sources})
add_dependencies(swiftDriver
  SwiftOptions)
target_link_libraries(swiftDriver PRIVATE
  swiftAST
  swiftBasic
  swiftOption)

# Generate the static-stdlib-args.lnk file used by -static-stdlib option
# for 'GenericUnix' (eg linux)
if(SWIFT_BUILD_STATIC_STDLIB)
  set(static_stdlib_lnk_file_list)
  foreach(sdk ${SWIFT_CONFIGURED_SDKS})
    if("${SWIFT_SDK_${sdk}_OBJECT_FORMAT}" STREQUAL "ELF")
      string(TOLOWER "${sdk}" lowercase_sdk)
      if(SWIFT_${SWIFT_HOST_VARIANT_SDK}_${SWIFT_HOST_VARIANT_ARCH}_ICU_STATICLIB)
        set(ICU_STATICLIB "TRUE")
      else()
        set(ICU_STATICLIB "FALSE")
        find_package(ICU REQUIRED COMPONENTS uc i18n)
        get_filename_component(ICU_UC_LIBDIR "${ICU_UC_LIBRARIES}" DIRECTORY)
        get_filename_component(ICU_I18N_LIBDIR "${ICU_I18N_LIBRARIES}" DIRECTORY)
      endif()
      set(linkfile "${lowercase_sdk}/static-stdlib-args.lnk")
      add_custom_command_target(swift_static_stdlib_${sdk}_args
        COMMAND
          "${SWIFT_SOURCE_DIR}/utils/gen-static-stdlib-link-args"
          "${sdk}"
          "${SWIFTSTATICLIB_DIR}/${linkfile}"
          "${ICU_STATICLIB}"
          "${ICU_UC_LIBDIR}"
          "${ICU_I18N_LIBDIR}"
        OUTPUT
          "${SWIFTSTATICLIB_DIR}/${linkfile}"
        DEPENDS
          "${SWIFT_SOURCE_DIR}/utils/gen-static-stdlib-link-args")

      list(APPEND static_stdlib_lnk_file_list ${swift_static_stdlib_${sdk}_args})
      swift_install_in_component(FILES "${SWIFTSTATICLIB_DIR}/${linkfile}"
                                 DESTINATION "lib/swift_static/${lowercase_sdk}"
                                 COMPONENT stdlib)
    endif()
  endforeach()
  add_custom_target(swift_static_lnk_args ALL DEPENDS ${static_stdlib_lnk_file_list})
  add_dependencies(stdlib swift_static_lnk_args)
endif()
