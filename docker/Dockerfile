FROM rocker/geospatial:4.0.2

ARG BUILD_DATE

LABEL maintainer="wwatkins@usgs.gov"
LABEL version=0.0.1
LABEL org.label-schema.build-date=$BUILD_DATE

RUN apt-get update   

#bring in DOI root cert.  Remove this statement for non-USGS persons
RUN /usr/bin/wget -O /usr/lib/ssl/certs/DOIRootCA.crt http://sslhelp.doi.net/docs/DOIRootCA2.cer && \
  ln -sf /usr/lib/ssl/certs/DOIRootCA.crt /usr/lib/ssl/certs/`openssl x509 -hash -noout -in /usr/lib/ssl/certs/DOIRootCA.crt`.0 && \
  echo "\\n\\nca-certificate = /usr/lib/ssl/certs/DOIRootCA.crt" >> /etc/wgetrc;

RUN install2.r --error \
  googleAuthR \
  googleAnalyticsR \
  remotes \
  usmap \
  rkt \
  aws.s3 \
  aws.signature \
  ineq

RUN Rscript -e 'install.packages("whisker", repos = "https://mran.revolutionanalytics.com/snapshot/2019-04-01")'
RUN Rscript -e 'remotes::install_github("richfitz/remake")'
RUN Rscript -e 'remotes::install_github("USGS-R/grithub")'
RUN Rscript -e 'remotes::install_github("usgs-vizlab/vizlab", ref="v0.3.11", force=TRUE, upgrade = "never")' #don't want to update whisker 


RUN mkdir -p /home/analytics_pipeline
WORKDIR /home/analytics_pipeline
CMD ["/init"]

