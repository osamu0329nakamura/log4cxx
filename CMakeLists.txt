cmake_minimum_required(VERSION 3.0)

project(log4cxx VERSION 0.10.0 LANGUAGES C CXX)
include(CheckFunctionExists)
include(CheckIncludeFile)

set(enable-unichar  0       CACHE BOOL "enable unichar(no)")
set(enable-cfstring 0       CACHE BOOL "enable cfstring(no)")
set(enable-wchar_t  1       CACHE BOOL "enable wchar_t")

set(with-logchar    utf-8   CACHE STRING "type for logchar. Accepted TYPE variants: auto, utf-8,\n"
                                  "wchar_t, unichar (default=utf-8)")
set(with-charset    auto    CACHE STRING "locale charset Accepted TYPE variants: auto, utf-8,\n"
                                  "wchar_t, unichar (default=auto)")
set(with-ODBC       NO      CACHE STRING "ODBC support. Accepted arguments: unixODBC, iODBC,\n"
                                         "Microsoft, NO (default=NO)")
set(with-SMTP       NO      CACHE STRING "SMTP support, Accepted arguments: libesmtp, NO\n"
                                         "(default=NO)")

# char api
set(CHAR_API            1)
set(UNICHAR_API         0)
set(WCHAR_T_API         1)
set(CFSTRING_API        0)

# logchar
set(LOGCHAR_IS_UTF8     1)
set(LOGCHAR_IS_UNICHAR  0)
set(LOGCHAR_IS_WCHAR    0)

# charset
set(CHARSET_UTF8        0)
set(CHARSET_ISO88591    0)
set(CHARSET_USASCII     0)
set(CHARSET_EBCDIC      0)

# check function exists
CHECK_FUNCTION_EXISTS(mbsrtowcs HAS_MBSRTOWCS)
CHECK_FUNCTION_EXISTS(wcstombs  HAS_WCSTOMBS)
CHECK_FUNCTION_EXISTS(fwide     HAS_FWIDE)

# check header exists
CHECK_INCLUDE_FILE(syslog.h HAS_SYSLOG)
CHECK_INCLUDE_FILE(locale.h HAS_STD_LOCALE)
# check libesmtp exists

set(HAS_LIBESMTP 0)
set(HAS_ODBC 0)
if(with-SMTP)
    find_library(LIBESMTP libesmtp REQUIRED)
    set(HAS_LIBESMTP 0)
endif()

# configure log4cxx.h
configure_file(${PROJECT_SOURCE_DIR}/src/main/include/log4cxx/log4cxx.h.in
    ${PROJECT_BINARY_DIR}/log4cxx/log4cxx.h
    @ONLY)

# configure log4cxx
configure_file(${PROJECT_SOURCE_DIR}/src/main/include/log4cxx/private/log4cxx_private.h.in
    ${PROJECT_BINARY_DIR}/log4cxx/private/log4cxx_private.h
    @ONLY)



file(GLOB log4cxx_source "${PROJECT_SOURCE_DIR}/src/main/cpp/*.cpp")
file(GLOB_RECURSE log4cxx_header "${PROJECT_SOURCE_DIR}/src/main/include/log4cxx/*.h")
add_library(log4cxx SHARED 
    ${log4cxx_source}
)

target_include_directories(log4cxx
    PUBLIC
        /usr/include/apr-1
        $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src/main/include>
        $<INSTALL_INTERFACE:include>
)

set_target_properties(log4cxx
    PROPERTIES
        VERSION "${PROJECT_VERSION}"
        SOVERSION "${PROJECT_VERSION}"
    )

install(TARGETS log4cxx
    EXPORT log4cxx-export
    LIBRARY DESTINATION lib
)

install(EXPORT log4cxx-export
    DESTINATION cmake)
export(EXPORT log4cxx-export)

install(DIRECTORY
    ${PROJECT_BINARY_DIR}/log4cxx
    DESTINATION include
    COMPONENT devel
)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/src/main/include/log4cxx
    DESTINATION include
    COMPONENT devel
    PATTERN "*.h"
    PATTERN "*.hw" EXCLUDE
    PATTERN "*.in" EXCLUDE
    PATTERN ".*" EXCLUDE
    PATTERN "Makefile.am" EXCLUDE
)
