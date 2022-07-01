FROM arm64v8/node:14.18.3

# crafted and tuned by pierre@ozoux.net and sing.li@rocket.chat
MAINTAINER buildmaster@rocket.chat

RUN groupadd -r rocketchat \
&&  useradd -r -g rocketchat rocketchat \
&&  mkdir -p /app/uploads \
&&  chown rocketchat.rocketchat /app/uploads \
&&  echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf

VOLUME /app/uploads

# gpg: key 4FD08014: public key "Rocket.Chat Buildmaster <buildmaster@rocket.chat>" imported
RUN gpg --batch --keyserver keyserver.ubuntu.com --recv-keys  0E163286C20D07B9787EBE9FD7F9D0414FD08104

WORKDIR /app

RUN ARCH="arm64" apt-get update && apt-get -y install g++ build-essential
RUN ARCH="arm64" apt-get update && apt-get -y upgrade
RUN ARCH="arm64" apt-get install -y curl ca-certificates imagemagick --no-install-recommends 
RUN ARCH="arm64" apt-get install libstdc++6
RUN ARCH="arm64" apt-get -y install python
RUN ARCH="arm64" curl -fSL "https://releases.rocket.chat/4.8.1/download" -o rocket.chat.tgz \
&&  curl -fSL "https://releases.rocket.chat/4.8.1/asc" -o rocket.chat.tgz.asc \
&&  gpg --batch --verify rocket.chat.tgz.asc rocket.chat.tgz \
&&  tar zxvf rocket.chat.tgz \
&&  rm rocket.chat.tgz rocket.chat.tgz.asc
ADD . /app

RUN ARCH="arm64" \
 && set -x \
 && cd /app/bundle/programs/server \
 && npm i npm@6.14.5 \
 # Start hack for sharp...
 && rm -rf npm/node_modules/sharp \
# && rm -rf npm/node_modules/grpc \
 && npm install sharp@0.22.1 \
# && npm install grpc@1.12.2 \
 && mv node_modules/sharp npm/node_modules/sharp \
# && mv node_modules/grpc npm/node_modules/grpc \
 # End hack for sharp
 && cd npm \
 && npm rebuild bcrypt --build-from-source \
 && npm cache clear --force

FROM arm64v8/node

RUN groupadd -r rocketchat \
&&  useradd -r -g rocketchat rocketchat \
&&  mkdir -p /app 

COPY --from=builder /app /app

RUN  mkdir -p /app/uploads \
&&   chown rocketchat.rocketchat /app/uploads 

VOLUME /app/uploads

USER rocketchat

WORKDIR /app/bundle


# needs a mongo instance - defaults to container linking with alias 'mongo'
ENV DEPLOY_METHOD=docker-arm64 \
    NODE_ENV=production \
    MONGO_URL=mongodb://mongo:27017/rocketchat \
    MONGO_OPLOG_URL=mongodb://mongo:27017/local \
    HOME=/tmp \
    PORT=23754 \
    ROOT_URL=http://localhost:23754 \
    Accounts_AvatarStorePath=/app/uploads


EXPOSE 23754

CMD ["node", "main.js"]
