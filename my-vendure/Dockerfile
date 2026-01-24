# ============================================
# VENDURE PRODUCTION DOCKERFILE
# Multi-stage build - npm for native modules
# ============================================

# Stage 1: Build with pnpm
FROM node:22-slim AS builder

RUN apt-get update && apt-get install -y \
    python3 \
    make \
    g++ \
    && rm -rf /var/lib/apt/lists/*

RUN corepack enable && corepack prepare pnpm@latest --activate

WORKDIR /app

COPY package.json pnpm-lock.yaml ./

RUN pnpm install --frozen-lockfile

COPY . .

RUN pnpm run build

RUN pnpm run build:dashboard || echo "Dashboard build skipped"

# ============================================
# Stage 2: Production with npm (better native module support)
FROM node:22-slim AS production

RUN apt-get update && apt-get install -y \
    python3 \
    make \
    g++ \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -g 1001 vendure && \
    useradd -u 1001 -g vendure -m vendure

WORKDIR /app

# Copy package.json only (not lock file - let npm resolve)
COPY package.json ./

# Use npm install (not pnpm) - better native module compilation
RUN npm install --omit=dev

# Copy built files from builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/static ./static

RUN mkdir -p /app/static/assets && chown -R vendure:vendure /app

USER vendure

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/shop-api?query=%7B__typename%7D || exit 1

CMD ["sh", "-c", "node dist/index.js & node dist/index-worker.js & wait"]
