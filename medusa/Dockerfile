# Drop-in replacement for medusa/Dockerfile
# Difference from the original: the build-time migration step is removed.
# Migrations now run at container startup via migrations.sh (see docker-compose.yml),
# so the image can be built without a live database.

FROM node:23 AS builder

WORKDIR /app/medusa

COPY . .

RUN rm -rf node_modules

RUN apt-get update && apt-get install -y python3 python3-pip python-is-python3

RUN yarn install --loglevel=error

RUN yarn run build


FROM node:23

WORKDIR /app/medusa

RUN mkdir .medusa

RUN apt-get update && apt-get install -y python3 python3-pip python-is-python3

COPY --from=builder /app/medusa/.medusa ./.medusa

WORKDIR /app/medusa/.medusa/server

RUN yarn install --production

# Keep migrations.sh in the image; it is used as the container entrypoint.
# It runs `medusa db:migrate` only when WORKER_MODE != worker, then execs the CMD.
COPY migrations.sh ./migrations.sh
RUN chmod +x ./migrations.sh

CMD ["yarn", "run", "start"]