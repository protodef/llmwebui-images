FROM ghcr.io/protodef/llmwebui-images/base:alpha

ARG LLAMAFACTORY_DIR=/opt/llama-factory
ARG LLAMAFACTORY_VERSION=v0.9.1

# install - LLaMA Factory
USER root
RUN mkdir -pv ${LLAMAFACTORY_DIR} \
 && chmod 2775 ${LLAMAFACTORY_DIR} \
 && chown -R ${LLMWEBUI_USER}:${LLMWEBUI_GID} ${LLAMAFACTORY_DIR}
USER $LLMWEBUI_USER
RUN git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git --branch ${LLAMAFACTORY_VERSION} ${LLAMAFACTORY_DIR}
RUN pip install -e "${LLAMAFACTORY_DIR}[torch,metrics,deepspeed,vllm]" \
 && pip cache purge

# s6 - copy scripts
COPY --chown=${LLMWEBUI_USER}:${LLMWEBUI_GID} --chmod=755 s6/ /etc

# expose Web UI port (default GRADIO port)
EXPOSE 7860
