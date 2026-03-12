# ============================================
# VENDURE PRODUCTION DOCKERFILE (clean)
# pnpm-only, reproducible, multi-stage
# ============================================

FROM node:20-slim AS builder

RUN apt-get update && apt-get install -y \
  python3 make g++ \
  && rm -rf /var/lib/apt/lists/*

RUN corepack enable && corepack prepare pnpm@9.15.4 --activate

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm run build
RUN pnpm run build:dashboard || echo "Dashboard build skipped"

# ---- runtime image
FROM node:20-slim AS production

RUN apt-get update && apt-get install -y \
  wget \
  && rm -rf /var/lib/apt/lists/*

RUN groupadd -g 1001 vendure && useradd -u 1001 -g vendure -m vendure
WORKDIR /app

# Copy deps from builder and prune to production-only
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /app/node_modules ./node_modules

RUN corepack enable && corepack prepare pnpm@9.15.4 --activate \
  && pnpm prune --prod

COPY --from=builder --chown=vendure:vendure /app/dist ./dist
COPY --from=builder --chown=vendure:vendure /app/static ./static

RUN mkdir -p /app/static/assets && chown -R vendure:vendure /app/static/assets


USER vendure

EXPOSE 3000


CMD ["node", "dist/index.js"]
