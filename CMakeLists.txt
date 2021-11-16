cmake_minimum_required(VERSION 3.19)

project(programas)
file(GLOB_RECURSE PROGRAMAS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.md)

list(REMOVE_ITEM PROGRAMAS "Programs.md")
list(REMOVE_ITEM PROGRAMAS "README.md")
list(REMOVE_ITEM PROGRAMAS "Intro.in.md")

foreach(P IN LISTS PROGRAMAS)
    message(STATUS "Procesando programa: \"${P}\"")
endforeach()

list(TRANSFORM PROGRAMAS PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/)


string(TIMESTAMP GENERATION_DATE "at %H:%M, on %A, %B %d, %Y (UTC)" UTC)

set(INTRO_IN "${CMAKE_CURRENT_SOURCE_DIR}/Intro.in.md")
set(INTRO "${CMAKE_CURRENT_BINARY_DIR}/Intro.md")

configure_file(${INTRO_IN} ${INTRO})

set(PANDOC_INPUT "--include-before-body=${INTRO}")
foreach(P IN LISTS PROGRAMAS)
    list(APPEND PANDOC_INPUT "${P}")
endforeach()

string(TIMESTAMP TODAY UTC)

set(MD_MERGE_TARGET "${CMAKE_CURRENT_SOURCE_DIR}/Programs.md")
message(STATUS "Generating Programs Markdown with Pandoc: ${MD_MERGE_TARGET}")
execute_process(
    COMMAND pandoc "--standalone" "--to=gfm"  "--toc" "--metadata=date:\"${TODAY}\"" "--metadata=title:\"Universidad Simon Bolivar Course Descriptions\""  "--output=${MD_MERGE_TARGET}" ${PANDOC_INPUT}
    COMMAND_ERROR_IS_FATAL ANY
    COMMAND_ECHO STDOUT 
)

set(DOCX_TARGET "${CMAKE_CURRENT_SOURCE_DIR}/Programs.docx")
message(STATUS "Generating Programs DOCX with Pandoc: ${DOCX_TARGET}")
execute_process(
    COMMAND pandoc "--output=${DOCX_TARGET}" "--from=gfm" "--to=docx" ${MD_MERGE_TARGET}
    COMMAND_ERROR_IS_FATAL ANY
)