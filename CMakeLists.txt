# Assuming environment variables and paths from buildenv.mk are set

# Project Name
project(SGX_MM_Library)

# Include directories
include_directories(include)
include_directories(tlibc)

# Compiler flags
add_definitions(-Wno-missing-braces)
add_definitions(-Wno-unused-parameter)
set(EMMFLAGS "-mno-sse")

set(COMMON_FLAGS "-fstack-protector-strong -ffunction-sections -fdata-sections -O2 -D_FORTIFY_SOURCE=2 -UDEBUG -DNDEBUG")

# turn on compiler warnings as much as possible
set(COMMON_FLAGS "${COMMON_FLAGS} -Wall -Wextra -Winit-self -Wpointer-arith -Wreturn-type \
-Waddress -Wsequence-point -Wformat-security \
-Wmissing-include-dirs -Wfloat-equal -Wundef -Wshadow \
-Wcast-align -Wconversion -Wredundant-decls")


# Enable the security flags
set(COMMON_LDFLAGS "-Wl,-z,relro,-z,now,-z,noexecstack")

# Get the system architecture using uname
execute_process(
    COMMAND uname -m
    OUTPUT_VARIABLE UNAME
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Determine HOST_ARCH based on the output of uname
if(UNAME MATCHES "86")
    # This only builds on x86 machines
else()
    message(FATAL_ERROR "Unknown host CPU architecture: ${UNAME}")
endif()

# Add conditional flags based on the ARCH variable
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(COMMON_FLAGS "${COMMON_FLAGS} -DITT_ARCH_IA64")
elseif(ARCH STREQUAL "x86_64")
    set(COMMON_FLAGS "${COMMON_FLAGS}  -DITT_ARCH_IA32")
endif()


# Compiler flags for C and C++
set(CMAKE_C_FLAGS " ${CMAKE_C_FLAGS} ${COMMON_FLAGS} -ffreestanding -nostdinc -fvisibility=hidden -fpie -fno-strict-overflow -fno-delete-null-pointer-checks")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_C_FLAGS} -nostdinc++")

# Linker flags
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -B${BINUTILS_DIR} ${COMMON_LDFLAGS} -Wl,-Bstatic -Wl,-Bsymbolic -Wl,--no-undefined -Wl,-pie,-eenclave_entry -Wl,--export-dynamic -Wl,--defsym,__ImageBase=0")

# Source files
set(C_FILES 
    bit_array.c
    ema.c
    emalloc.c
    emm_private.c
    sgx_mm.c
)

# Define the static library
add_library(sgx_mm STATIC ${C_FILES})
target_compile_options(sgx_mm PRIVATE ${EMMFLAGS})