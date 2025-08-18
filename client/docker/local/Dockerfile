FROM docker.io/node:20.11-alpine3.18 AS base

FROM base AS node-build-stage

RUN apk add --no-cache libc6-compat

COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./

RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i; \
  else echo "Lockfile not found." && exit 1; \
  fi

FROM base AS node-run-stage

ARG APP_HOME=/app


COPY --from=node-build-stage /node_modules ./node_modules

COPY . ${APP_HOME}

RUN mkdir -p /app/.next && \
    chown -R node:node /app/.next
WORKDIR ${APP_HOME}

RUN addgroup -g 1001 -S nodejs && \
  adduser -S nextjs -u 1001 -G nodejs

RUN chown -R node:node ${APP_HOME}



USER node
EXPOSE 3000
CMD [ "npm","run","dev" ]


