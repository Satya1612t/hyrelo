FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
# `npm ci` triggers the `postinstall: prisma generate` script (see
# package.json), which needs prisma/schema.prisma present to find — it
# never connects to a database, so a syntactically valid placeholder
# DATABASE_URL is enough here.
COPY prisma ./prisma
ENV DATABASE_URL="postgresql://placeholder:placeholder@placeholder:5432/placeholder"
RUN npm ci

FROM node:20-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV DATABASE_URL="postgresql://placeholder:placeholder@placeholder:5432/placeholder"
RUN npx prisma generate
# NEXT_PUBLIC_* vars are inlined into the JS bundle at build time (not read
# at container runtime like everything else in docker-compose.prod.yml), so
# they have to arrive as build args here — see the `admin.build.args` block
# in docker-compose.prod.yml. Without this, the Firebase client SDK
# initializes with an empty apiKey during prerendering and crashes the
# build with "Firebase: Error (auth/invalid-api-key)".
ARG NEXT_PUBLIC_FIREBASE_API_KEY
ARG NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN
ARG NEXT_PUBLIC_FIREBASE_PROJECT_ID
ARG NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET
ARG NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID
ARG NEXT_PUBLIC_FIREBASE_APP_ID
ARG NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID
ENV NEXT_PUBLIC_FIREBASE_API_KEY=$NEXT_PUBLIC_FIREBASE_API_KEY
ENV NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=$NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN
ENV NEXT_PUBLIC_FIREBASE_PROJECT_ID=$NEXT_PUBLIC_FIREBASE_PROJECT_ID
ENV NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=$NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET
ENV NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=$NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID
ENV NEXT_PUBLIC_FIREBASE_APP_ID=$NEXT_PUBLIC_FIREBASE_APP_ID
ENV NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID=$NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/.next/standalone ./
COPY --from=build /app/.next/static ./.next/static
COPY --from=build /app/node_modules/.prisma ./node_modules/.prisma
EXPOSE 3000
CMD ["node", "server.js"]
