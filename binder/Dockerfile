FROM pangeo/pangeo-notebook:2020.04.22

RUN echo "Checking for 'start'..." \
        ; [ -d binder ] && cd binder \
        ; if test -f "start" ; then \
        chmod +x start \
        && cp start /srv/start \
        ; fi
