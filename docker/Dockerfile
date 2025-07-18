ARG NODE_VERSION=20.11.0

# Base Alpine Node image
FROM --platform=linux/amd64 node:${NODE_VERSION}-alpine AS base
LABEL fly_launch_runtime="Next.js/Prisma"

# Install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat openssl
WORKDIR /app

COPY prisma ./prisma
COPY package.json pnpm-lock.yaml* ./
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# Build phase
FROM base AS builder
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/prisma ./prisma
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
ENV DOCKER_OUTPUT 1
ENV SKIP_ENV_VALIDATION 1

RUN corepack enable pnpm && pnpm run build

# Production runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV DOCKER_OUTPUT=1
ENV PORT=3000

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Static files
COPY --from=builder /app/public ./public

# Create and assign permissions to .next
RUN mkdir .next && chown nextjs:nodejs .next

# Copy standalone build
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# Copy Prisma and migration scripts
COPY --from=builder --chown=nextjs:nodejs /app/scripts/migrate.sh ./scripts/migrate.sh
COPY --from=builder --chown=nextjs:nodejs /app/prisma ./prisma

RUN chmod +x ./scripts/migrate.sh

USER nextjs

EXPOSE 3000

# If you want to run migration on container start, uncomment:
# ENTRYPOINT ["./scripts/migrate.sh"]

CMD HOSTNAME="0.0.0.0" node server.js
