# claude-code/.devcontainer/Dockerfile

FROM node:20

ARG TZ
ENV TZ="$TZ"

# 1. 安装系统依赖（合并原有和AutoClipper依赖）
RUN apt update && apt install -y \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  python3-pip \
  python3-venv \
  python3-dev \
  ffmpeg \
  libsndfile1 \
  libsndfile1-dev \
  build-essential \
  pkg-config \
  curl \
  wget

# 2. 确保 node 用户有权限
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# 3. 持久化 bash history
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

ENV DEVCONTAINER=true

# 4. 创建工作区
RUN mkdir -p /workspace /home/node/.claude && \
  chown -R node:node /workspace /home/node/.claude

WORKDIR /workspace

# 5. 安装 git-delta
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/0.18.2/git-delta_0.18.2_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_0.18.2_${ARCH}.deb" && \
  rm "git-delta_0.18.2_${ARCH}.deb"

USER node

# 6. Node 全局包
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

ENV SHELL=/bin/zsh

# 7. zsh 配置
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.0/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# 8. 安装 Claude
RUN npm install -g @anthropic-ai/claude-code

# 9. Python 依赖
# 建议将 requirements.txt 放在 .devcontainer 目录下
COPY requirements.txt /workspace/requirements.txt
RUN python3 -m venv /workspace/venv && \
    /workspace/venv/bin/pip install --upgrade pip && \
    /workspace/venv/bin/pip install -r /workspace/requirements.txt

# 10. 安装 ClipsAI
RUN git clone https://github.com/ClipsAI/clipsai.git /workspace/clipsai && \
    /workspace/venv/bin/pip install -e /workspace/clipsai

# 11. 安装 pyannote-whisper
RUN /workspace/venv/bin/pip install git+https://github.com/yinruiqing/pyannote-whisper.git

# 12. FunClip（可选）
RUN git clone https://github.com/modelscope/FunClip.git /workspace/FunClip && \
    /workspace/venv/bin/pip install -r /workspace/FunClip/requirements.txt

# 13. 前端依赖（可选）
# RUN curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - && \
#     sudo apt-get install -y nodejs && \
#     npm install -g yarn

# 14. 防火墙脚本
COPY init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/node-firewall && \
  chmod 0440 /etc/sudoers.d/node-firewall
USER node
