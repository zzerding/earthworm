# ABOUTME: Multi-stage Dockerfile for the Earthworm API (NestJS + pnpm monorepo)
# ABOUTME: Builds TypeScript to JavaScript in build stage, then creates minimal runtime image

# Build stage
FROM node:22.21.1-alpine AS builder

# Install pnpm
RUN corepack enable && corepack prepare pnpm@9.3.0 --activate

WORKDIR /app

# Copy workspace configuration files
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./

# Copy all workspace packages and apps for dependency resolution
COPY packages ./packages
COPY apps/api ./apps/api

# Install all dependencies (including dev dependencies for build)
RUN pnpm install --frozen-lockfile

# Build the API application
WORKDIR /app/apps/api
RUN pnpm run build

# Production stage
FROM node:22.21.1-alpine AS runner

# Install pnpm for production dependency installation
RUN corepack enable && corepack prepare pnpm@9.3.0 --activate

WORKDIR /app

# Copy workspace configuration files
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./

# Copy workspace packages (needed for workspace dependencies)
COPY packages ./packages

# Copy API package.json
COPY apps/api/package.json ./apps/api/

# Install only production dependencies
RUN pnpm install --prod --ignore-scripts

# Copy built application from builder stage
COPY --from=builder /app/apps/api/dist ./apps/api/dist

# Set working directory to API app
WORKDIR /app/apps/api

# Expose the API port
EXPOSE 3001

# Run the application
CMD ["node", "dist/src/main.js"]
