# ==============================
# Build Stage
# ==============================
FROM node:24-alpine3.21 AS build

# Working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy application source
COPY . .

# Build application
RUN npm run build

# ==============================
# Production Stage
# ==============================
FROM nginx:1.28-alpine3.21 AS production

# Apply latest security patches
RUN apk update && apk upgrade --no-cache

# Remove default nginx files
RUN rm -rf /etc/nginx/conf.d/default.conf \
           /usr/share/nginx/html/*

# Create required directories for non-root nginx
RUN mkdir -p /tmp/nginx/client-body \
             /tmp/nginx/proxy \
             /tmp/nginx/fastcgi \
             /tmp/nginx/uwsgi \
             /tmp/nginx/scgi \
             /var/cache/nginx \
             /var/run

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy frontend build
COPY --from=build /app/dist /usr/share/nginx/html

# Set permissions
RUN chown -R nginx:nginx \
    /usr/share/nginx/html \
    /var/cache/nginx \
    /var/run \
    /tmp/nginx \
    /etc/nginx

# Switch to non-root user
USER nginx

# Expose application port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://127.0.0.1:8080/health || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]