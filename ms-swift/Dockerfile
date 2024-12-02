FROM ghcr.io/protodef/llmwebui-images/base:alpha

# args - software versions
# https://github.com/pytorch/pytorch/releases
# https://github.com/pytorch/audio/releases
# https://github.com/pytorch/vision/releases
ARG PYTORCH_VERSION=2.3.1
ARG TORCHAUDIO_VERSION=2.3.1
ARG TORCHVISION_VERSION=0.18.1

ARG SWIFT_DIR=/opt/ms-swift
ARG SWIFT_VERSION=v2.6.0
ARG DEEPSPEED_VERSION=0.15.4
ARG VLLM_VERSION=0.6.4.post1

# install - pytorch (cuda)
RUN pip install --quiet --index-url https://download.pytorch.org/whl/cu121 \
    torch==${PYTORCH_VERSION} \
    torchaudio==${TORCHAUDIO_VERSION} \
    torchvision==${TORCHVISION_VERSION}

# install - SWIFT
USER root
RUN mkdir -pv ${SWIFT_DIR} \
 && chmod 2775 ${SWIFT_DIR} \
 && chown -R ${LLMWEBUI_USER}:${LLMWEBUI_GID} ${SWIFT_DIR}
USER $LLMWEBUI_USER
RUN git clone https://github.com/modelscope/swift.git --branch ${SWIFT_VERSION} ${SWIFT_DIR}
RUN pip install -e "${SWIFT_DIR}[llm]"
RUN pip install deepspeed==${DEEPSPEED_VERSION} vllm==${VLLM_VERSION} \
 && pip cache purge

# s6 - copy scripts
COPY --chown=${LLMWEBUI_USER}:${LLMWEBUI_GID} --chmod=755 s6/ /etc

# expose Web UI port (default GRADIO port)
EXPOSE 7860
