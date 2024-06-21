# The runtime image.
FROM php:8.1.8-apache-buster

# The koel version to download
ARG KOEL_VERSION_REF=v6.12.1

# Install necessary packages and clean up
RUN apt-get update && apt-get install -y --no-install-recommends \
    vim \
    git \
    unzip \
    libapache2-mod-xsendfile \
    libzip-dev \
    zip \
    ffmpeg \
    locales \
    libpng-dev \
    libjpeg62-turbo-dev \
    libpq-dev \
    && docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install \
    bcmath \
    exif \
    gd \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    zip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Download the koel release and remove unnecessary files
RUN curl -L https://github.com/koel/koel/releases/download/${KOEL_VERSION_REF}/koel-${KOEL_VERSION_REF}.tar.gz | tar -xz -C /tmp \
  && cd /tmp/koel/ \
  && rm -rf .editorconfig \
    .eslintignore \
    .eslintrc \
    .git \
    .gitattributes \
    .github \
    .gitignore \
    .gitmodules \
    .gitpod.dockerfile \
    .gitpod.yml \
    api-docs \
    cypress \
    cypress.json \
    nginx.conf.example \
    package.json \
    phpstan.neon.dist \
    phpunit.xml.dist \
    resources/artifacts/ \
    ruleset.xml \
    scripts/ \
    tag.sh \
    tests \
    vite.config.js

# Create necessary directories with correct permissions
RUN mkdir /music && chown www-data:www-data /music \
  && mkdir -p /var/www/html/storage/search-indexes \
  && chown www-data:www-data /var/www/html/storage/search-indexes \
  && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && /usr/sbin/locale-gen

# Copy Apache configuration
COPY ./apache.conf /etc/apache2/sites-available/000-default.conf

# Copy php.ini
COPY ./php.ini "$PHP_INI_DIR/php.ini"

# Enable Apache rewrite module
RUN a2enmod rewrite

# Copy the downloaded release to the web directory
RUN cp -R /tmp/koel/. /var/www/html && chown -R www-data:www-data /var/www/html

# Define volumes
VOLUME ["/music", "/var/www/html/storage/search-indexes"]

# Environment variables
ENV FFMPEG_PATH=/usr/bin/ffmpeg \
    MEDIA_PATH=/music \
    STREAMING_METHOD=x-sendfile \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# Copy entrypoint script and set entrypoint
COPY ./koel-entrypoint /usr/local/bin/
RUN chmod +x /usr/local/bin/koel-entrypoint
ENTRYPOINT ["koel-entrypoint"]
CMD ["apache2-foreground"]

# Expose port 80
EXPOSE 80

# Healthcheck to ensure the server is running
HEALTHCHECK --interval=5m --timeout=5s \
  CMD curl -f http://localhost/ || exit 1
