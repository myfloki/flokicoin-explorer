FROM node:24 AS builder

ARG commitHash
ENV MEMPOOL_COMMIT_HASH=${commitHash}

WORKDIR /build
COPY . .

RUN apt-get update
RUN apt-get install -y build-essential python3 pkg-config curl ca-certificates

# Install Rust via rustup
RUN CPU_ARCH=$(uname -m); if [ "$CPU_ARCH" = "armv7l" ]; then c_rehash; fi
#RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain stable
#Workaround to run on github actions from https://github.com/rust-lang/rustup/issues/2700#issuecomment-1367488985
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sed 's#/proc/self/exe#\/bin\/sh#g' | sh -s -- -y --default-toolchain stable
ENV PATH="/root/.cargo/bin:$PATH"

COPY --from=backend . .
# Copy the Rust GBT project into /rust/gbt so backend preinstall can cd ../rust/gbt
COPY --from=rustgbt . ../rust/gbt/
ENV FD=/build/rust-gbt
RUN npm install --omit=dev --omit=optional

WORKDIR /build
RUN npm run package

FROM node:24

WORKDIR /backend

RUN chown 1000:1000 ./
RUN apt-get update && apt-get install -y --no-install-recommends bash && rm -rf /var/lib/apt/lists/*
COPY --from=builder --chown=1000:1000 /build/package ./package/
# Ensure GeoIP directory exists; optional data can be mounted or added later
RUN mkdir -p /backend/GeoIP
# Copy runtime scripts and config from docker context
COPY --from=docker --chown=1000:1000 backend/mempool-config.json ./mempool-config.json
COPY --from=docker --chown=1000:1000 backend/start.sh ./start.sh
COPY --from=docker --chown=1000:1000 backend/wait-for-it.sh ./wait-for-it.sh

USER 1000

EXPOSE 8999

CMD ["/backend/start.sh"]
