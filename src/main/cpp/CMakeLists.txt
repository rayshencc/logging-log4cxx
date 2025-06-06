#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Build the log4cxx library
add_library(log4cxx)
if(${log4cxx_ABI_VER} GREATER 15)
  set_target_properties(log4cxx PROPERTIES CXX_VISIBILITY_PRESET hidden)
endif()
if(BUILD_SHARED_LIBS)
    target_compile_definitions(log4cxx PRIVATE LOG4CXX)
    if(UNIX AND NOT APPLE)
        # Make defined symbols non-preemptible, which can optimize relocation processing
        target_link_options(log4cxx PRIVATE "LINKER:-Bsymbolic")
    endif()
else()
    target_compile_definitions(log4cxx PUBLIC LOG4CXX_STATIC)
endif()
add_dependencies(log4cxx log4cxx-include)

set(extra_classes "")

if(LOG4CXX_NETWORKING_SUPPORT)
    list(APPEND extra_classes
        datagrampacket.cpp
        datagramsocket.cpp
        aprdatagramsocket.cpp
        serversocket.cpp
        aprserversocket.cpp
        socket.cpp
        aprsocket.cpp
        telnetappender.cpp
        socketappenderskeleton.cpp
        socketoutputstream.cpp
        xmlsocketappender.cpp
        syslogwriter.cpp
        syslogappender.cpp
    )
endif()

if(LOG4CXX_DOMCONFIGURATOR_SUPPORT)
    list(APPEND extra_classes
	domconfigurator.cpp
	)
endif()

if(LOG4CXX_MULTIPROCESS_ROLLING_FILE_APPENDER)
    target_compile_definitions(log4cxx PRIVATE LOG4CXX_MULTI_PROCESS)
    list(APPEND extra_classes
        multiprocessrollingfileappender.cpp
    )
endif()

if(${ENABLE_FMT_LAYOUT})
    list(APPEND extra_classes
        fmtlayout.cpp
    )
endif()

