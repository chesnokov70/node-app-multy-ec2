FROM nginx:alpine

RUN apk add --no-cache tzdata && \
cp /usr/share/zoneinfo/America/New_York /etc/localtime && \
echo "America/New_York" > /etc/timezone && \
apk del tzdata

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
