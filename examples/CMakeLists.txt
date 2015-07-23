add_executable(decoder_example "decoder_example.c")
target_link_libraries(decoder_example libvorbis_static)

add_executable(encoder_example "encoder_example.c")
target_link_libraries(encoder_example libvorbis_static)

add_executable(chaining_example "chaining_example.c")
target_link_libraries(chaining_example libvorbis_static)

add_executable(seeking_example "seeking_example.c")
target_link_libraries(seeking_example libvorbis_static)

add_executable(vorbisfile_example "vorbisfile_example.c")
target_link_libraries(vorbisfile_example libvorbisfile_static)
