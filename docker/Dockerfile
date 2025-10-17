FROM devopsedu/webapp
COPY web/ /var/www/html/
RUN rm -f /var/www/html/index.html || true
CMD ["apache2ctl","-D","FOREGROUND"]
