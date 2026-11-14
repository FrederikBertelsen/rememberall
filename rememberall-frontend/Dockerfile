# Build stage
FROM node:20-alpine AS build

WORKDIR /app

# Install pnpm globally first
RUN npm install -g pnpm@12.18.0

# Copy package files
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build the SvelteKit application
RUN pnpm run build

# Runtime stage
FROM node:20-alpine AS runtime

WORKDIR /app

# Install pnpm globally
RUN npm install -g pnpm@12.18.0

# Copy package files
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./

# Install production dependencies only
RUN pnpm install --frozen-lockfile --prod

# Copy built application from build stage
COPY --from=build /app/build ./

EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "index.js"]
