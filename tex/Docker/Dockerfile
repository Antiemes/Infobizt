FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install --no-install-recommends -y \
  make \
  texlive-xetex \
  texlive-science \
  texlive-font-utils \
  ghostscript \
  texlive-lang-european \
  texlive-latex-recommended \
  texlive-latex-extra \
  imagemagick \
  pdftk \
  lmodern \
&& apt-get clean && rm -rf /var/lib/apt/lists/* \
&& sed -i '/^<\/policymap>/i   <policy domain="coder" rights="read | write" pattern="PDF" \/>' /etc/ImageMagick-6/policy.xml
