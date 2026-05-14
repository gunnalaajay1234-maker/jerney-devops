# ================================
# Build Stage
# ================================
FROM node:22-alpine AS build

WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --omit=dev \
    && npm cache clean --force

# Copy application source
COPY src ./src

# ================================
# Production Stage
# ================================
FROM node:22-alpine

# Install required packages with pinned versions
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3

# Create non-root user and group
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

# Copy application from build stage
COPY --from=build /app /app

# Set ownership
RUN chown -R appuser:appgroup /app

# Use non-root user
USER appuser

# Production environment
ENV NODE_ENV=production

# Application port
EXPOSE 5000

# Proper signal handling
ENTRYPOINT ["dumb-init", "--"]

# Start application
CMD ["node", "src/index.js"]