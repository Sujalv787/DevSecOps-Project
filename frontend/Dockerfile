# ============================================================
# Wanderlust Frontend — Vite Dev Server
# Single stage (dev): hot-reload via volume mounts in compose
# ============================================================
FROM node:21-alpine

RUN apk add --no-cache dumb-init

WORKDIR /app

# Copy dependency manifests first (layer cache)
COPY package.json ./

# Install all dependencies (dev needed for Vite)
# --ignore-scripts skips lifecycle hooks like 'prepare' (husky) that fail in Docker
RUN npm install --ignore-scripts

# Copy source (will be overridden by volume mount in compose)
COPY . .

# Expose Vite dev server port
EXPOSE 5173

# Run Vite with host binding so it's accessible outside the container
ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5173"]
