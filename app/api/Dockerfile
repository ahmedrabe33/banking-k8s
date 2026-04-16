# ----------- Stage 1: Build -----------
FROM node:18-alpine AS builder

WORKDIR /usr/src/app

# Install dependencies with clean cache
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force

# Copy source code
COPY . .

# ----------- Stage 2: Runtime -----------
FROM node:18-alpine

# Set environment
ENV NODE_ENV=production

WORKDIR /usr/src/app

# Install tini (for signal handling)
RUN apk add --no-cache tini

# Create non-root user with no shell
RUN addgroup -S nodeuser && \
 adduser -S nodeuser -G nodeuser -s /sbin/nologin

# Copy only required files with correct ownership
COPY --from=builder --chown=nodeuser:nodeuser /usr/src/app ./

# Remove unnecessary files (extra hardening)
RUN rm -rf /usr/src/app/.git \
 /usr/src/app/node_modules/.cache \
 /tmp/*

# Switch to non-root user
USER nodeuser

# Use tini as init system
ENTRYPOINT ["/sbin/tini", "--"]

EXPOSE 3000

CMD ["node", "app.js"]
