FROM ubuntu:latest

ENV TZ=Europe/Paris
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get -y upgrade && apt-get install -yq --no-install-recommends \
	apt-utils \
	curl \
	# Install apache2
	apache2 \
	# Install PHP
	libapache2-mod-php7.2 \
	php7.2-cli \
	php7.2-json \
	php7.2-curl \
	php7.2-mbstring \
	php7.2-zip \
	&& apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN rm /var/www/html/index.html
# ADD ./app /var/www/html
# COPY ./app/ /var/www/html

RUN service apache2 restart

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

EXPOSE 80
