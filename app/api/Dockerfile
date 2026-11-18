
FROM node:22-alpine3.24 AS build

WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev --no-audit --no-fund && npm cache clean --force

COPY src ./src

FROM alpine:3.24.4

RUN apk add --no-cache libstdc++ libgcc
WORKDIR /app

COPY --from=build /usr/local/bin/node /usr/local/bin/node
COPY --from=build /usr/lib/libstdc++.so.6 /usr/lib/
COPY --from=build /usr/lib/libgcc_s.so.1 /usr/lib/
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY --from=build /app/package.json ./package.json

ENV NODE_ENV=production

USER 1000:1000

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
    CMD node -e "fetch('http://127.0.0.1:3000/api/health').then(r => process.exit(r.ok ? 0 : 1)).catch(() => process.exit(1))"

CMD ["sh", "-c", "node src/migrate.js && node src/server.js"]