target_sources(log4cxx
  PRIVATE
  action.cpp
  andfilter.cpp
  appenderattachableimpl.cpp
  appenderskeleton.cpp
  aprinitializer.cpp
  asyncappender.cpp
  basicconfigurator.cpp
  bufferedwriter.cpp
  bytearrayinputstream.cpp
  bytearrayoutputstream.cpp
  bytebuffer.cpp
  cacheddateformat.cpp
  charsetdecoder.cpp
  charsetencoder.cpp
  class.cpp
  classnamepatternconverter.cpp
  classregistration.cpp
  colorstartpatternconverter.cpp
  colorendpatternconverter.cpp
  configurator.cpp
  consoleappender.cpp
  cyclicbuffer.cpp
  date.cpp
  dateformat.cpp
  datepatternconverter.cpp
  dbappender.cpp
  defaultconfigurator.cpp
  defaultloggerfactory.cpp
  defaultrepositoryselector.cpp
  exception.cpp
  fallbackerrorhandler.cpp
  file.cpp
  fileappender.cpp
  filedatepatternconverter.cpp
  fileinputstream.cpp
  filelocationpatternconverter.cpp
  fileoutputstream.cpp
  filerenameaction.cpp
  filewatchdog.cpp
  filter.cpp
  filterbasedtriggeringpolicy.cpp
  fixedwindowrollingpolicy.cpp
  formattinginfo.cpp
  fulllocationpatternconverter.cpp
  gzcompressaction.cpp
  hexdump.cpp
  hierarchy.cpp
  htmllayout.cpp
  inetaddress.cpp
  inputstream.cpp
  inputstreamreader.cpp
  integer.cpp
  integerpatternconverter.cpp
  jsonlayout.cpp
  layout.cpp
  level.cpp
  levelmatchfilter.cpp
  levelpatternconverter.cpp
  levelrangefilter.cpp
  linelocationpatternconverter.cpp
  lineseparatorpatternconverter.cpp
  literalpatternconverter.cpp
  loader.cpp
  locale.cpp
  locationinfo.cpp
  locationinfofilter.cpp
  logger.cpp
  loggermatchfilter.cpp
  loggerpatternconverter.cpp
  loggingevent.cpp
  loggingeventpatternconverter.cpp
  loglog.cpp
  logmanager.cpp
  logstream.cpp
  manualtriggeringpolicy.cpp
  mapfilter.cpp
  mdc.cpp
  messagebuffer.cpp
  messagepatternconverter.cpp
  methodlocationpatternconverter.cpp
  nameabbreviator.cpp
  namepatternconverter.cpp
  ndc.cpp
  mdcpatternconverter.cpp
  ndcpatternconverter.cpp
  nteventlogappender.cpp
  odbcappender.cpp
  onlyonceerrorhandler.cpp
  optionconverter.cpp
  outputdebugstringappender.cpp
  outputstream.cpp
  outputstreamwriter.cpp
  patternconverter.cpp
  patternlayout.cpp
  patternparser.cpp
  pool.cpp
  properties.cpp
  propertiespatternconverter.cpp
  propertyconfigurator.cpp
  propertyresourcebundle.cpp
  propertysetter.cpp
  reader.cpp
  relativetimedateformat.cpp
  relativetimepatternconverter.cpp
  resourcebundle.cpp
  rollingfileappender.cpp
  rollingpolicy.cpp
  rollingpolicybase.cpp
  rolloverdescription.cpp
  rootlogger.cpp
  shortfilelocationpatternconverter.cpp
  simpledateformat.cpp
  simplelayout.cpp
  sizebasedtriggeringpolicy.cpp
  smtpappender.cpp
  strftimedateformat.cpp
  stringhelper.cpp
  stringmatchfilter.cpp
  stringtokenizer.cpp
  system.cpp
  systemerrwriter.cpp
  systemoutwriter.cpp
  threadlocal.cpp
  threadpatternconverter.cpp
  threadusernamepatternconverter.cpp
  threadspecificdata.cpp
  threadutility.cpp
  throwableinformationpatternconverter.cpp
  timebasedrollingpolicy.cpp
  timezone.cpp
  transcoder.cpp
  transform.cpp
  triggeringpolicy.cpp
  writer.cpp
  writerappender.cpp
  xmllayout.cpp
  zipcompressaction.cpp
  ${extra_classes}
)

if(LOG4CXX_EVENTS_AT_EXIT)
  target_sources(log4cxx
    PRIVATE atexitregistry.cpp
  )
endif(LOG4CXX_EVENTS_AT_EXIT)

if(WIN32)
  target_sources(log4cxx
    PRIVATE visualstudio.natvis
  )
endif(WIN32)

set(LOG4CXX_LIB_NAME "log4cxx" CACHE STRING "Main library base name")
set_target_properties(log4cxx PROPERTIES
  VERSION ${LIBLOG4CXX_LIB_VERSION}
  SOVERSION ${LIBLOG4CXX_LIB_SOVERSION}
  OUTPUT_NAME ${LOG4CXX_LIB_NAME}
  PDB_NAME ${LOG4CXX_LIB_NAME}
  COMPILE_PDB_NAME ${LOG4CXX_LIB_NAME}
  EXPORT_NAME ${LOG4CXX_LIB_NAME}
)
get_directory_property( FILESYSTEM_IMPL DIRECTORY "${LOG4CXX_SOURCE_DIR}/src/main/include" DEFINITION FILESYSTEM_IMPL )
if("${FILESYSTEM_IMPL}" STREQUAL "std::filesystem" OR
   "${FILESYSTEM_IMPL}" STREQUAL "std::experimental::filesystem" )
  target_link_libraries(log4cxx PUBLIC $<$<AND:$<CXX_COMPILER_ID:GNU>,$<VERSION_LESS:$<CXX_COMPILER_VERSION>,9.0>>:stdc++fs>)
endif()

if(${ENABLE_FMT_LAYOUT})
    target_link_libraries(log4cxx PUBLIC fmt::fmt)
endif()
