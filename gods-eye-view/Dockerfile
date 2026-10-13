# God's Eye View (https://github.com/bilawalsidhu/gods-eye-view)
# Upstream ships no image. The app is served by Vite with its /api provider
# middleware, so the container runs the Vite server exactly like `npm run dev`.
# API keys are read from the environment at runtime — none are baked in.
FROM node:24-slim

ARG GEV_REPO=https://github.com/bilawalsidhu/gods-eye-view.git
# Branch, tag or commit. CI passes the upstream commit so the clone layer is
# rebuilt only when upstream actually changes.
ARG GEV_REF=main

ENV PUPPETEER_SKIP_DOWNLOAD=1 \
    NODE_ENV=development \
    HOST=0.0.0.0 \
    PORT=4173

RUN apt-get update \
 && apt-get install -y --no-install-recommends git ca-certificates \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN git init -q . \
 && git fetch -q --depth 1 "${GEV_REPO}" "${GEV_REF}" \
 && git checkout -q FETCH_HEAD \
 && rm -rf .git \
 && npm ci --no-audit --no-fund \
 && npm cache clean --force

EXPOSE 4173
CMD ["sh", "-c", "exec npx vite --host \"$HOST\" --port \"$PORT\" --strictPort"]
