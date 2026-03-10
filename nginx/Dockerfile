FROM nginx:alpine

# Remove default config
RUN rm /etc/nginx/conf.d/default.conf

# Copy our custom config
COPY nginx.conf /etc/nginx/nginx.conf

# Create directory for map files
RUN mkdir -p /var/www/html/park

EXPOSE 80 443
