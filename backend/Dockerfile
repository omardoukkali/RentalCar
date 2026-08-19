# =============================================================================
#  Backend — Laravel 12 sur PHP 8.3 (FPM, Alpine)
#  Image légère fournissant PHP-FPM, Composer et les extensions requises
#  par Laravel et PostgreSQL.
# =============================================================================
FROM php:8.4-fpm-alpine

# --- Dépendances système nécessaires à la compilation des extensions PHP ------
RUN apk add --no-cache \
        bash \
        git \
        curl \
        unzip \
        icu-dev \
        oniguruma-dev \
        libzip-dev \
        postgresql-dev

# --- Extensions PHP requises --------------------------------------------------
#  pdo_pgsql / pgsql : connexion à PostgreSQL
#  intl              : internationalisation (validation, formats)
#  mbstring          : chaînes multi-octets (Laravel en dépend)
#  bcmath            : calculs de précision (montants, commissions)
#  zip / opcache     : archives et cache d'opcodes (performances)
RUN docker-php-ext-install \
        pdo \
        pdo_pgsql \
        pgsql \
        intl \
        mbstring \
        bcmath \
        zip \
        opcache

# --- Composer (copié depuis l'image officielle) -------------------------------
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# En développement, le code est monté via docker-compose (bind mount).
# Les dépendances s'installent au premier lancement :  composer install
# (voir README.md, section « Premier démarrage »).

EXPOSE 9000
CMD ["php-fpm"]
