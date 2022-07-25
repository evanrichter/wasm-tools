# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv target/x86_64-unknown-linux-gnu/release/text-parser /text-parser && \
    mv target/x86_64-unknown-linux-gnu/release/validate /validate && \
    mv target/x86_64-unknown-linux-gnu/release/print /print && \
    mv target/x86_64-unknown-linux-gnu/release/roundtrip /roundtrip && \
    mv target/x86_64-unknown-linux-gnu/release/incremental-parse /incremental-parse && \
    mv target/x86_64-unknown-linux-gnu/release/validate-valid-module /validate-valid-module && \
    mv target/x86_64-unknown-linux-gnu/release/mutate /mutate && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv target/release/text-parser /text-parser_no_inst && \
    mv target/release/validate /validate_no_inst && \
    mv target/release/print /print_no_inst && \
    mv target/release/roundtrip /roundtrip_no_inst && \
    mv target/release/incremental-parse /incremental-parse_no_inst && \
    mv target/release/validate-valid-module /validate-valid-module_no_inst && \
    mv target/release/mutate /mutate_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /text-parser /text-parser_no_inst \
                    /validate /validate_no_inst \
                    /print /print_no_inst \
                    /roundtrip /roundtrip_no_inst \
                    /incremental-parse /incremental-parse_no_inst \
                    /validate-valid-module /validate-valid-module_no_inst \
                    /mutate /mutate_no_inst /
