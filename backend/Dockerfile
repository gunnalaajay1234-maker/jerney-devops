# ================================
# Build Stage
# ================================
FROM node:22-alpine AS build

WORKDIR /app

# Copy dependency files first
COPY package*.json ./

# Install only production dependencies
RUN npm ci --omit=dev \
    && npm cache clean --force

# Copy application source
COPY src ./src

# ================================
# Production Stage
# ================================
FROM node:22-alpine

# Install security updates + dumb-init
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

# Copy app from build stage
COPY --from=build /app /app

# Set ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Environment
ENV NODE_ENV=production

# Expose app port
EXPOSE 5000

# Use dumb-init for signal handling
ENTRYPOINT ["dumb-init", "--"]

# Start app
CMD ["node", "src/index.js"]