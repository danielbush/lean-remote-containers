# FROM ubuntu:20.04
FROM debian:unstable

# Install create a lean user and install lean theorem prover.
# For personal, non-server use.
# Loosely based on https://raw.githubusercontent.com/leanprover-community/mathlib-tools/master/scripts/install_debian.sh.

LABEL maintainer="dlb.id.au@gmail.com"

# There are at least 2 approaches to set up on linux:
# https://leanprover-community.github.io/install/linux.html
# https://leanprover-community.github.io/install/debian.html

RUN set -x; \
  apt-get update; \
  apt-get install -y --no-install-recommends sudo less wget man-db; \
  apt-get install -y git curl python3 python3-pip python3-venv;
RUN set -x; \
  groupadd lean; \
  useradd --create-home -g lean -G sudo --home-dir=/home/lean --shell=/bin/bash lean; \
  echo "lean ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers; \
  chown -R lean:lean /home/lean;

USER lean
WORKDIR /home/lean

# - Move .profile into .bash_profile.
#   So that lean and leanproject are loaded into the shell.

RUN set -x; \
  wget https://raw.githubusercontent.com/Kha/elan/master/elan-init.sh; \
  bash elan-init.sh -y; \
  python3 -m pip install --user pipx; \
  python3 -m pipx ensurepath; \
  mv ~/.profile ~/.bash_profile; \
  . ~/.bash_profile; \
  pipx install mathlibtools;