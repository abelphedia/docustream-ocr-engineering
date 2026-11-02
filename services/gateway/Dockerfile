# --- Stage 1: Build Recipe ---
FROM rust:1.88-alpine AS chef
RUN apk add --no-cache musl-dev
WORKDIR /app
RUN cargo install cargo-chef --locked

# --- Stage 2: Prepare Planner ---
FROM chef AS planner
COPY Cargo.toml ./
COPY Cargo.lock ./
COPY src ./src
RUN cargo chef prepare --recipe-path recipe.json

# --- Stage 3: Build Dependencies ---
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json
COPY Cargo.toml ./
COPY Cargo.lock ./
COPY src ./src
RUN cargo build --release

# --- Stage 4: Final Image ---
FROM alpine:3.20 AS runtime
RUN apk add --no-cache libgcc
WORKDIR /app
COPY --from=builder /app/target/release/docustream-gateway /app/producer
EXPOSE 5000
CMD ["/app/producer"]

