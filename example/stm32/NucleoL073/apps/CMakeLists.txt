##################################################################################
##
## Copyright (C) 2019, New H3C Technologies Co., Ltd. 
##
##      License: Revised BSD License, see LICENSE.TXT file included in the project
## Data Created: 2019/10/9
##      Authors: wangzhen
##  Description: CMake project file
##
##################################################################################
project(application)
cmake_minimum_required(VERSION 3.6)

#---------------------------------------------------------------------------------------
# 案例应用定制
#---------------------------------------------------------------------------------------
set(APPLICATION_LIST classA classB classC newclassA FwUpdate)
set(APPLICATION newclassA CACHE STRING "Default Application is classA")
set_property(CACHE APPLICATION PROPERTY STRINGS ${APPLICATION_LIST})

if(APPLICATION STREQUAL classA)
	list(APPEND ${PROJECT_NAME}_SOURCES
		"${CMAKE_CURRENT_SOURCE_DIR}/LoRaMac/classA/main.c"
	)
elseif(APPLICATION STREQUAL classB)
	list(APPEND ${PROJECT_NAME}_SOURCES
		"${CMAKE_CURRENT_SOURCE_DIR}/LoRaMac/classB/main.c"
	)
elseif(APPLICATION STREQUAL classC)
	list(APPEND ${PROJECT_NAME}_SOURCES
		"${CMAKE_CURRENT_SOURCE_DIR}/LoRaMac/classC/main.c"
	)
elseif(APPLICATION STREQUAL newclassA)
	list(APPEND ${PROJECT_NAME}_SOURCES
		"${CMAKE_CURRENT_SOURCE_DIR}/new-classA/main.c"
	)
elseif(APPLICATION STREQUAL FwUpdate)
	list(APPEND ${PROJECT_NAME}_SOURCES
		"${CMAKE_CURRENT_SOURCE_DIR}/FwUpdate/main.c"
	)
endif()


add_executable( ${APPLICATION}
                ${${PROJECT_NAME}_SOURCES}
                $<TARGET_OBJECTS:loramac>
                $<TARGET_OBJECTS:system>
                $<TARGET_OBJECTS:radio>
                $<TARGET_OBJECTS:mac>
                $<TARGET_OBJECTS:ladapter>
                $<TARGET_OBJECTS:3rd_board>
                $<TARGET_OBJECTS:lorawan-apps>
                $<TARGET_OBJECTS:net>
                $<TARGET_OBJECTS:NucleoL073>
		$<TARGET_OBJECTS:middleware>             
	)

target_include_directories( ${APPLICATION} PUBLIC
		${CMAKE_CURRENT_SOURCE_DIR}/LoRaMac/classA
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/system
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/lorawan/loramac/mac
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/lorawan/loramac/radio
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/apps/common
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/include
		${CMAKE_CURRENT_SOURCE_DIR}/../../../include
		${CMAKE_CURRENT_SOURCE_DIR}/../board
		${CMAKE_CURRENT_SOURCE_DIR}/../../../../lora-sdk/middleware
)
	
#	target_compile_definitions(${APPLICATION} PRIVATE $<$<BOOL:${CLASSB_ENABLED}>:LORAMAC_CLASSB_ENABLED>)
	

target_compile_definitions(${APPLICATION}  PUBLIC
		$<BUILD_INTERFACE:$<TARGET_PROPERTY:mac,INTERFACE_COMPILE_DEFINITIONS>>
)

if(MBED_RADIO_SHIELD STREQUAL WSL305S)
    target_compile_definitions(${APPLICATION} PUBLIC -DLORA_MODULE_WSL30X)
endif()
	
set_property(TARGET ${APPLICATION} PROPERTY C_STANDARD 11)

target_link_libraries(${APPLICATION} m)


# 打印进程的段空间占用以及编译bin和hex交付件
include(binutils-arm-none-eabi)

# Print section sizes of target
print_section_sizes(${APPLICATION})

# Create output in hex and binary format
create_bin_output(${APPLICATION})
create_hex_output(${APPLICATION})

