FROM node:18-alpine

# Install curl for health checks
RUN apk add --no-cache curl

# Create app directory with proper permissions
WORKDIR /app

# Use existing node user (UID 1000)

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install --only=production && \
    npm cache clean --force && \
    rm -rf /tmp/*

# Copy source code
COPY src/ ./src/

# Create writable directories for the app
RUN mkdir -p /tmp /app/.npm && \
    chown -R node:node /app /tmp

# Switch to non-root user
USER node

# Add health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000

CMD ["npm", "start"]