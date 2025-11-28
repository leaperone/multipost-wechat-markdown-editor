FROM node:20.18.1-alpine3.20 AS base
RUN apk update --no-cache && apk add --no-cache \
    tzdata \
    make \
    g++ \
    pixman-dev \
    cairo-dev \
    pango-dev \
    pkgconfig \
    curl

# Install pnpm using npm (more reliable than corepack in Docker)
RUN npm install -g pnpm@latest

COPY . /app
WORKDIR /app

FROM base AS builder
ENV NODE_OPTIONS="--max-old-space-size=8192"

# Install dependencies using pnpm
RUN pnpm install --frozen-lockfile

# Build the web app
RUN pnpm --filter @md/web run build:h5-netlify:only

FROM nginx:alpine AS final
ENV TZ=Asia/Shanghai

# Copy nginx configuration
COPY docker/nginx.conf /etc/nginx/conf.d/default.conf

# Copy build artifacts from the new location
COPY --from=builder /app/apps/web/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
