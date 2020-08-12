if(NOT CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	set(x86_asm x86)

	set(encoder_uniq_x86_64_sources
		x86/sse2fdct.c
	)

	add_definitions(-Wno-parentheses -Wno-shift-negative-value)
elseif(NOT CMAKE_CL_64)
	set(x86_asm x86_vc)
endif()

if(NOT CMAKE_CL_64)
	set(decoder_x86_sources
		${x86_asm}/mmxidct.c
		${x86_asm}/mmxfrag.c
		${x86_asm}/mmxstate.c
		${x86_asm}/x86state.c
	)

	if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)")
		set(decoder_arch_sources ${decoder_x86_sources})
		add_definitions(-DOC_X86_ASM)
	endif()
endif()

set(decoder_sources
	apiwrapper.c
	bitpack.c
	decapiwrapper.c
	decinfo.c
	decode.c
	dequant.c
	fragment.c
	huffdec.c
	idct.c
	info.c
	internal.c
	quant.c
	state.c
	${decoder_arch_sources}
)

if(NOT CMAKE_CL_64)
	set(encoder_uniq_x86_sources
		${x86_asm}/mmxencfrag.c
		${x86_asm}/mmxfdct.c
		${x86_asm}/x86enc.c
	)

	set(encoder_shared_x86_sources
		${x86_asm}/mmxfrag.c
		${x86_asm}/mmxidct.c
		${x86_asm}/mmxstate.c
		${x86_asm}/x86state.c
	)

	if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64)|(X86_64)|(amd64)|(AMD64)")
		set(encoder_uniq_arch_sources ${encoder_uniq_x86_sources} ${encoder_uniq_x86_64_sources})
		set(encoder_shared_arch_sources ${encoder_shared_x86_sources})
		add_definitions(-DOC_X86_64_ASM)
	elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)")
		set(encoder_uniq_arch_sources ${encoder_uniq_x86_sources})
		set(encoder_shared_arch_sources ${encoder_shared_x86_sources})
		add_definitions(-DOC_X86_ASM)
	endif()
endif()

set(encoder_uniq_sources
	analyze.c
	fdct.c
	encfrag.c
	encapiwrapper.c
	encinfo.c
	encode.c
	enquant.c
	huffenc.c
	mathops.c
	mcenc.c
	rate.c
	tokenize.c
	${encoder_uniq_arch_sources}
)

set(encoder_sources
	apiwrapper.c
	fragment.c
	idct.c
	internal.c
	state.c
	quant.c
	${encoder_shared_arch_sources}
	${encoder_uniq_sources}
)

include_directories(
	../include
	${OGG_INCLUDE_DIRS}
)

add_library(theoradec ${decoder_sources})
add_library(theoraenc ${encoder_sources})
add_library(theora ${decoder_sources} ${encoder_uniq_sources})

target_link_libraries(theoradec ${OGG_LIBRARIES})
target_link_libraries(theoraenc ${OGG_LIBRARIES})
target_link_libraries(theora ${OGG_LIBRARIES})

install(TARGETS theoradec theoraenc theora
	EXPORT "${targets_export_name}"
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}
	INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
