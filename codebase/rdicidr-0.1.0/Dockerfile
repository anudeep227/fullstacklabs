# ---- Base builder: install deps, build app ----
FROM node:20-alpine AS builder

# For reproducible installs & faster caching
ENV NODE_ENV=development
WORKDIR /app

# Install OS deps you might need (optional; comment out if unused)
# RUN apk add --no-cache python3 make g++

# Install only from lockfile for deterministic builds
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Copy sources and build
COPY . .
# If your build is optional, keep as-is; otherwise ensure it exists
RUN npm run build

# ---- Runtime: minimal image with only prod deps + build artifacts ----
FROM node:20-alpine AS runtime

ENV NODE_ENV=production \
    # change if your app lets PORT be configured
    PORT=8080

WORKDIR /app

# Copy only package files first to install prod deps (cache-friendly)
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev

# Copy compiled app (adjust if your output folder differs, e.g., dist/ or build/)
COPY --from=builder /app/dist ./dist
# If the app needs any non-compiled assets, copy them too:
# COPY --from=builder /app/public ./public

# Drop privileges
RUN addgroup -S nodejs && adduser -S node -G nodejs
USER node

# App should listen on 8080 to match the k8s Service
EXPOSE 8080

# If your start script is "node dist/index.js" this will just work
CMD ["npm", "start"]