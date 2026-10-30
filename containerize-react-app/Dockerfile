# 1. Build our production bundle
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json .

# Temporarily adding --force to avoid underlying dependency conflicts
# due to latest versions of react-scripts.
RUN npm ci --force

COPY . .
RUN npm run build

# 2. Serve the bundle with an HTTP server
FROM nginx:1.27.0

COPY --from=build /app/build /usr/share/nginx/html