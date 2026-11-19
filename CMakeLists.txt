project(lsof)

# Apple's lsof for Darwin (libproc dialect). The upstream Configure script
# generates version.h and links the dialect sources into the top level; we do
# the equivalent here.
set(LSOF_DIALECT ${CMAKE_CURRENT_SOURCE_DIR}/lsof/dialects/darwin/libproc)
set(LSOF_VERSION "4.91")

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/version.h
"#define LSOF_VERSION \"${LSOF_VERSION}\"
#define LSOF_URL \"ftp://lsof.itap.purdue.edu/pub/tools/unix/lsof/\"
#define LSOF_CINFO \"libproc-based\"
#define LSOF_CCDATE \"\"
#define LSOF_HOST \"\"
#define LSOF_LOGNAME \"\"
#define LSOF_USER \"\"
#define LSOF_CC \"cc\"
#define LSOF_CCV \"\"
#define LSOF_CCFLAGS \"\"
#define LSOF_LDFLAGS \"\"
#define LSOF_SYSINFO \"\"
")

add_definitions(
	-DDARWINV=2000
	-DHASIPv6
	-DHASUTMPX
	-w
)

# The files in lsof/lib include the dialect headers as ../machine.h etc.;
# Configure makes those visible in the source tree, so give them a copy of the
# same layout in the build tree instead.
foreach(hdr dlsof.h dproto.h machine.h)
	configure_file(${LSOF_DIALECT}/${hdr} ${CMAKE_CURRENT_BINARY_DIR}/${hdr} COPYONLY)
endforeach()
foreach(hdr lsof.h lsof_fields.h proto.h regex.h)
	configure_file(${CMAKE_CURRENT_SOURCE_DIR}/lsof/${hdr} ${CMAKE_CURRENT_BINARY_DIR}/${hdr} COPYONLY)
endforeach()
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

include_directories(
	${CMAKE_CURRENT_BINARY_DIR}
	${CMAKE_CURRENT_BINARY_DIR}/lib
	${LSOF_DIALECT}
	${CMAKE_CURRENT_SOURCE_DIR}/lsof
)

add_darling_executable(lsof
	lsof/arg.c
	lsof/main.c
	lsof/misc.c
	lsof/node.c
	lsof/print.c
	lsof/proc.c
	lsof/store.c
	lsof/usage.c
	lsof/util.c
	${LSOF_DIALECT}/ddev.c
	${LSOF_DIALECT}/dfile.c
	${LSOF_DIALECT}/dmnt.c
	${LSOF_DIALECT}/dnetpolicy.c
	${LSOF_DIALECT}/dproc.c
	${LSOF_DIALECT}/dsock.c
	${LSOF_DIALECT}/dstore.c
	lsof/lib/ckkv.c
	lsof/lib/cvfs.c
	lsof/lib/dvch.c
	lsof/lib/fino.c
	lsof/lib/isfn.c
	lsof/lib/lkud.c
	lsof/lib/pdvn.c
	lsof/lib/prfp.c
	lsof/lib/ptti.c
	lsof/lib/rdev.c
	lsof/lib/regex.c
	lsof/lib/rmnt.c
	lsof/lib/rnam.c
	lsof/lib/rnch.c
	lsof/lib/rnmh.c
	lsof/lib/snpf.c
)

install(TARGETS lsof DESTINATION libexec/darling/usr/sbin)
install(FILES lsof/lsof.8 DESTINATION libexec/darling/usr/share/man/man8)
