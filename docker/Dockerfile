FROM ubuntu:focal

ENV TZ=US/Eastern
ENV MATHICS_HOME=/usr/src/app
ENV ENTRYPOINT_COMMAND="docker run -it {MATHICS_IMAGE}"

WORKDIR $MATHICS_HOME

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

COPY requirements.txt ./
RUN apt-get update
RUN apt-get install -qq apt-utils
# we need libsqlite3-dev now if ubuntu doesn't come with that, we'll need
# to build our own Python
RUN DEBIAN_FRONTEND="noninteractive" TZ="US/Eastern" apt-get install -qq liblapack-dev llvm-10 llvm-10-dev gfortran maria libmysqlclient-dev sqlite3 python3-pip python3-setuptools git python3-matplotlib python-matplotlib-data x11-apps asymptote w3m
RUN pip install --no-cache-dir -r requirements.txt
COPY requirements-mathicsscript.txt ./
RUN pip install --no-cache-dir -r requirements-mathicsscript.txt

RUN pip install "Mathics3[full]>=3.0.0"
# RUN pip install git+git://github.com/mathics/Mathics.git#egg=Mathics3
RUN python3 -m pip install -e git://github.com/Mathics3/mathics-django#egg=Mathics-Django[full]
# RUN pip install Mathics-Django[full]
RUN pip install pymathics-natlang
RUN pip install pymathics-graph
RUN (cd src/mathics-django && python3 mathics_django/docpipeline.py -o)
# RUN python3 -m mathics_django.docpipeline.py -o
RUN mkdir -p /usr/src/app/.local/var/mathics && cp -v /root/.local/var/mathics/doc_xml_data.pcl /usr/src/app/.local/var/mathics/doc_xml_data.pcl

# RUN python3 -m pip install -e git://github.com/Mathics3/mathicsscript#egg=mathicsscript
RUN pip install mathicsscript[full]
# RUN (cd src/mathicsscript && make)

RUN python3 -m nltk.downloader wordnet omw
RUN python3 -m spacy download en

# debug stuff
# RUN apt-get install -qq emacs
# RUN pip install trepan3k

EXPOSE 8000

RUN groupadd mathics && \
    useradd -d $MATHICS_HOME -g mathics -m -s /bin/bash mathics && \
    mkdir -p $MATHICS_HOME/data && \
    chown -R mathics:mathics $MATHICS_HOME

USER mathics
COPY django-db/mathics.sqlite /usr/src/app/.local/var/mathics/mathics.sqlite

ENTRYPOINT ["/entrypoint.sh"]

CMD ["--help"]
