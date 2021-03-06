﻿# Copyright (C) Microsoft Corporation. All rights reserved.
# This file is distributed under the University of Illinois Open Source License. See LICENSE.TXT for details.

if (WIN32)
  find_package(DiaSDK REQUIRED) # Used for constants and declarations.
endif (WIN32)

set(LLVM_LINK_COMPONENTS
  ${LLVM_TARGETS_TO_BUILD}
  analysis
  asmparser
#  asmprinter # no support for LLVM codegen
  bitreader
  bitwriter
#  codegen # no support for LLVM codegen
  core
#  debuginfodwarf # no support for DWARF files (IR debug info is OK)
#  debuginfopdb # no support for PDB files
  dxcsupport
  hlsl
  instcombine
  ipa
  ipo
  irreader
#  libdriver
#  lineeditor
  linker
  lto
#  mirparser # no support for LLVM codegen
  mssupport
#  object # no support for object files (coff, elf)
  option
#  passes
  profiledata
  scalaropts
#  selectiondag # no support for LLVM codegen
  support
  target
  transformutils
  vectorize
  )

if (WIN32)
set(SOURCES
  dxcapi.cpp
  dxcassembler.cpp
  dxcdia.cpp
  dxclibrary.cpp
  dxcompilerobj.cpp
  dxcvalidator.cpp
  DXCompiler.cpp
  DXCompiler.rc
  DXCompiler.def
  dxcfilesystem.cpp
  dxillib.cpp
  dxcontainerbuilder.cpp
  dxcutil.cpp
  dxcdisassembler.cpp
  dxclinker.cpp
)
else ()
set(SOURCES
  dxcapi.cpp
  dxcassembler.cpp
  dxclibrary.cpp
  dxcompilerobj.cpp
  dxcfilesystem.cpp
  dxcontainerbuilder.cpp
  dxcutil.cpp
  dxcdisassembler.cpp
  dxillib.cpp
  dxcvalidator.cpp
)
set (HLSL_IGNORE_SOURCES
  dxcdia.cpp
  DXCompiler.cpp
  DXCompiler.rc
  DXCompiler.def
  dxclinker.cpp
)
endif(WIN32)

set(LIBRARIES
  clangIndex
#  clangARCMigrate
  clangRewrite
  clangCodeGen
  clangRewriteFrontend
  clangFrontend
  clangDriver
#  clangSerialization
  clangSema
  clangEdit
  clangAST
  clangCodeGen
  clangLex
  clangTooling
  clangBasic
  libclang
  )

set(GENERATED_HEADERS
  ClangAttrClasses
  ClangAttrList
  ClangAttrParsedAttrList
  ClangCommentNodes
  ClangDiagnosticCommon
  ClangDiagnosticFrontend
  ClangDeclNodes
  ClangStmtNodes
  )

add_clang_library(dxcompiler SHARED ${SOURCES})
if (WIN32)
  target_link_libraries(dxcompiler PRIVATE ${LIBRARIES} ${DIASDK_LIBRARIES})
  if (ENABLE_SPIRV_CODEGEN)
    target_link_libraries(dxcompiler PRIVATE clangSPIRV)
  endif (ENABLE_SPIRV_CODEGEN)
  add_dependencies(dxcompiler DxcEtw)
  include_directories(AFTER ${LLVM_INCLUDE_DIR}/dxc/Tracing ${DIASDK_INCLUDE_DIRS})
else ()
  # No DIASDK or DxcEtw on non-Windows platforms.
  target_link_libraries(dxcompiler PRIVATE ${LIBRARIES})
  if (ENABLE_SPIRV_CODEGEN)
    target_link_libraries(dxcompiler PRIVATE clangSPIRV)
  endif (ENABLE_SPIRV_CODEGEN)
  include_directories(AFTER ${LLVM_INCLUDE_DIR}/dxc/Tracing)
endif (WIN32)

set_target_properties(dxcompiler
  PROPERTIES
  OUTPUT_NAME "dxcompiler"
  VERSION ${LIBCLANG_LIBRARY_VERSION}
  DEFINE_SYMBOL _CINDEX_LIB_)

hlsl_update_product_ver("dxcompiler")
