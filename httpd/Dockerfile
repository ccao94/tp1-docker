FROM httpd:2.4-alpine

# Install required proxy modules (mod_proxy and mod_proxy_http)
# Alpine uses 'apk' instead of 'apt', and we need to enable modules manually
RUN apk add --no-cache apache2-utils && \
    echo "LoadModule proxy_module modules/mod_proxy.so" >> /usr/local/apache2/conf/httpd.conf && \
    echo "LoadModule proxy_http_module modules/mod_proxy_http.so" >> /usr/local/apache2/conf/httpd.conf

# Copy the custom Apache configuration file
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf

# Copy a default homepage for testing
COPY ./index.html /usr/local/apache2/htdocs/index.html
