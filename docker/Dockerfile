FROM ubuntu:16.04
MAINTAINER Dave Kincaid <kincaid.dave@gmail.com>
LABEL version="0.1"

ENV anafora_version 1.1
ENV anafora_home /home/anafora
ENV anafora_app $anafora_home/anafora-$anafora_version

RUN apt-get update && apt-get install -y \
    curl \
    apache2 \
    apache2-utils \
    python-django \
    libapache2-mod-wsgi

# Apache setup
RUN ln -sf /etc/apache2/mods-available/auth_digest.load /etc/apache2/mods-enabled/auth-digest.load
COPY apache/000-default.conf /etc/apache2/sites-available/
COPY apache/alias.conf /etc/apache2/mods-available/
RUN echo "WSGIScriptAlias /anafora /home/anafora/anafora-1.1/src/main/web/wsgi.py" >> /etc/apache2/apache2.conf

# Anafora setup stuff
RUN groupadd anaforaadmin
RUN useradd -m -g anaforaadmin -d $anafora_home anafora
RUN curl -SL https://github.com/weitechen/anafora/archive/v${anafora_version}.tar.gz \
     | tar -xzC $anafora_home

RUN htdigest -c $anafora_app/anafora.htdigest anafora anafora
RUN echo "annotatorGroup: anafora" > $anafora_app/anafora.authgroup

RUN chgrp -R www-data $anafora_app
RUN chmod -R g+rw $anafora_app

RUN mkdir $anafora_home/project_root
RUN chown anafora.www-data $anafora_home/project_root
COPY anafora/sampleanaforaprojectfile $anafora_home/project_root
RUN chmod -R g+rw $anafora_home/project_root

RUN export PYTHONPATH=$anafora_app

# TODO: need to figure out how to use the variables for the anafora app directory
COPY anafora/settings.py $anafora_app/src/main
RUN sed -i "s/TIME_ZONE.*/TIME_ZONE = 'America\/Chicago'/g" $anafora_app/src/main/settings.py
RUN sed -i "s/STATIC_ROOT.*/STATIC_ROOT = '\/home\/anafora\/anafora-1.1\/src\/main\/static'/g" $anafora_app/src/main/settings.py
RUN sed -i "s/STATIC_URL.*/STATIC_URL = '\/static\/'/" $anafora_app/src/main/settings.py
RUN sed -i "s/ROOT_URL.*/ROOT_URL = '\/anafora'/" $anafora_app/src/main/settings.py
RUN sed -i "s/SECRET_KEY.*/SECRET_KEY = 'changeme'/" $anafora_app/src/main/settings.py
RUN sed -i "s/TEMPLATE_DIRS.*/TEMPLATE_DIRS = ('\/home\/anafora\/anafora-1.1\/src\/Templates',)/" $anafora_app/src/main/settings.py
RUN sed -i "s/ANAFORA_PROJECT_FILE_ROOT.*/ANAFORA_PROJECT_FILE_ROOT = '\/home\/anafora\/project_root'/" $anafora_app/src/main/settings.py
RUN sed -i "s/GROUP_FILE.*/GROUP_FILE = '\/home\/anafora\/anafora-1.1\/anafora.authgroup/" $anafora_app/src/main/settings.py
RUN sed -i "s/ADMIN_GROUPNAME.*/ADMIN_GROUPNAME = 'anaforaadmin'/" $anafora_app/src/main/settings.py
RUN sed -i "s/ANAFORA_PROJECT_SETTING_FILENAME.*/ANAFORA_PROJECT_SETTING_FILENAME = '.setting.xml' /" $anafora_app/src/main/settings.py



# Run Apache web server
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]