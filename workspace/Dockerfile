# Based on laradock
#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#
ARG PHP_VERSION
ARG BASE_IMAGE_TAG_PREFIX=latest
FROM laradock/workspace:${BASE_IMAGE_TAG_PREFIX}-${PHP_VERSION}

LABEL maintainer="Tuan Anh Nguyen <nguyenhuutuananh2@gmail.com>"

ARG PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

ARG NON_ROOT_USER=php-docker

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} ${NON_ROOT_USER} && \
    useradd -l -u ${PUID} -g ${NON_ROOT_USER} -m ${NON_ROOT_USER} -G docker_env && \
    usermod -p "*" ${NON_ROOT_USER} -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
      # installed on the base image 'laradock/workspace' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/workspace).
      #
      # next lines are here because there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip \
      # Install the zip extension
      php${PHP_VERSION}-zip \
      # nasm
      nasm && \
      php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/${NON_ROOT_USER}/.composer/composer.json

# Add the auth.json for magento 2 credentials
COPY ./auth.json /home/${NON_ROOT_USER}/.composer/auth.json

# Make sure that ~/.composer belongs to ${NON_ROOT_USER}
RUN chown -R ${NON_ROOT_USER}:${NON_ROOT_USER} /home/${NON_ROOT_USER}/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}
RUN set -eux; \
      if [ "$COMPOSER_VERSION" = "1" ] || [ "$COMPOSER_VERSION" = "2" ]; then \
          composer self-update --${COMPOSER_VERSION}; \
      else \
          composer self-update ${COMPOSER_VERSION}; \
      fi

USER ${NON_ROOT_USER}

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

# Check if auth file is disabled
ARG COMPOSER_AUTH_JSON=false
ENV COMPOSER_AUTH_JSON ${COMPOSER_AUTH_JSON}

RUN if [ ${COMPOSER_AUTH_JSON} = false ]; then \
    # remove the file
    rm /home/${NON_ROOT_USER}/.composer/auth.json \
;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

USER root

###########################################################################
# ssh:
###########################################################################

ARG INSTALL_WORKSPACE_SSH=false

COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/${NON_ROOT_USER} \
    && chown -R ${NON_ROOT_USER}:${NON_ROOT_USER} /home/${NON_ROOT_USER}/.ssh \
;fi

###########################################################################
# Oh My ZSH!
###########################################################################

USER root

ARG SHELL_OH_MY_ZSH=false
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
    apt install -y zsh \
;fi

ARG SHELL_OH_MY_ZSH_AUTOSUGESTIONS=false
ARG SHELL_OH_MY_ZSH_ALIASES=false

USER ${NON_ROOT_USER}
RUN if [ ${SHELL_OH_MY_ZSH} = true ]; then \
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh) --keep-zshrc" && \
    sed -i -r 's/^plugins=\(.*?\)$/plugins=(laravel composer)/' /home/${NON_ROOT_USER}/.zshrc && \
    echo '\n\
bindkey "^[OB" down-line-or-search\n\
bindkey "^[OC" forward-char\n\
bindkey "^[OD" backward-char\n\
bindkey "^[OF" end-of-line\n\
bindkey "^[OH" beginning-of-line\n\
bindkey "^[[1~" beginning-of-line\n\
bindkey "^[[3~" delete-char\n\
bindkey "^[[4~" end-of-line\n\
bindkey "^[[5~" up-line-or-history\n\
bindkey "^[[6~" down-line-or-history\n\
bindkey "^?" backward-delete-char\n' >> /home/${NON_ROOT_USER}/.zshrc && \
  if [ ${SHELL_OH_MY_ZSH_AUTOSUGESTIONS} = true ]; then \
    sh -c "git clone https://github.com/zsh-users/zsh-autosuggestions /home/${NON_ROOT_USER}/.oh-my-zsh/custom/plugins/zsh-autosuggestions" && \
    sed -i 's~plugins=(~plugins=(zsh-autosuggestions ~g' /home/${NON_ROOT_USER}/.zshrc && \
    sed -i '1iZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20' /home/${NON_ROOT_USER}/.zshrc && \
    sed -i '1iZSH_AUTOSUGGEST_STRATEGY=(history completion)' /home/${NON_ROOT_USER}/.zshrc && \
    sed -i '1iZSH_AUTOSUGGEST_USE_ASYNC=1' /home/${NON_ROOT_USER}/.zshrc && \
    sed -i '1iTERM=xterm-256color' /home/${NON_ROOT_USER}/.zshrc \
  ;fi && \
  if [ ${SHELL_OH_MY_ZSH_ALIASES} = true ]; then \
    echo "" >> /home/${NON_ROOT_USER}/.zshrc && \
    echo "# Load Custom Aliases" >> /home/${NON_ROOT_USER}/.zshrc && \
    echo "source /home/${NON_ROOT_USER}/aliases.sh" >> /home/${NON_ROOT_USER}/.zshrc && \
    echo "" >> /home/${NON_ROOT_USER}/.zshrc \
  ;fi \
;fi

USER root

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
