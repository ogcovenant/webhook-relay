FROM node:22-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
RUN corepack enable

FROM base AS build
WORKDIR /app

# 1. Copy root files + only the package.json files needed by web
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY apps/web/package.json ./apps/web/package.json

# 2. Install ONLY web's dependencies (and its workspace dependencies)
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm --filter web... install --frozen-lockfile

# 3. Copy source code
COPY . .

# 4. Declare build args and set them as ENV before Next.js builds
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

# 5. Build and bundle production files
RUN pnpm --filter web... build
RUN pnpm --filter web --prod deploy /prod/web

# 6. Copy the built .next folder (pnpm deploy ignores gitignored folders like .next by default)
COPY --from=build /app/apps/web/.next /prod/web/.next

# 7. Final lightweight production image
FROM base AS runtime
WORKDIR /prod/web
COPY --from=build /prod/web .

EXPOSE 3000
CMD ["pnpm", "start"]
