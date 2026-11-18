FROM node:22-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0
RUN corepack enable

FROM base AS build
WORKDIR /app

# 1. Copy root files + only the package.json files needed by api
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY apps/api/package.json ./apps/api/package.json
# If api depends on shared packages, copy their package.json files here too:
# COPY packages/shared/package.json ./packages/shared/package.json

# 2. Install ONLY api's dependencies (and its workspace dependencies)
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm --filter api... install --frozen-lockfile

# 3. Copy source code, build, and bundle production files
COPY . .
RUN pnpm --filter api... build
RUN pnpm --filter api --prod deploy /prod/api

# 4. Final lightweight production image
FROM base AS runtime
WORKDIR /prod/api
COPY --from=build /prod/api .
COPY --from=build /app/apps/api/dist ./dist

EXPOSE 4567
CMD ["node", "dist/main.js"]
