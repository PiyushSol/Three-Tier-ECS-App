# Stage 1 - Install dependencies
FROM node:14-alpine AS builder

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

# Stage 2 - Production image
FROM node:14-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --production

COPY --from=builder /usr/src/app .

CMD ["node", "index.js"]