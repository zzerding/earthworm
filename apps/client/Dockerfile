# ABOUTME: Multi-stage Dockerfile for the Earthworm Client (Nuxt 3 static SPA + pnpm monorepo)
# ABOUTME: Builds Nuxt to static files in build stage, then serves with nginx in runtime stage

# Build stage
FROM node:22.21.1-alpine AS builder

# Install pnpm
RUN corepack enable && corepack prepare pnpm@9.3.0 --activate

WORKDIR /app

# Copy workspace configuration files
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./

# Copy all workspace packages and apps for dependency resolution
COPY packages ./packages
COPY apps/client ./apps/client

# Install all dependencies
# Set CYPRESS_INSTALL_BINARY=0 to skip Cypress binary download in production builds
ENV CYPRESS_INSTALL_BINARY=0
RUN pnpm install --frozen-lockfile

# Build the client application (generates static files to .output/public)
WORKDIR /app/apps/client

# Accept build arguments for environment variables
#ARG API_BASE
#ARG LOGTO_APP_ID
#ARG LOGTO_ENDPOINT
#ARG BACKEND_ENDPOINT
#ARG LOGTO_SIGN_IN_REDIRECT_URI
#ARG LOGTO_SIGN_OUT_REDIRECT_URI
#ARG HELP_DOCS_URL
#ARG NODE_ENV

# Set environment variables for Nuxt build
#ENV API_BASE=${API_BASE}
#ENV LOGTO_APP_ID=${LOGTO_APP_ID}
#ENV LOGTO_ENDPOINT=${LOGTO_ENDPOINT}
#ENV BACKEND_ENDPOINT=${BACKEND_ENDPOINT}
#ENV LOGTO_SIGN_IN_REDIRECT_URI=${LOGTO_SIGN_IN_REDIRECT_URI}
#ENV LOGTO_SIGN_OUT_REDIRECT_URI=${LOGTO_SIGN_OUT_REDIRECT_URI}
#ENV HELP_DOCS_URL=${HELP_DOCS_URL}
#ENV NODE_ENV=${NODE_ENV}

RUN pnpm run generate

# Production stage - serve with nginx
FROM nginx:alpine AS runner

# Copy nginx configuration for SPA routing
COPY apps/client/nginx.conf /etc/nginx/conf.d/default.conf

# Copy built static files from builder stage
COPY --from=builder /app/apps/client/.output/public /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
