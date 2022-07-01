FROM arm64v8/node:14.18.3-stretch as builder

RUN npm i npm@6.14.5 \
&& sudo apk add python make g++ \
&& npm install fibers@5.0.1
