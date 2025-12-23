FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /root

# 基本ツールのインストール
RUN apt-get update && apt-get install -y \
    vim \
    less \
    tree \
    bash-completion \
    wget \
    curl \
    iputils-ping \
    git \
    unzip \
    openssh-server \
    lsb-release \
    software-properties-common \
    gnupg \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# LLVM 15のリポジトリを手動で追加
RUN wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc && \
    echo "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-15 main" >> /etc/apt/sources.list.d/llvm.list && \
    echo "deb-src http://apt.llvm.org/jammy/ llvm-toolchain-jammy-15 main" >> /etc/apt/sources.list.d/llvm.list

# CMakeのインストール
RUN apt-get update && apt-get install -y \
    && wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null \
    && echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ jammy main' | tee /etc/apt/sources.list.d/kitware.list >/dev/null \
    && apt-get update \
    && rm /usr/share/keyrings/kitware-archive-keyring.gpg \
    && apt-get install -y kitware-archive-keyring \
    && apt-get install -y cmake \
    && rm -rf /var/lib/apt/lists/*

# 依存パッケージと解析ツールのインストール
RUN apt-get update && apt-get install -y \
    build-essential file g++-multilib gcc-multilib \
    libcap-dev libgoogle-perftools-dev libncurses5-dev libsqlite3-dev libtcmalloc-minimal4 \
    python3-pip graphviz doxygen zlib1g-dev libzstd-dev \
    libz3-dev \
    libexpat1-dev \
    clang-15 llvm-15 llvm-15-dev llvm-15-tools libclang-15-dev \
    clang-tools-15 \
    cppcheck \
    flex bison libglib2.0-dev libpixman-1-dev \
    && rm -rf /var/lib/apt/lists/*

# ツールのシンボリックリンク作成
RUN ln -sf /usr/bin/clang-15 /usr/bin/clang && \
    ln -sf /usr/bin/clang++-15 /usr/bin/clang++ && \
    ln -sf /usr/bin/llvm-config-15 /usr/bin/llvm-config && \
    ln -sf /usr/bin/scan-build-15 /usr/bin/scan-build && \
    ln -sf /usr/bin/scan-view-15 /usr/bin/scan-view

# FlawFinderのインストール
RUN pip3 install flawfinder

# RATSのインストール
RUN wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/rough-auditing-tool-for-security/rats-2.4.tgz && \
    tar -xvf rats-2.4.tgz && \
    cd rats-2.4 && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf rats-2.4 rats-2.4.tgz

# AFL++のインストール(source-only)
RUN git clone https://github.com/AFLplusplus/AFLplusplus.git && \
    cd AFLplusplus && \
    export LLVM_CONFIG=llvm-config-15 && \
    make source-only && \
    make install && \
    cd .. && \
    rm -rf AFLplusplus

# KLEEのインストール
# --- uClibc のビルド ---
RUN git clone https://github.com/klee/klee-uclibc.git && \
    cd klee-uclibc && \
    ./configure --make-llvm-lib --with-llvm-config=/usr/bin/llvm-config-15 && \
    make -j$(nproc)

# --- KLEE 本体のビルド ---
RUN git clone https://github.com/klee/klee.git && \
    cd klee && \
    mkdir build && cd build && \
    cmake \
        -DENABLE_SOLVER_Z3=ON \
        -DENABLE_POSIX_RUNTIME=ON \
        -DENABLE_KLEE_UCLIBC=ON \
        -DKLEE_UCLIBC_PATH=/root/klee-uclibc \
        -DLLVM_CONFIG_BINARY=/usr/bin/llvm-config-15 \
        -DENABLE_UNIT_TESTS=OFF \
        -DENABLE_SYSTEM_TESTS=OFF \
        .. && \
    make -j$(nproc) && \
    make install

# パスの更新
ENV PATH="/usr/local/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/lib:/usr/lib/llvm-15/lib"
ENV AFL_PATH="/usr/local/lib/afl"

CMD ["/bin/bash"]