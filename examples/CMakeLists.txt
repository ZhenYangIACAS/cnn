CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

foreach(TARGET  rnnlm2_cls_based attentional poisson-regression tag-bilstm embed-cl encdec xor xor-xent rnnlm-aevb rnnlm nlm textcat rnnlm2 mp)
  ADD_EXECUTABLE(${TARGET} ${TARGET}.cc)
  target_link_libraries(${TARGET} cnn ${LIBS})
  if (WIN32 OR WIN64)
  if (WITH_CUDA_BACKEND)
    add_dependencies(${TARGET} cnncuda ${CUDNN_LIB} cudart cublas curand)
    target_link_libraries(${TARGET} cnncuda ${CUDNN_LIB} cudart cublas curand)
    CUDA_ADD_CUBLAS_TO_TARGET(${TARGET})
  endif (WITH_CUDA_BACKEND)
  else()
  if (WITH_CUDA_BACKEND)
    add_dependencies(${TARGET} cnncuda ${CUDNN_LIB} cudart cublas culibos dl)
    target_link_libraries(${TARGET} cnncuda ${CUDNN_LIB} cudart cublas culibos dl)
    CUDA_ADD_CUBLAS_TO_TARGET(${TARGET})
  endif (WITH_CUDA_BACKEND)
  endif()
endforeach()

