# ============================================================
# Stage 1 — Builder
# Install ALL deps (including devDeps), run tests, prune after
# ============================================================
FROM node:21-alpine AS builder

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

WORKDIR /app

# Copy dependency manifests first (layer cache optimisation)
COPY package.json ./

# Install ALL dependencies (dev + prod needed for tests)
# Note: no package-lock.json in repo (.gitignore'd), so we use npm install
RUN npm install

# Copy the rest of the source
COPY . .

# Run tests in CI mode (--watchAll=false so it exits after one run)
RUN CI=true npm test -- --watchAll=false --passWithNoTests

# Prune dev dependencies to get a clean prod-only node_modules
RUN npm prune --omit=dev


# ============================================================
# Stage 2 — Production Runtime
# Minimal alpine image; copy only what we need
# ============================================================
FROM node:21-alpine AS runtime

# Install dumb-init (PID 1 signal forwarding)
RUN apk add --no-cache dumb-init

# Set NODE_ENV for production optimisations
ENV NODE_ENV=production

WORKDIR /app

# Copy entire app from builder (includes prod-only node_modules after npm prune)
COPY --from=builder /app .

# Drop root — run as built-in non-root 'node' user
USER node

# Expose the backend port
EXPOSE 8080

# Health check — hits the root route every 30s
HEALTHCHECK --interval=30s --timeout=10s --start-period=20s --retries=3 \
  CMD wget -qO- http://localhost:8080/ || exit 1

# Use dumb-init as PID 1 to forward signals correctly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
