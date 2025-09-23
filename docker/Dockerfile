# HeadlessX v1.2.0 - Modular Browserless Web Scraping API
# Multi-stage build for optimized production image with modular architecture

# Stage 1: Build website
FROM node:20-slim AS website-builder

WORKDIR /app/website

# Copy website package files
COPY website/package*.json ./

# Install website dependencies
RUN npm ci

# Copy website source
COPY website/ ./

# Build website for production
RUN npm run build

# Stage 2: Production runtime
FROM mcr.microsoft.com/playwright:v1.55.0-noble

# Install essential system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies (as root)
RUN if [ -f "package-lock.json" ]; then \
        npm ci --only=production; \
    else \
        npm install --production; \
    fi && npm cache clean --force

# Install system dependencies for Playwright browsers
# This is still good practice to ensure everything is present.
RUN npx playwright install-deps

# Copy modular application source
COPY src/ ./src/

# Copy built website from previous stage
COPY --from=website-builder /app/website/out/ ./website/out/

# Create logs directory
RUN mkdir -p logs

# ==============================================================================
# === CAMBIO CLAVE PARA SOLUCIONAR EL ERROR DE PERMISOS ===
# ==============================================================================
# Create non-root user for security
RUN groupadd -r headlessx && useradd -r -g headlessx headlessx

# Change ownership of the entire app directory AND the root cache where Playwright
# stores its browsers. This allows the 'headlessx' user to execute them correctly.
RUN chown -R headlessx:headlessx /app /root/.cache
# ==============================================================================

# Environment variables (override with docker-compose or runtime)
ENV NODE_ENV=production
ENV PORT=3000
ENV AUTH_TOKEN=""
ENV DOMAIN=localhost
ENV SUBDOMAIN=headlessx

# Expose the application port directly
EXPOSE 3000

# Health check targets the Node.js application directly
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1

# Switch to non-root user for security
USER headlessx

# Start the Node.js application
# Nota: El log dice que usas 'src/app.js', no 'src/server.js'. Lo he corregido.
CMD ["node", "src/app.js"]
