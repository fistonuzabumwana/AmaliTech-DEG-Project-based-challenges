FROM node:20-alpine

# Set working directory inside container
WORKDIR /app

# Copy package files first
COPY app/package*.json ./

# Install dependencies
RUN npm install --omit=dev

# Copy the rest of the app
COPY app/ .

# Create non-root user (important requirement)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Start the app
CMD ["npm", "start"]
